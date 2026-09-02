---
tags: [pandera, testing, validacion-de-datos, errores, cheat-sheet]
---

# 06 — Manejo de Errores y Validación Perezosa (Lazy)

> Continúa de [[05 - Validación de Funciones - Decoradores]].

## Comportamiento por defecto — falla en el primer error

```python
import pandera as pa

schema = pa.DataFrameSchema({
    "office_id": pa.Column(int, pa.Check.greater_than(0)),
    "total_demand": pa.Column(float, pa.Check.greater_than_or_equal_to(0)),
})

try:
    schema.validate(df)
except pa.errors.SchemaError as e:
    print(e.failure_cases)   # DataFrame con las filas específicas que fallaron
    print(e.data)              # el DataFrame original completo
    print(e.schema)             # el esquema/columna que causó el fallo
```

Por defecto, `validate()` se detiene en el **primer** check que falla — si hay múltiples columnas con problemas, solo se ve el primero, obligando a corregir y volver a ejecutar repetidamente para descubrir el resto de los problemas uno por uno.

## `lazy=True` — recolectar TODOS los errores de una vez

```python
try:
    schema.validate(df, lazy=True)
except pa.errors.SchemaErrors as e:   # nótese el plural: SchemaErrorS
    print(e.failure_cases)   # DataFrame con TODAS las filas/columnas que fallaron, de todos los checks
```

`lazy=True` cambia el comportamiento de "falla rápido" a "recolecta todo" — evalúa **todos** los checks de **todas** las columnas antes de lanzar la excepción, reportando el panorama completo de problemas en una sola pasada. En la práctica, casi siempre es la opción preferible durante desarrollo/debugging de un esquema nuevo, para no tener que iterar error por error.

## `SchemaError` vs. `SchemaErrors` — la diferencia importa

| Excepción | Cuándo se lanza | Contenido |
|---|---|---|
| `SchemaError` | `lazy=False` (default), se detiene en el primer fallo | Un solo problema específico |
| `SchemaErrors` | `lazy=True` | Colección de TODOS los problemas encontrados |

```python
except pa.errors.SchemaErrors as e:
    print(e.failure_cases.head(20))   # tabla con: columna, check que falló, índice de fila, valor problemático
```

## Anatomía de `failure_cases` — el DataFrame de diagnóstico

```
   schema_context   column          check                  failure_case  index
0  Column           office_id       greater_than(0)         -3            2
1  Column           total_demand    greater_than_or_equal_to(0)   -50.0    7
2  DataFrameSchema   None            error relativo <0.5     None         2
```

Cada fila de `failure_cases` describe un fallo específico: qué columna, qué check falló, qué valor causó el fallo, y en qué índice del DataFrame original — suficiente información para localizar y corregir el problema sin tener que inspeccionar el DataFrame completo manualmente.

## Extraer las filas problemáticas del DataFrame original

```python
try:
    schema.validate(df, lazy=True)
except pa.errors.SchemaErrors as e:
    indices_con_error = e.failure_cases["index"].unique()
    filas_problematicas = df.loc[indices_con_error]
    print(filas_problematicas)
```

Patrón habitual para logging/alertas: capturar exactamente qué filas causaron el problema, no solo que "algo falló" — conecta directamente con el logging estructurado cubierto en `Machine Learning/11-Logging-en-Python-para-ML.md`.

## Loguear errores de validación de forma estructurada

```python
import logging

logger = logging.getLogger(__name__)

def validar_con_logging(df, schema):
    try:
        return schema.validate(df, lazy=True)
    except pa.errors.SchemaErrors as e:
        logger.error(
            "Validación de datos falló: %d problemas encontrados",
            len(e.failure_cases),
            extra={"failure_cases": e.failure_cases.to_dict("records")},
        )
        raise
```

Esto es exactamente el patrón descrito en `Machine Learning/13-Testing-en-Machine-Learning.md`: cuando una validación falla, el logging estructurado es lo que permite después responder *qué* falló y *por qué*, en vez de solo saber que el pipeline se detuvo.

## `report_duplicates` — controlar cómo se reportan duplicados

```python
schema = pa.DataFrameSchema(
    {"id": pa.Column(int, unique=True)},
    report_duplicates="all",   # "exclude_first" (default) o "all" — reporta TODAS las filas duplicadas, no solo la segunda ocurrencia
)
```

## Validación parcial — capturar el error pero continuar el pipeline con los datos válidos

```python
def validar_y_continuar(df, schema):
    try:
        return schema.validate(df, lazy=True), None
    except pa.errors.SchemaErrors as e:
        return None, e.failure_cases

df_validado, errores = validar_y_continuar(df, schema)
if errores is not None:
    enviar_alerta_a_slack(f"{len(errores)} problemas de calidad de datos detectados")
    # decisión de negocio: ¿detener el pipeline, o continuar con drop_invalid_rows (ver 07)?
```

Este patrón — capturar el error sin re-lanzarlo, decidir explícitamente qué hacer — es la base para implementar políticas de tolerancia a errores de datos (alertar pero continuar, vs. detener el pipeline por completo), cubierto con más profundidad en [[07 - Coerción, Tipos de Datos y Filtrado de Filas Inválidas]] y [[10 - Buenas Prácticas y Pandera en Producción]].

## Ver también

- [[04 - Checks en Profundidad]]
- [[07 - Coerción, Tipos de Datos y Filtrado de Filas Inválidas]]
- `Machine Learning/11-Logging-en-Python-para-ML.md`
- `Machine Learning/13-Testing-en-Machine-Learning.md`
