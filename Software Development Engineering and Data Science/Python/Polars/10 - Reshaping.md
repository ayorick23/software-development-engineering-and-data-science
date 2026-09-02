---
tags: [polars, python, data-science, reshape, cheat-sheet]
---

# 10 — Reshaping

> Continúa de [[09 - Combinación de DataFrames]].

## `melt()` — de wide a long

```python
df_long = df.melt(
    id_vars=["fecha"],
    value_vars=["producto_A", "producto_B"],
    variable_name="producto",
    value_name="valor",
)
```

Misma idea que `melt()` de Pandas (ver [[Python/Pandas/10 - Reshaping y Pivoting|Python/Pandas]]) — solo cambian los nombres de los parámetros (`variable_name`/`value_name` en vez de `var_name`/`value_name`).

## `pivot()` — de long a wide, con agregación

```python
df_long.pivot(index="fecha", columns="producto", values="valor", aggregate_function="sum")
df_long.pivot(index="fecha", columns="producto", values="valor", aggregate_function="first")   # sin agregar, si ya es único
```

A diferencia de Pandas (que separa `pivot()` sin agregación de `pivot_table()` con agregación), Polars unifica ambos casos en una sola función `pivot()` — `aggregate_function` es opcional y solo se necesita cuando hay múltiples filas por combinación `(index, columns)`.

## `transpose()`

```python
df.transpose(include_header=True, header_name="columna", column_names="fila")
```

## `explode()` — desanidar columnas de tipo lista

```python
df = pl.DataFrame({"id": [1, 2], "tags": [["python", "sql"], ["ml"]]})
df.explode("tags")
```

Mismo comportamiento que `explode()` de Pandas — particularmente natural en Polars porque el dtype `List` (ver [[12 - Texto y Categóricos]]) es un tipo de primera clase nativo, no una columna `object` genérica conteniendo listas de Python como en Pandas.

## `unnest()` — desanidar columnas tipo struct

```python
df = pl.DataFrame({"info": [{"nombre": "Ana", "edad": 28}, {"nombre": "Luis", "edad": 35}]})
df.unnest("info")     # convierte la columna 'info' (dtype Struct) en dos columnas: 'nombre' y 'edad'
```

El dtype `Struct` (un valor anidado tipo diccionario dentro de una celda) es otro tipo nativo propio de Polars sin equivalente directo tan limpio en Pandas — común al leer JSON anidado; `unnest()` es la operación inversa a construir un struct, aplanando esos campos a columnas de nivel superior.

## Ver también

- [[09 - Combinación de DataFrames]]
- [[12 - Texto y Categóricos]]
- [[Python/Pandas/10 - Reshaping y Pivoting|Python/Pandas]]
