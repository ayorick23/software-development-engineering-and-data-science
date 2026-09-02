---
tags: [polars, python, data-science, io, cheat-sheet]
---

# 03 — Creación y Carga de Datos

> Continúa de [[02 - Eager vs Lazy API]].

## Creación manual

```python
df = pl.DataFrame({
    "a": [1, 2, 3],
    "b": ["x", "y", "z"],
})

df = pl.DataFrame({"a": [1, 2, 3]}, schema={"a": pl.Int64})   # dtype explícito en la creación
```

## Lectura eager: `read_*`

```python
df = pl.read_csv("ventas.csv")
df = pl.read_csv("ventas.csv", separator=";", try_parse_dates=True)
df = pl.read_parquet("ventas.parquet")
df = pl.read_excel("reporte.xlsx", sheet_name="Ventas")     # requiere el paquete 'xlsx2csv' o 'openpyxl' instalado
df = pl.read_json("ventas.json")
df = pl.read_database(query="SELECT * FROM ventas", connection=conexion)   # vía SQLAlchemy o un connection string
```

Estas funciones cargan el archivo **completo** en memoria de inmediato — equivalentes conceptuales a `pd.read_csv()` de Pandas.

## Lectura lazy: `scan_*` (la forma recomendada en producción)

```python
lf = pl.scan_csv("ventas.csv")               # NO lee nada todavía, solo registra la fuente
lf = pl.scan_parquet("ventas.parquet")
lf = pl.scan_parquet("ventas_*.parquet")       # patrón glob — lee MÚLTIPLES archivos parquet como si fueran uno solo

resultado = lf.filter(pl.col("monto") > 100).collect()    # la lectura real ocurre aquí, ya optimizada
```

`scan_parquet()` con un patrón glob es particularmente útil para datasets particionados en múltiples archivos (el patrón común de un data lake) — Polars los trata como una sola fuente lógica sin necesitar concatenarlos manualmente primero.

## Parámetros importantes de lectura de CSV

```python
pl.read_csv(
    "ventas.csv",
    separator=",",
    has_header=True,
    columns=["fecha", "producto", "monto"],       # leer solo estas columnas (equivalente a usecols de Pandas)
    dtypes={"producto": pl.Categorical},             # dtype explícito por columna
    try_parse_dates=True,                              # intenta inferir y parsear columnas de fecha automáticamente
    null_values=["N/A", "-", ""],
    n_rows=1000,                                          # solo para explorar archivos grandes
    infer_schema_length=10000,                              # cuántas filas usar para INFERIR el schema (no todo el archivo)
)
```

`infer_schema_length` es un parámetro propio de Polars sin equivalente directo en Pandas: en vez de leer el archivo completo para inferir tipos, Polars solo mira las primeras N filas — más rápido, pero puede fallar si el tipo real de una columna cambia más adelante en el archivo (en ese caso, declarar `dtypes=` explícitamente es más seguro).

## Desde NumPy y otras estructuras

```python
import numpy as np
df = pl.DataFrame(np.random.randn(4, 3), schema=["x", "y", "z"])

df = pl.from_dicts([{"a": 1, "b": 2}, {"a": 3, "b": 4}])     # lista de diccionarios -> DataFrame
df = pl.from_numpy(np.random.randn(4, 3), schema=["x", "y", "z"])
```

## Interoperar con Pandas y Arrow al cargar

```python
df_polars = pl.from_pandas(df_pandas)          # convertir un DataFrame de Pandas existente
df_polars = pl.from_arrow(tabla_arrow)           # desde una tabla de PyArrow, sin copia si es posible
```

Ver el detalle completo de interoperabilidad bidireccional en [[16 - Integración con el Ecosistema]].

## Ver también

- [[02 - Eager vs Lazy API]]
- [[15 - Exportación de Datos]]
- [[16 - Integración con el Ecosistema]]
