---
tags: [testing, mlops, python, calidad-de-datos]
---

# 13 — Testing en Machine Learning

> Nota del mentor: en software tradicional, "testear" significa verificar que el código haga lo que dice que hace. En ML tienes que testear **tres cosas distintas y separadas**: el código, los datos, y el modelo. La mayoría de los equipos jóvenes solo testean el primero, y ahí es donde vienen los desastres silenciosos — un pipeline que "corre bien" pero entrena con datos corruptos y nadie se entera hasta que el negocio nota que el forecast está mal por semanas.

## 1. Las tres capas de testing en ML

```
┌─────────────────────┐
│  Tests de Código     │  ← pytest tradicional: funciones, clases, lógica
├─────────────────────┤
│  Tests de Datos       │  ← Great Expectations, Pandera: esquema, rangos, nulos
├─────────────────────┤
│  Tests de Modelo      │  ← comportamiento, invariancia, métricas mínimas
└─────────────────────┘
```

Esta capa intermedia (datos) es la gran ausente en la mayoría de proyectos heredados como el que tomaste de Adrián — y es, en la práctica, la que más incidentes de producción previene en pipelines de forecasting como el de Claro RD.

## 2. Tests de código con `pytest`

Lo básico que ya conoces de ingeniería de software aplica igual:

```python
# tests/test_feature_engineering.py
import pandas as pd
from forecasting_pipeline.feature_engineering import build_lag_features

def test_build_lag_features_shape():
    df = pd.DataFrame({"demand": range(100)})
    result = build_lag_features(df, lags=[1, 7, 30])
    assert result.shape[0] == df.shape[0]
    assert "demand_lag_1" in result.columns

def test_build_lag_features_no_leakage():
    """El lag no debe usar información del futuro."""
    df = pd.DataFrame({"demand": [10, 20, 30]})
    result = build_lag_features(df, lags=[1])
    assert result.loc[1, "demand_lag_1"] == 10
    assert pd.isna(result.loc[0, "demand_lag_1"])
```

Cosas específicas de ML que debes probar aquí, más allá de "la función no truena":

- **Fugas de datos (data leakage)**: que un feature no use información del futuro (crítico en series de tiempo como tu pipeline de Claro RD).
- **Casos borde**: DataFrame vacío, una sola fila, valores nulos, un `office_id` que nunca ha tenido demanda.
- **Determinismo**: dado el mismo input, el mismo output — importante cuando hay pasos con aleatoriedad (`train_test_split`, inicialización de modelos) y necesitas fijar `random_state`.

## 3. Tests de datos: el eslabón que casi nadie hace bien

Aquí es donde entran librerías como **Great Expectations** y **Pandera**, mencionadas en [[07-Librerias-de-Data-Science-y-ML]]. La idea central: declaras "expectativas" sobre tus datos, y el pipeline falla (o alerta) si no se cumplen — **antes** de que esos datos lleguen al modelo.

```python
import pandera as pa
from pandera import Column, Check

demand_schema = pa.DataFrameSchema({
    "office_id": Column(int, Check.greater_than(0)),
    "total_demand": Column(float, Check.in_range(0, 100_000), nullable=False),
    "interval_start": Column("datetime64[ns]"),
    "avg_service_time": Column(float, Check.in_range(0, 3600)),
})

def validate_input(df):
    return demand_schema.validate(df)  # lanza excepción si algo no cumple
```

Piensa en esto como el equivalente de datos a un `assert` de código — pero corriendo automáticamente en cada ejecución del pipeline, no solo en tus pruebas locales. Esto conecta directo con [[11-Logging-en-Python-para-ML]]: cuando una validación de datos falla, el logging es lo que te dice exactamente **qué** falló y **por qué**.

## 4. Tests de modelo: lo más distinto a testing tradicional

Aquí no puedes usar `assert resultado == esperado` porque un modelo de ML es probabilístico, no determinista en el sentido clásico. Las técnicas que sí funcionan:

- **Tests de invarianza**: si cambias algo que *no debería* afectar la predicción (ej. el orden de las filas, o una unidad irrelevante), la predicción no debería cambiar significativamente.
- **Tests de comportamiento esperado (behavioral tests)**: si aumentas artificialmente la demanda histórica de un feature de entrada, la predicción de agentes requeridos debería subir, no bajar — esto prueba que el modelo "entiende" la relación de negocio, no solo que memorizó datos.
- **Tests de umbral mínimo de performance**: antes de promover un modelo a producción (o antes de reemplazar un "champion" por un "challenger", como en tu sistema de autoaprendizaje), el modelo nuevo debe superar un MAE/RMSE mínimo en un conjunto de validación fijo.
- **Tests de regresión de modelo**: comparar las predicciones del modelo nuevo contra las del modelo anterior en un mismo conjunto de datos — si el cambio es drástico sin razón de negocio clara, es una señal de alerta antes de desplegar.

Este último punto es exactamente lo que ya vives en tu sistema champion/challenger: el gate de MAE/RMSE que evita reemplazar un modelo bueno por uno peor es, en esencia, un test de modelo automatizado.

## 5. Cobertura de tests — cuidado con la métrica vanidosa

`pytest-cov` te da un porcentaje de líneas cubiertas por tests. Es útil, pero un consultor con 20 años en esto te va a decir lo mismo siempre: **100% de cobertura no significa 0% de bugs**. Puedes cubrir una línea sin probar el caso que realmente importa. Prioriza:

1. Lógica de negocio crítica (cálculo de Erlang C, transformación de features).
2. Puntos de integración (conexión a base de datos, llamadas a MLflow).
3. Casos borde conocidos por incidentes pasados (ej. el bug de `warm_start` que descubriste — ese tipo de bug, una vez corregido, **siempre** debe quedar cubierto por un test de regresión para que nunca vuelva a colarse silenciosamente).

## 6. Dónde viven los tests en el flujo de trabajo

```
Escribes código → pytest local → commit → push a GitLab
                                              ↓
                                   GitLab CI ejecuta pytest + validaciones de datos
                                              ↓
                                   Si pasa → merge permitido → deploy
```

Esto es exactamente el puente hacia [[14-CICD-para-ML-con-GitLab]]: los tests no sirven de mucho si solo corren en tu máquina. El valor real aparece cuando **ningún cambio puede llegar a producción sin pasar por ellos automáticamente**.

## Ver también

- [[11-Logging-en-Python-para-ML]]
- [[12-Gestion-Moderna-de-Proyectos-Python]]
- [[14-CICD-para-ML-con-GitLab]]
- [[07-Librerias-de-Data-Science-y-ML]]
