---
tags: [pandera, testing, validacion-de-datos, checks, cheat-sheet]
---

# 04 — Checks en Profundidad

> Continúa de [[03 - DataFrameModel - API Basada en Clases]].

## Checks incorporados — referencia rápida

```python
import pandera as pa

pa.Check.greater_than(0)
pa.Check.greater_than_or_equal_to(0)
pa.Check.less_than(100)
pa.Check.less_than_or_equal_to(100)
pa.Check.in_range(0, 100)                          # inclusive por defecto
pa.Check.equal_to(5)
pa.Check.not_equal_to(0)
pa.Check.isin(["A", "B", "C"])
pa.Check.notin(["X", "Y"])
pa.Check.str_matches(r"^[A-Z]{2}\d{4}$")             # regex completo
pa.Check.str_contains("error")
pa.Check.str_startswith("prod_")
pa.Check.str_endswith(".csv")
pa.Check.str_length(min_value=1, max_value=50)
pa.Check.unique_values_eq(["A", "B", "C"])            # el conjunto de valores únicos debe ser EXACTAMENTE ese
```

## Combinar múltiples checks en una columna

```python
pa.Column(float, checks=[
    pa.Check.greater_than(0),
    pa.Check.less_than(1_000_000),
])
```

Todos los checks de la lista deben cumplirse — se evalúan de forma independiente, y con `lazy=True` (ver [[06 - Manejo de Errores y Validación Perezosa (Lazy)]]) se reportan todos los que fallen, no solo el primero.

## Checks a nivel de elemento vs. vectorizados

```python
# Element-wise: la función se aplica a CADA valor individual
check_elemento = pa.Check(lambda x: x % 2 == 0, element_wise=True)

# Vectorizado (default): la función recibe la Serie COMPLETA, debe retornar una Serie booleana
check_vectorizado = pa.Check(lambda serie: serie % 2 == 0)   # más eficiente — usa operaciones de pandas/numpy
```

> **Preferir checks vectorizados sobre `element_wise=True` por rendimiento**: `element_wise=True` itera fila por fila en Python puro, mucho más lento en DataFrames grandes que una operación vectorizada de pandas/NumPy que procesa la Serie completa de una vez. Reservar `element_wise=True` para lógica que genuinamente no se puede vectorizar fácilmente.

## Checks a nivel de DataFrame completo — relaciones entre columnas

```python
schema = pa.DataFrameSchema(
    columns={
        "demanda_predicha": pa.Column(float),
        "demanda_real": pa.Column(float),
    },
    checks=pa.Check(
        lambda df: (abs(df["demanda_predicha"] - df["demanda_real"]) / df["demanda_real"]) < 0.5,
        error="el error relativo no debe superar el 50%",
    ),
)
```

A diferencia de un check declarado dentro de una `Column` (que solo ve esa columna), un check a nivel de `DataFrameSchema` recibe el **DataFrame completo** — indispensable para validar relaciones entre columnas (ej. `fecha_fin` debe ser posterior a `fecha_inicio`, o que la suma de columnas de un desglose coincida con una columna de total).

```python
schema = pa.DataFrameSchema(
    columns={"fecha_inicio": pa.Column("datetime64[ns]"), "fecha_fin": pa.Column("datetime64[ns]")},
    checks=pa.Check(lambda df: df["fecha_fin"] > df["fecha_inicio"], error="fecha_fin debe ser posterior a fecha_inicio"),
)
```

## Checks agrupados — `groupby`

```python
pa.Column(
    float,
    checks=pa.Check(
        lambda grupo: grupo.sum() > 0,
        groupby="region",   # el check se evalúa POR CADA GRUPO de "region", no sobre la columna completa de una vez
    ),
)
```

Verifica una condición que debe cumplirse dentro de cada grupo por separado — por ejemplo, "la demanda total de cada región debe ser positiva", en vez de solo "la suma de toda la columna es positiva" (que no detectaría una región específica con demanda negativa compensada por otra).

## Mensajes de error personalizados

```python
pa.Check.greater_than(0, error="office_id debe ser un entero positivo, nunca cero o negativo")
```

Un mensaje de error descriptivo facilita enormemente el debugging cuando la validación falla en producción — el mensaje por defecto (`"greater_than(0)"`) dice *qué regla* falló pero no *por qué importa*, mientras un mensaje custom puede explicar el contexto de negocio.

## Checks personalizados reutilizables — `register_check_method`

```python
import pandera.extensions as extensions

@extensions.register_check_method(statistics=["umbral_maximo"])
def es_demanda_razonable(serie: pd.Series, umbral_maximo: float) -> pd.Series:
    return serie <= umbral_maximo

# Uso, ya integrado como si fuera un check nativo:
pa.Column(float, pa.Check.es_demanda_razonable(umbral_maximo=100_000))
```

`register_check_method` convierte una función propia en un check de **primera clase**, reutilizable en cualquier esquema del proyecto con la misma sintaxis fluida que los checks incorporados (`pa.Check.mi_check_custom(...)`) — preferible a repetir un `pa.Check(lambda ...)` idéntico en múltiples esquemas.

## Checks estadísticos / de hipótesis

```python
from pandera import Hypothesis
from scipy import stats

pa.Column(
    float,
    checks=Hypothesis(
        test=stats.normaltest,
        relationship=lambda stat, p_value: p_value > 0.05,
        error="la columna debería seguir una distribución aproximadamente normal",
    ),
)
```

`Hypothesis` conecta Pandera con pruebas estadísticas de `scipy.stats` — útil para validaciones más sofisticadas que un simple rango, como verificar que una columna no se haya desviado significativamente de una distribución esperada (relevante para detectar drift temprano en un pipeline, complementario a herramientas de monitoreo dedicadas como Evidently, ver `Machine Learning/18-Monitoreo-y-Observabilidad-de-Modelos.md`).

## Comparar dos columnas directamente entre sí

```python
pa.Check(lambda df: df["columna_a"] >= df["columna_b"], error="columna_a debe ser >= columna_b")
```

Patrón general para cualquier invariante entre columnas que no encaje en los checks incorporados (`greater_than`, etc., que comparan contra un valor fijo, no contra otra columna) — el check recibe el DataFrame y compara las Series directamente.

## Ver también

- [[02 - DataFrameSchema - API Funcional]]
- [[03 - DataFrameModel - API Basada en Clases]]
- [[06 - Manejo de Errores y Validación Perezosa (Lazy)]]
