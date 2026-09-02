---
tags: [polars, python, data-science, expressions, cheat-sheet]
---

# 04 — Expresiones (pl.col)

> Continúa de [[03 - Creación y Carga de Datos]]. Si hay un solo concepto que separa mentalmente a Polars de Pandas, es este.

## Qué es una expresión

Una **expresión** en Polars (`pl.col(...)`, `pl.lit(...)`, y todo lo que se les encadena) es un objeto que **describe una operación**, sin ejecutarla todavía — es la unidad fundamental de composición de toda la API, usada dentro de `select()`, `filter()`, `with_columns()`, `group_by().agg()`, etc.

```python
pl.col("precio")                    # expresión: "la columna precio"
pl.col("precio") * 1.16                # expresión: "la columna precio, multiplicada por 1.16"
pl.col("precio").mean()                  # expresión: "la media de la columna precio"
(pl.col("precio") * pl.col("cantidad")).alias("total")   # expresión compuesta, con nombre de salida explícito
```

Una expresión no hace nada por sí sola — se vuelve código ejecutable cuando se pasa dentro de un **contexto** (`select`, `filter`, `with_columns`...), que le dice a Polars **cómo** aplicarla sobre un DataFrame real.

## Los contextos: dónde se usan las expresiones

```python
df.select(pl.col("precio") * 1.16)                          # contexto SELECT: elegir/transformar columnas -> nuevo DataFrame
df.filter(pl.col("precio") > 100)                              # contexto FILTER: filtrar filas -> nuevo DataFrame
df.with_columns((pl.col("precio") * 1.16).alias("con_iva"))      # contexto WITH_COLUMNS: agregar/reemplazar columnas, conserva las demás
df.group_by("region").agg(pl.col("monto").sum())                  # contexto GROUP_BY/AGG: agregación por grupo
```

La **misma expresión** (`pl.col("precio") * 1.16`) significa algo distinto según el contexto en el que se use — esto es intencional: separa "qué operación hacer" (la expresión) de "en qué forma aplicarla" (el contexto), permitiendo reusar la misma lógica de transformación en distintos lugares.

## Múltiples expresiones en una sola llamada — paralelización automática

```python
df.select([
    pl.col("precio").mean().alias("precio_promedio"),
    pl.col("precio").max().alias("precio_maximo"),
    pl.col("stock").sum().alias("stock_total"),
])
```

Cuando se pasan varias expresiones independientes a `select()`/`with_columns()`, Polars las ejecuta **en paralelo** automáticamente usando varios hilos — sin que el usuario tenga que pensar en paralelismo explícitamente. Esta es una de las razones estructurales del rendimiento de Polars frente al bucle secuencial implícito de operaciones encadenadas en Pandas.

## Seleccionar columnas por patrón, no solo por nombre exacto

```python
pl.col("precio")                          # una columna específica
pl.col("precio", "stock")                   # varias columnas específicas
pl.col("^precio_.*$")                         # regex — todas las columnas que coincidan con el patrón
pl.col(pl.Float64)                              # TODAS las columnas de un dtype específico
pl.all()                                          # TODAS las columnas del DataFrame
pl.exclude("id")                                    # todas las columnas EXCEPTO estas
```

`pl.col(pl.Float64)` seleccionando por tipo de dato (en vez de por nombre) no tiene un equivalente tan directo en la sintaxis de Pandas — ahí normalmente se necesitaría `df.select_dtypes(include="float64")` como paso separado, mientras que en Polars es una expresión más que se combina naturalmente con el resto.

## Encadenar métodos sobre una expresión

```python
(
    pl.col("nombre")
    .str.to_uppercase()
    .str.strip_chars()
    .alias("nombre_limpio")
)
```

Igual que el accessor `.str` de Pandas (ver [[Python/Pandas/12 - Texto y Datos Categóricos|Python/Pandas]]), las expresiones de Polars exponen namespaces especializados (`.str`, `.dt`, `.list`, `.cat`) que se encadenan de la misma forma — la diferencia es que en Polars **todo**, no solo el texto, pasa por este mismo mecanismo unificado de expresiones.

## `pl.lit()` — valores literales dentro de una expresión

```python
df.with_columns((pl.col("precio") * pl.lit(1.16)).alias("con_iva"))     # equivalente a pl.col("precio") * 1.16
df.filter(pl.col("categoria") == pl.lit("Electrónica"))
```

`pl.lit()` rara vez es necesario explícitamente (Polars generalmente infiere el literal), pero es útil cuando se necesita ser explícito sobre el tipo de dato del literal o al construir expresiones dinámicamente en una función reutilizable.

## Ver también

- [[03 - Creación y Carga de Datos]]
- [[05 - Selección y Filtrado]]
- [[06 - Transformación de Columnas]]
- [[08 - Agrupación y Agregación]]
