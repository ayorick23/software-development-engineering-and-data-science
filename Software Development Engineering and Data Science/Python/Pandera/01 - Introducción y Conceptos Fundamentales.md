---
tags: [pandera, testing, validacion-de-datos, cheat-sheet]
---

# 01 — Introducción y Conceptos Fundamentales

> Este cheat-sheet profundiza en la sintaxis y API de Pandera. Para el contexto de "por qué validar datos" dentro de las tres capas de testing en ML, ver [[13-Testing-en-Machine-Learning]] en `Machine Learning/`.

## ¿Qué es Pandera y qué problema resuelve?

**Pandera** es una librería de validación de datos en tiempo de ejecución para DataFrames — permite declarar el "esquema esperado" de un DataFrame (columnas, tipos, rangos, nulos permitidos, relaciones entre columnas) y verificar automáticamente que los datos reales lo cumplan, lanzando un error informativo si no.

El problema que resuelve: en un pipeline de ML, el código puede ejecutarse "sin errores" y aun así producir resultados basura porque los **datos de entrada** estaban corruptos (una columna con nulos inesperados, un tipo de dato que cambió silenciosamente, valores fuera de rango) — un pipeline sin validación de datos falla *silenciosamente*, y el problema se descubre semanas después cuando el negocio nota que las predicciones están mal.

```python
# Sin Pandera — el error se descubre tarde, en el resultado final, no en el origen
def procesar(df):
    df["total"] = df["cantidad"] * df["precio"]   # si "precio" tiene nulos, esto falla silenciosamente en pandas
    return df

# Con Pandera — el error se detecta INMEDIATAMENTE al entrar al pipeline, con un mensaje claro
schema = pa.DataFrameSchema({
    "cantidad": pa.Column(int, pa.Check.greater_than(0)),
    "precio": pa.Column(float, nullable=False),
})

def procesar(df):
    df = schema.validate(df)   # lanza SchemaError aquí mismo si algo no cumple
    df["total"] = df["cantidad"] * df["precio"]
    return df
```

## Validación en runtime vs. type hints estáticos — por qué no basta con `mypy`

Un type hint como `df: pd.DataFrame` no dice nada sobre **qué columnas** tiene ese DataFrame, sus tipos, ni sus rangos válidos — `mypy`/type hints verifican la estructura del código en tiempo de análisis estático, pero no pueden verificar el **contenido real** de los datos, que solo se conoce en tiempo de ejecución. Pandera cierra exactamente ese hueco: es a los DataFrames lo que Pydantic es a los objetos Python — validación de estructura y contenido, en runtime, con mensajes de error claros.

## Instalación

```bash
pip install pandera

# Con soporte de librerías adicionales de DataFrame:
pip install "pandera[polars]"
pip install "pandera[dask]"
pip install "pandera[pyspark]"
pip install "pandera[hypotheses]"   # para generación de datos sintéticos, ver 08
```

## Las dos APIs de Pandera — panorama

Pandera ofrece dos formas equivalentes de declarar un esquema — elegir una es principalmente una cuestión de estilo y del resto del proyecto:

| API | Estilo | Cuándo preferirla |
|---|---|---|
| **`DataFrameSchema`** (funcional) | Diccionario de columnas, similar a construir el esquema "a mano" | Esquemas generados dinámicamente, migraciones rápidas desde validación ad-hoc |
| **`DataFrameModel`** (basada en clases) | Clases con anotaciones de tipo, estilo Pydantic | Proyectos que ya usan Pydantic/type hints extensivamente, mejor soporte de autocompletado en IDE |

```python
# DataFrameSchema — funcional
import pandera as pa

schema = pa.DataFrameSchema({
    "office_id": pa.Column(int, pa.Check.greater_than(0)),
    "total_demand": pa.Column(float, pa.Check.in_range(0, 100_000)),
})

# DataFrameModel — basada en clases, equivalente funcionalmente
from pandera.typing import Series

class DemandSchema(pa.DataFrameModel):
    office_id: Series[int] = pa.Field(gt=0)
    total_demand: Series[float] = pa.Field(in_range={"min_value": 0, "max_value": 100_000})
```

Ver [[02 - DataFrameSchema - API Funcional]] y [[03 - DataFrameModel - API Basada en Clases]] para el detalle completo de cada una.

## Quickstart — validar un DataFrame

```python
import pandas as pd
import pandera as pa

schema = pa.DataFrameSchema({
    "office_id": pa.Column(int, pa.Check.greater_than(0)),
    "total_demand": pa.Column(float, pa.Check.in_range(0, 100_000), nullable=False),
    "interval_start": pa.Column("datetime64[ns]"),
})

df = pd.DataFrame({
    "office_id": [1, 2, 3],
    "total_demand": [120.5, 89.2, 45.0],
    "interval_start": pd.to_datetime(["2026-08-01", "2026-08-02", "2026-08-03"]),
})

df_validado = schema.validate(df)   # retorna el MISMO DataFrame si es válido; lanza SchemaError si no
```

`schema.validate(df)` retorna el DataFrame validado (permitiendo encadenar en un pipeline funcional: `df.pipe(schema.validate)`) — no es solo un chequeo booleano, es parte del flujo de datos.

## `schema.validate` como parte de un pipeline con pandas

```python
resultado = (
    df
    .pipe(schema.validate)
    .assign(demanda_por_hora=lambda d: d["total_demand"] / 24)
)
```

Este patrón — validar como un paso más de una cadena de `.pipe()` — es la forma idiomática de insertar Pandera en un pipeline de transformación de pandas sin romper el estilo fluido de encadenamiento de métodos.

## Panorama de este cheat-sheet

| Tema | Archivo |
|---|---|
| API funcional con diccionarios | [[02 - DataFrameSchema - API Funcional]] |
| API basada en clases (estilo Pydantic) | [[03 - DataFrameModel - API Basada en Clases]] |
| Checks incorporados y personalizados | [[04 - Checks en Profundidad]] |
| Validar funciones completas, no solo DataFrames | [[05 - Validación de Funciones - Decoradores]] |
| Manejo de errores y reporte completo de fallos | [[06 - Manejo de Errores y Validación Perezosa (Lazy)]] |
| Coerción de tipos y filtrado de filas inválidas | [[07 - Coerción, Tipos de Datos y Filtrado de Filas Inválidas]] |
| Generar datos de prueba desde un esquema | [[08 - Generación de Datos Sintéticos con Hypothesis]] |
| Polars, Dask, PySpark, FastAPI, pytest | [[09 - Integración con el Ecosistema]] |
| Dónde ubicar validaciones en producción | [[10 - Buenas Prácticas y Pandera en Producción]] |

## Ver también

- [[13-Testing-en-Machine-Learning]] (en `Machine Learning/`)
- `Machine Learning/07-Librerias-de-Data-Science-y-ML.md`
- `MLflow/02 - Tracking - Fundamentos y API de Logging.md`
