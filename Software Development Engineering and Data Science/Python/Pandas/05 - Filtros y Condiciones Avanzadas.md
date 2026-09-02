---
tags: [pandas, python, data-science, filtering, cheat-sheet]
---

# 05 — Filtros y Condiciones Avanzadas

> Continúa de [[04 - Selección e Indexación]].

## `query()` — filtrar con sintaxis de expresión en texto

```python
df.query("precio > 100 and stock < 50")
df.query("region in ['Norte', 'Sur']")
df.query("precio > @umbral")            # @ referencia una variable Python externa
```

`query()` es más legible que encadenar `&`/`|` con corchetes cuando la condición tiene varias partes, y en DataFrames grandes puede ser más rápido porque usa el motor `numexpr` internamente (evalúa la expresión completa en C sin crear arreglos booleanos intermedios). Ver comparación de rendimiento en [[15 - Rendimiento y Optimización]].

## `isin()` — pertenencia a un conjunto de valores

```python
df[df["region"].isin(["Norte", "Sur"])]
df[~df["region"].isin(["Norte", "Sur"])]     # NOT isin — el resto de las regiones
```

## `between()` — rango inclusivo

```python
df[df["precio"].between(50, 150)]                    # 50 <= precio <= 150
df[df["fecha"].between("2026-01-01", "2026-03-31")]  # también funciona con fechas
```

## Filtrado con el accessor `.str`

```python
df[df["producto"].str.contains("Pro", case=False, na=False)]   # na=False evita error si hay nulos
df[df["email"].str.match(r"^[\w.]+@empresa\.com$")]             # regex completo
df[df["producto"].str.startswith("SKU-")]
```

Catálogo completo del accessor `.str` en [[12 - Texto y Datos Categóricos]].

## `np.where` y `np.select` — crear columnas condicionales

```python
df["categoria_precio"] = np.where(df["precio"] > 100, "Alto", "Bajo")   # if/else vectorizado

df["categoria_precio"] = np.select(
    condlist=[df["precio"] > 200, df["precio"] > 100],
    choicelist=["Premium", "Alto"],
    default="Bajo",
)   # equivalente vectorizado a if/elif/elif/else
```

`np.select`/`np.where` son la forma **vectorizada** de lógica condicional para crear columnas — muchísimo más rápidos que `.apply(lambda row: ...)` (ver [[06 - Operaciones con Columnas]] y [[15 - Rendimiento y Optimización]] para el porqué).

## `.mask()` y `.where()` — reemplazo condicional in-place en la Series

```python
df["precio"].where(df["precio"] > 0, other=0)     # mantiene valor donde la condición es True, reemplaza donde es False
df["precio"].mask(df["precio"] < 0, other=0)       # exactamente lo opuesto: reemplaza donde la condición es True
```

`.where()` y `.mask()` son complementarios: `.where(cond)` conserva donde `cond` es `True`; `.mask(cond)` conserva donde `cond` es `False`.

## Combinar múltiples condiciones legibles

```python
condicion_alto_valor = df["precio"] > 100
condicion_bajo_stock = df["stock"] < 20
condicion_region_valida = df["region"].isin(["Norte", "Sur"])

df[condicion_alto_valor & condicion_bajo_stock & condicion_region_valida]
```

Guardar condiciones booleanas complejas en variables con nombre descriptivo (en vez de una sola línea con muchos `&`/`|`) mejora legibilidad sin costo de rendimiento — Pandas no re-evalúa nada distinto.

## Ver también

- [[04 - Selección e Indexación]]
- [[06 - Operaciones con Columnas]]
- [[12 - Texto y Datos Categóricos]]
- [[15 - Rendimiento y Optimización]]
