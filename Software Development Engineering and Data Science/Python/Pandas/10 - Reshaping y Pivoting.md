---
tags: [pandas, python, data-science, reshape, pivot, cheat-sheet]
---

# 10 — Reshaping y Pivoting

> Continúa de [[09 - Combinación de DataFrames]].

## Formato "wide" vs "long"

- **Wide**: cada variable/categoría es su propia columna (fácil de leer para humanos).
- **Long** (o "tidy"): cada fila es una observación individual con columnas tipo `variable`/`valor` (formato que esperan la mayoría de las librerías de graficación y modelado — ver [[Machine Learning/40-Feature-Engineering-Avanzado|Feature Engineering]]).

```python
# WIDE
#   fecha       producto_A  producto_B
#   2026-01-01  100         200

# LONG (equivalente)
#   fecha       producto    valor
#   2026-01-01  producto_A  100
#   2026-01-01  producto_B  200
```

## `melt()` — de wide a long

```python
df_long = df.melt(
    id_vars=["fecha"],                       # columnas que se mantienen igual (identificadores)
    value_vars=["producto_A", "producto_B"], # columnas a "derretir" en filas
    var_name="producto",                      # nombre de la nueva columna con los nombres originales
    value_name="valor",                        # nombre de la nueva columna con los valores
)
```

## `pivot()` — de long a wide (sin agregación)

```python
df_wide = df_long.pivot(index="fecha", columns="producto", values="valor")
```

`pivot()` **requiere** que la combinación `(index, columns)` sea única — si hay filas duplicadas para la misma combinación, lanza error. Para ese caso (con agregación de por medio), usar `pivot_table()` (ver [[08 - Agrupación y Agregación (GroupBy)]]).

```python
# pivot_table SÍ tolera duplicados porque agrega con aggfunc
pd.pivot_table(df_long, index="fecha", columns="producto", values="valor", aggfunc="sum")
```

## `stack()` / `unstack()` — mover niveles entre filas y columnas

```python
df_apilado = df_wide.stack()        # convierte columnas en un nivel adicional del índice de filas (wide -> long-ish)
df_desapilado = df_apilado.unstack()  # operación inversa: nivel de índice -> columnas
```

`stack`/`unstack` operan sobre **niveles de un `MultiIndex`** (ver [[13 - MultiIndex y Datos Jerárquicos]]) — son la forma "de bajo nivel" de mover información entre el eje de filas y el eje de columnas; `melt`/`pivot` son la forma "de alto nivel" pensada para trabajar con nombres de columna en vez de niveles de índice.

## `wide_to_long()` — reshape con patrón de nombre de columna

```python
# columnas: id, ventas_2024, ventas_2025, ventas_2026
pd.wide_to_long(df, stubnames="ventas", i="id", j="anio", sep="_")
```

Útil cuando las columnas wide siguen un patrón `prefijo_sufijo` (por ejemplo, una medición repetida por año) y se quiere extraer el sufijo como su propia columna (`anio`) automáticamente.

## `explode()` — desanidar celdas con listas

```python
df = pd.DataFrame({"id": [1, 2], "tags": [["python", "sql"], ["ml"]]})
df.explode("tags")
#    id    tags
#    1     python
#    1     sql
#    2     ml
```

Común al procesar datos semi-estructurados (ej. de una API JSON) donde una celda contiene una lista de valores que en realidad representan filas independientes.

## `T` — transponer

```python
df.T   # intercambia filas y columnas — útil para inspeccionar DataFrames con muchas columnas y pocas filas
```

## Ver también

- [[08 - Agrupación y Agregación (GroupBy)]]
- [[09 - Combinación de DataFrames]]
- [[13 - MultiIndex y Datos Jerárquicos]]
- [[Machine Learning/40-Feature-Engineering-Avanzado|Machine Learning/40 - Feature Engineering Avanzado]]
