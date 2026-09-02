---
tags: [polars, python, data-science, groupby, aggregation, cheat-sheet]
---

# 08 — Agrupación y Agregación

> Continúa de [[07 - Valores Nulos]].

## `group_by()` + `agg()` — el patrón central

```python
df.group_by("region").agg(pl.col("monto").sum())
df.group_by("region").agg(pl.col("monto").sum().alias("monto_total"))

df.group_by(["region", "producto"]).agg(pl.col("monto").sum())     # múltiples claves de agrupación
```

A diferencia de Pandas, donde `groupby()` seguido de una sola función (`.sum()`) es el patrón simple y `agg()` se reserva para casos con múltiples agregaciones, en Polars **siempre** se usa `.agg()` con expresiones — no existe un atajo `.group_by("region").sum()` tan idiomático como en Pandas, porque el patrón de expresiones ya es igual de compacto.

## Múltiples agregaciones en una sola llamada

```python
df.group_by("region").agg(
    pl.col("monto").sum().alias("monto_total"),
    pl.col("monto").mean().alias("monto_promedio"),
    pl.col("monto").count().alias("num_transacciones"),
    pl.col("precio").max().alias("precio_maximo"),
)
```

Cada expresión dentro de `agg()` es independiente y se calcula en **paralelo** — el mismo mecanismo de paralelización automática visto en [[04 - Expresiones (pl.col)#Múltiples expresiones en una sola llamada — paralelización automática|Expresiones]], aplicado ahora dentro de cada grupo.

## Agregaciones sin especificar columna primero

```python
df.group_by("region").agg(pl.len())                       # número de filas por grupo (equivalente a size() de Pandas)
df.group_by("region").agg(pl.col("*").count())               # conteo de valores no nulos, todas las columnas
```

## `over()` — funciones de ventana (window functions), el equivalente a `transform()`

```python
df.with_columns(
    pl.col("monto").mean().over("region").alias("promedio_region")
)
```

`over()` es el equivalente directo de `transform()` de Pandas (ver [[Python/Pandas/08 - Agrupación y Agregación (GroupBy)#transform() vs apply() vs agg() — la diferencia que confunde a todos|Python/Pandas]]): calcula una agregación por grupo, pero devuelve un resultado del **mismo tamaño** que el DataFrame original (broadcast), en vez de una fila por grupo — útil para comparar cada fila individual contra el resumen de su propio grupo.

```python
df.with_columns(
    (pl.col("monto") - pl.col("monto").mean().over("region")).alias("desviacion_vs_region")
)
```

## Ordenar dentro de cada grupo — `over()` con `order_by`

```python
df.with_columns(
    pl.col("monto").rank(descending=True).over("region").alias("ranking_en_region")
)
```

`rank().over("region")` asigna un ranking a cada fila **dentro de su propio grupo** — equivalente al patrón `ROW_NUMBER() OVER (PARTITION BY ...)` de SQL (ver [[SQL/Window Functions|SQL]]), expresado como una expresión encadenable de Polars.

## `pivot()` — agregación con salida en formato tabla cruzada

```python
df.pivot(index="region", columns="producto", values="monto", aggregate_function="sum")
```

Equivalente a `pivot_table()` de Pandas — ver el detalle completo de reshaping en [[10 - Reshaping]].

## Filtrar grupos completos según una condición agregada

```python
(
    df.group_by("producto")
    .agg(pl.col("monto").sum().alias("total"))
    .filter(pl.col("total") > 10_000)
)
```

A diferencia de `filter()` de `groupby` en Pandas (que devuelve las filas originales completas), en Polars el patrón idiomático es agregar primero y luego filtrar el resultado agregado — si se necesitan las filas originales de los grupos que pasan el filtro, se hace un `join` de vuelta contra el DataFrame original usando la clave de grupo.

## Ver también

- [[07 - Valores Nulos]]
- [[09 - Combinación de DataFrames]]
- [[Python/Pandas/08 - Agrupación y Agregación (GroupBy)|Python/Pandas]]
- [[SQL/Window Functions|SQL]]
