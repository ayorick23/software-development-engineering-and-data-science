---
tags: [polars, python, data-science, strings, categorical, cheat-sheet]
---

# 12 — Texto y Categóricos

> Continúa de [[11 - Series de Tiempo]].

## El namespace `.str`

```python
df.select(pl.col("nombre").str.to_uppercase())
df.select(pl.col("nombre").str.to_lowercase())
df.select(pl.col("nombre").str.strip_chars())
df.select(pl.col("nombre").str.len_chars())
df.select(pl.col("email").str.split("@").list.get(0))     # split devuelve una LISTA; .list.get(0) extrae el primer elemento
df.select(pl.col("nombre").str.replace(" ", "_"))            # reemplaza la PRIMERA coincidencia
df.select(pl.col("nombre").str.replace_all(" ", "_"))          # reemplaza TODAS las coincidencias
```

**Diferencia con Pandas a notar:** `str.replace()` de Polars reemplaza solo la primera coincidencia por default (similar a `str.replace()` nativo de Python) — para reemplazar todas las coincidencias se necesita explícitamente `replace_all()`. En Pandas, `.str.replace()` reemplaza todas por default.

## Expresiones regulares

```python
df.filter(pl.col("producto").str.contains(r"^SKU-\d+$"))
df.select(pl.col("email").str.extract(r"(\w+)@(\w+)", group_index=1))    # extrae un grupo específico de captura
df.select(pl.col("texto").str.extract_all(r"\d+"))                          # todas las coincidencias, como una lista por celda
```

## `split()` devuelve listas — el dtype `List`

```python
df.with_columns(pl.col("nombre_completo").str.split(" ").alias("partes"))
# dtype resultante: List[Utf8] — cada celda contiene una LISTA de strings, no columnas separadas

df.with_columns(
    pl.col("nombre_completo").str.split(" ").list.get(0).alias("nombre"),
    pl.col("nombre_completo").str.split(" ").list.get(1).alias("apellido"),
)
```

A diferencia de `str.split(expand=True)` de Pandas (que crea columnas separadas directamente), Polars siempre devuelve un dtype `List` nativo — se extraen columnas individuales explícitamente con `.list.get(i)` después. El dtype `List` es un tipo de primera clase en Polars (ver también `explode()` en [[10 - Reshaping]]), no una lista de Python dentro de una celda `object` como en Pandas.

## Dtype `Categorical`

```python
df.with_columns(pl.col("region").cast(pl.Categorical))
```

Igual que el dtype `category` de Pandas (ver [[Python/Pandas/12 - Texto y Datos Categóricos#Dtype category — cuándo y por qué|Python/Pandas]]): reduce memoria en columnas de baja cardinalidad codificando internamente como enteros con un mapa de categorías.

## `Enum` — categórico con categorías fijas y ordenadas

```python
tallas = pl.Enum(["S", "M", "L", "XL"])
df.with_columns(pl.col("talla").cast(tallas))

df.filter(pl.col("talla") > "M")     # comparación válida según el ORDEN declarado en el Enum
```

**Diferencia clave con `Categorical`:** `pl.Enum` requiere declarar el conjunto **completo y fijo** de categorías válidas por adelantado (un valor fuera de ese conjunto lanza error al castear) y preserva un orden explícito para comparaciones — `pl.Categorical` es más flexible (acepta cualquier valor, categorías descubiertas dinámicamente) pero no garantiza esa validación ni ese orden. `Enum` es el equivalente más cercano a un `CategoricalDtype(categories=[...], ordered=True)` de Pandas (ver [[Python/Pandas/12 - Texto y Datos Categóricos#Categorías ordenadas|Python/Pandas]]).

## `cut()` — discretizar variables numéricas

```python
df.with_columns(
    pl.col("edad").cut([18, 35, 60], labels=["Menor", "Joven", "Adulto", "Mayor"]).alias("rango_edad")
)
```

Equivalente a `pd.cut()` de Pandas — ver [[Python/Pandas/12 - Texto y Datos Categóricos#cut() y qcut() — discretizar variables numéricas en categorías|Python/Pandas]].

## Ver también

- [[11 - Series de Tiempo]]
- [[10 - Reshaping]]
- [[Python/Pandas/12 - Texto y Datos Categóricos|Python/Pandas]]
