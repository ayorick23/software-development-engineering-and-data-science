---
tags: [polars, python, data-science, transformation, cheat-sheet]
---

# 06 — Transformación de Columnas

> Continúa de [[05 - Selección y Filtrado]].

## `with_columns()` — agregar o reemplazar columnas, conservando el resto

```python
df.with_columns(
    (pl.col("precio") * pl.col("cantidad")).alias("total")
)

df.with_columns(
    total=pl.col("precio") * pl.col("cantidad"),                # forma alternativa: keyword = nombre de la nueva columna
    total_iva=pl.col("precio") * pl.col("cantidad") * 1.16,
)
```

`with_columns()` es el equivalente directo de `df.assign()` de Pandas (ver [[Python/Pandas/06 - Operaciones con Columnas|Python/Pandas]]) — a diferencia de `select()` (que descarta las columnas no mencionadas), `with_columns()` conserva todas las columnas originales y añade/reemplaza las indicadas.

## Renombrar columnas

```python
df.rename({"precio": "precio_usd", "stock": "inventario"})
df.select(pl.col("precio").alias("precio_usd"))     # alias dentro de una expresión, equivalente puntual
```

## Cambiar tipo de dato con `cast()`

```python
df.with_columns(pl.col("precio").cast(pl.Float32))
df.with_columns(pl.col("id").cast(pl.Utf8))            # a texto
df.with_columns(pl.col("fecha_str").str.to_date("%Y-%m-%d"))   # string -> Date, con formato explícito
```

`cast()` es el equivalente de `.astype()` de Pandas — a diferencia de Pandas, Polars distingue explícitamente entre `pl.Utf8` (string), `pl.Date` (solo fecha) y `pl.Datetime` (fecha + hora) como tipos separados, en vez de un único `datetime64` genérico.

## `map_elements()` — la salida de emergencia (usar con moderación)

```python
df.with_columns(
    pl.col("descripcion").map_elements(lambda x: procesar_texto_complejo(x), return_dtype=pl.Utf8)
)
```

`map_elements()` es el equivalente de `.apply()` de Pandas: aplica una función Python arbitraria elemento por elemento — **rompe la paralelización y optimización nativas** de Polars porque ejecuta código Python puro fila por fila. Usarlo solo cuando de verdad no existe una expresión nativa equivalente (la gran mayoría de transformaciones comunes sí la tienen).

```python
# LENTO — rompe la vectorización nativa de Polars
df.with_columns(pl.col("precio").map_elements(lambda x: x * 1.16, return_dtype=pl.Float64))

# RÁPIDO — expresión nativa, paralelizada y optimizable
df.with_columns((pl.col("precio") * 1.16).alias("precio"))
```

## `pipe()` — encadenar funciones propias

```python
def agregar_iva(df):
    return df.with_columns((pl.col("precio") * 1.16).alias("precio_con_iva"))

resultado = df.pipe(agregar_iva).filter(pl.col("precio_con_iva") > 100)
```

Igual que `pipe()` en Pandas (ver [[Python/Pandas/06 - Operaciones con Columnas#pipe() — encadenar funciones custom dentro de una cadena de métodos|Python/Pandas]]) — permite meter lógica de transformación propia dentro de una cadena fluida de métodos.

## Operaciones aritméticas y funciones matemáticas directas

```python
df.with_columns(
    (pl.col("precio") / pl.col("stock")).alias("precio_por_unidad"),
    pl.col("precio").log().alias("log_precio"),
    pl.col("precio").sqrt().alias("raiz_precio"),
    pl.col("precio").round(2),
)
```

## Ver también

- [[05 - Selección y Filtrado]]
- [[07 - Valores Nulos]]
- [[Python/Pandas/06 - Operaciones con Columnas|Python/Pandas]]
