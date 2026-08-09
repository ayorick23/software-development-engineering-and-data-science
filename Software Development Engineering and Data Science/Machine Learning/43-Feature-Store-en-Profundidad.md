---
tags: [ingenieria-de-datos, feature-store, feast, mlops]
---

# 43 — Feature Store en Profundidad (Feast)

> Nota del mentor: en la arquitectura empresarial que vimos en [[03-Arquitectura-Empresarial-de-Datos-y-ML]] (ERP→...→Feature Store→Entrenamiento→...), el Feature Store aparece como un eslabón más de la cadena. Pero nunca profundizamos en **por qué existe** y **qué problema real resuelve** — y es exactamente el problema que ya viviste con la duplicación cuádruple de `feature_engineering.py` antes de unificarla.

## 1. El problema que un Feature Store resuelve — duplicación e inconsistencia a escala de organización

Imagina que ACF Technologies no solo tiene el proyecto de Claro RD, sino diez proyectos de forecasting para diez clientes distintos. Sin un feature store centralizado, cada equipo reimplementa su propia lógica de `demand_lag_1`, `promedio_movil_7`, `es_feriado` — exactamente el mismo problema que resolviste al unificar dos notebooks, pero multiplicado por diez proyectos y diez equipos, con el riesgo real de que cada implementación calcule el "mismo" feature de forma sutilmente distinta (¿el lag se calcula sobre datos ya imputados o crudos? ¿el feriado incluye medios días?).

Un Feature Store centraliza la **definición, cálculo y almacenamiento** de features, para que se calculen una sola vez y se reutilicen consistentemente entre proyectos, entre el momento de entrenamiento y el momento de inferencia en producción.

## 2. El problema más sutil: Training-Serving Skew

Este es el problema técnico más importante que resuelve un feature store, y el más fácil de pasar por alto:

```python
# En el notebook de entrenamiento (batch, con pandas, acceso a todo el histórico)
df["promedio_movil_7"] = df.groupby("office_id")["demand"].transform(
    lambda x: x.shift(1).rolling(7).mean()
)

# En el servicio de inferencia en producción (online, un solo registro a la vez)
def calcular_promedio_movil_7(office_id, fecha):
    # ¿esta implementación calcula EXACTAMENTE lo mismo que el pandas de arriba?
    # ¿usa la misma ventana, el mismo shift, el mismo manejo de nulos?
    ...
```

**Training-Serving Skew** ocurre cuando la lógica de cálculo de un feature en entrenamiento (normalmente batch, con pandas/SQL sobre históricos completos) difiere sutilmente de la lógica en producción (normalmente online, calculando sobre un registro a la vez) — el modelo entrena con una versión del feature y predice con otra ligeramente distinta, degradando silenciosamente el desempeño sin que ningún error explícito lo señale. Es una de las causas más comunes y más difíciles de diagnosticar de "el modelo funcionaba bien en desarrollo pero mal en producción".

## 3. Cómo un Feature Store resuelve esto — una sola definición, dos formas de servir

```python
from feast import Entity, FeatureView, Field, FileSource
from feast.types import Float32, Int64
from datetime import timedelta

oficina = Entity(name="office_id", join_keys=["office_id"])

fuente_demanda = FileSource(
    path="demanda_historica.parquet",
    timestamp_field="interval_start",
)

vista_features_demanda = FeatureView(
    name="features_demanda_oficina",
    entities=[oficina],
    ttl=timedelta(days=1),
    schema=[
        Field(name="promedio_movil_7", dtype=Float32),
        Field(name="demand_lag_1", dtype=Float32),
        Field(name="es_feriado", dtype=Int64),
    ],
    source=fuente_demanda,
)
```

Una vez definido, el mismo feature se sirve de dos formas distintas, **desde la misma definición subyacente**:

```python
# Offline store — para entrenamiento, sobre históricos completos (batch)
from feast import FeatureStore
store = FeatureStore(repo_path=".")

datos_entrenamiento = store.get_historical_features(
    entity_df=entity_df,  # office_id + timestamps de interés
    features=["features_demanda_oficina:promedio_movil_7", "features_demanda_oficina:demand_lag_1"],
).to_df()

# Online store — para inferencia, un registro a la vez, baja latencia
features_en_produccion = store.get_online_features(
    features=["features_demanda_oficina:promedio_movil_7", "features_demanda_oficina:demand_lag_1"],
    entity_rows=[{"office_id": 145}],
).to_dict()
```

El **offline store** (típicamente Parquet/BigQuery/Snowflake) sirve grandes volúmenes históricos para entrenamiento. El **online store** (típicamente Redis/DynamoDB, de baja latencia) sirve un registro a la vez para inferencia en tiempo real. Ambos derivan de la **misma definición de feature** — eliminando por diseño la posibilidad de training-serving skew, porque no hay dos implementaciones separadas que puedan divergir.

## 4. Point-in-time correctness — el segundo problema crítico que resuelve

```python
entity_df = pd.DataFrame({
    "office_id": [145, 145],
    "event_timestamp": ["2026-03-15 10:00:00", "2026-06-20 14:00:00"],
})
```

Cuando pides features históricos para entrenar, `get_historical_features` garantiza que para cada fila obtienes el valor del feature **tal como existía en ese momento exacto del pasado**, no el valor actual — esto es exactamente el mismo principio de leakage temporal de [[37-Validacion-Rigurosa-en-ML]], pero resuelto automáticamente por la infraestructura del feature store en vez de depender de que cada ingeniero recuerde poner el `shift(1)` correcto manualmente en cada nuevo proyecto.

## 5. ¿Tu proyecto de Claro RD necesita esto hoy?

Honestamente, probablemente no todavía — un feature store agrega infraestructura operativa (definir entidades, mantener un offline y online store, sincronización entre ambos) que solo se justifica cuando:

- **Múltiples modelos o proyectos comparten los mismos features** (si ACF Technologies tuviera varios modelos de Claro RD, o features reutilizables entre distintos clientes con patrones de negocio similares).
- **Hay inferencia online de baja latencia**, no solo batch — tu pipeline actual es batch (ver [[17-Arquitecturas-de-Despliegue-de-Modelos]]), así que el problema de training-serving skew es menos agudo, porque tanto entrenamiento como "inferencia" (tu forecast cada 30 minutos) corren en el mismo tipo de proceso batch con la misma lógica de `feature_engineering.py`.
- **El equipo ha sufrido ya un incidente real de training-serving skew** — la señal más clara de que vale la pena la inversión.

## 6. Conocerlo igual vale — es una conversación de arquitectura que vas a tener

Aunque no lo implementes hoy, entender el concepto te prepara para dos escenarios reales: proponer con criterio si ACF Technologies decide consolidar features entre clientes, y reconocer rápidamente un training-serving skew si algún día tu pipeline se vuelve híbrido (batch + algún componente online) sin que esa lógica esté centralizada.

## Ver también
- [[03-Arquitectura-Empresarial-de-Datos-y-ML]]
- [[37-Validacion-Rigurosa-en-ML]]
- [[40-Feature-Engineering-Avanzado]]
- [[17-Arquitecturas-de-Despliegue-de-Modelos]]
