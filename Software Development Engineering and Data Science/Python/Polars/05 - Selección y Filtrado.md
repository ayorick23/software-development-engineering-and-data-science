---
tags: [polars, python, data-science, filtering, cheat-sheet]
---

# 05 — Selección y Filtrado

> Continúa de [[04 - Expresiones (pl.col)]].

## `select()` — elegir y transformar columnas

```python
df.select("precio")                                  # una columna, devuelve DataFrame de una columna (no una Series suelta)
df.select(["precio", "stock"])                          # varias columnas
df.select(pl.col("precio"), pl.col("stock") * 2)          # con expresiones, incluye transformaciones
df.select(pl.all().exclude("id"))                            # todas menos una
```

**Diferencia clave con Pandas:** `df["precio"]` en Pandas devuelve una `Series`; `df.select("precio")` en Polars devuelve un `DataFrame` de una sola columna. Para obtener una `Series` suelta de Polars, se usa `df["precio"]` o `df.get_column("precio")` — pero el estilo idiomático de Polars favorece encadenar dentro de expresiones en vez de extraer columnas sueltas.

## Indexado directo — soportado, pero no es el estilo idiomático

```python
df["precio"]                 # SÍ funciona, devuelve una Series de Polars
df[0]                          # primera fila
df[0:5]                          # slice de filas
df[0, "precio"]                    # una celda específica (fila, columna)
```

Polars soporta indexado tipo Pandas por compatibilidad/comodidad exploratoria, pero el código de producción idiomático en Polars casi siempre usa `select()`/`filter()`/`with_columns()` con expresiones en vez de indexado directo — esto es lo que le permite al motor optimizar (ver [[13 - Optimización de Queries Lazy]]), algo que el indexado directo no puede aprovechar.

## `filter()` — filtrar filas

```python
df.filter(pl.col("precio") > 100)
df.filter((pl.col("precio") > 100) & (pl.col("stock") < 50))     # AND — paréntesis obligatorios, igual que en Pandas
df.filter(pl.col("region").is_in(["Norte", "Sur"]))
df.filter(pl.col("region").is_in(["Norte", "Sur"]).not_())          # negación explícita
```

## `is_between()` — rango inclusivo

```python
df.filter(pl.col("precio").is_between(50, 150))                # 50 <= precio <= 150 (inclusivo por default)
df.filter(pl.col("precio").is_between(50, 150, closed="left"))   # solo el extremo izquierdo inclusivo
```

## Filtrar con expresiones de texto

```python
df.filter(pl.col("producto").str.contains("(?i)pro"))         # (?i) al inicio del regex = case-insensitive
df.filter(pl.col("email").str.ends_with("@empresa.com"))
```

## `pl.when().then().otherwise()` — condicional vectorizado

```python
df.with_columns(
    pl.when(pl.col("precio") > 100)
    .then(pl.lit("Alto"))
    .otherwise(pl.lit("Bajo"))
    .alias("categoria_precio")
)

# Encadenar múltiples condiciones — equivalente a if/elif/elif/else
df.with_columns(
    pl.when(pl.col("precio") > 200).then(pl.lit("Premium"))
    .when(pl.col("precio") > 100).then(pl.lit("Alto"))
    .otherwise(pl.lit("Bajo"))
    .alias("categoria_precio")
)
```

`pl.when().then().otherwise()` es el equivalente directo de `np.select()`/`np.where()` de NumPy (ver [[Python/NumPy/06 - Indexado Avanzado|Python/NumPy]]), pero expresado como una expresión de Polars encadenable dentro de `select()`/`with_columns()` — es la forma idiomática de lógica condicional en Polars, en vez de `.apply()` con una función Python.

## Ver también

- [[04 - Expresiones (pl.col)]]
- [[06 - Transformación de Columnas]]
- [[Python/NumPy/06 - Indexado Avanzado|Python/NumPy]]
