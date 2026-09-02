---
tags: [polars, python, data-science, missing-data, cheat-sheet]
---

# 07 — Valores Nulos

> Continúa de [[06 - Transformación de Columnas]].

## Un solo tipo de nulo, sin importar el dtype

```python
df = pl.DataFrame({"a": [1, None, 3], "b": [1.0, None, 3.0]})
df.dtypes     # [Int64, Float64] — 'a' SIGUE siendo Int64, el nulo no lo degrada a float
```

**Diferencia clave con Pandas:** Polars tiene un único valor nulo (`null`) que funciona nativamente en **cualquier** dtype, incluyendo enteros — a diferencia de NumPy/Pandas clásico, donde un entero con nulos se degrada silenciosamente a `float64` porque `NaN` solo existe en punto flotante (ver [[Python/Pandas/01 - Introducción y Arquitectura Interna#El sistema de dtype NumPy vs Extension Arrays|Python/Pandas]]). Esto elimina una clase entera de sorpresas de tipo que sí ocurren en Pandas clásico.

## Detección de nulos

```python
df.null_count()                          # nulos por columna, todo el DataFrame de un vistazo
df.select(pl.col("a").is_null())            # máscara booleana
df.select(pl.col("a").is_not_null())
df.filter(pl.col("a").is_null())              # filas donde 'a' es nulo
```

## `fill_null()` — imputación

```python
df.with_columns(pl.col("precio").fill_null(0))                          # valor fijo
df.with_columns(pl.col("precio").fill_null(pl.col("precio").mean()))      # con la media de la columna
df.with_columns(pl.col("precio").fill_null(strategy="forward"))             # forward fill
df.with_columns(pl.col("precio").fill_null(strategy="backward"))              # backward fill
df.with_columns(pl.col("precio").fill_null(strategy="mean"))                    # estrategia nombrada, equivalente a pasar la expresión mean()
```

`strategy=` con un nombre (`"forward"`, `"backward"`, `"mean"`, `"min"`, `"max"`, `"zero"`, `"one"`) es más legible que construir la expresión equivalente manualmente, y cubre los casos más comunes de imputación sin escribir la lógica explícita.

## `interpolate()` — imputación numérica basada en tendencia

```python
df.with_columns(pl.col("temperatura").interpolate())
```

Equivalente a `interpolate()` de Pandas (ver [[Python/Pandas/07 - Datos Nulos y Duplicados#interpolate() — imputación numérica basada en tendencia|Python/Pandas]]) — interpola linealmente entre valores conocidos.

## `drop_nulls()` — eliminar filas con nulos

```python
df.drop_nulls()                             # elimina cualquier fila con AL MENOS un nulo
df.drop_nulls(subset=["precio", "stock"])     # considera nulos solo en estas columnas
```

## Duplicados

```python
df.is_duplicated()                          # máscara booleana, True en cada fila duplicada
df.unique()                                    # elimina duplicados exactos
df.unique(subset=["email"], keep="last")         # duplicado según columnas específicas, conserva la última ocurrencia
```

`unique()` es el equivalente de `drop_duplicates()` de Pandas — la nomenclatura distinta (`unique` en vez de `drop_duplicates`) es representativa del estilo general de Polars: nombres de método más cortos y consistentes entre sí.

## `coalesce()` — el primer valor no nulo entre varias columnas/expresiones

```python
df.with_columns(
    pl.coalesce(["telefono_movil", "telefono_casa", "telefono_oficina"]).alias("telefono_contacto")
)
```

Equivalente conceptual a `combine_first()` de Pandas pero para múltiples fuentes a la vez, en el mismo estilo que `COALESCE` de SQL (ver [[SQL/DQL (Data Query Language)|SQL]]) — devuelve el primer valor no nulo encontrado, columna por columna, en el orden dado.

## Ver también

- [[06 - Transformación de Columnas]]
- [[08 - Agrupación y Agregación]]
- [[Python/Pandas/07 - Datos Nulos y Duplicados|Python/Pandas]]
