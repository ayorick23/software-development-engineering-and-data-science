---
tags: [pandas, python, data-science, multiindex, cheat-sheet]
---

# 13 — MultiIndex y Datos Jerárquicos

> Continúa de [[12 - Texto y Datos Categóricos]].

## Qué es un `MultiIndex`

Un `MultiIndex` es un índice con **varios niveles** — permite representar datos con más de una dimensión de agrupación (ej. región + producto) sin necesidad de una columna combinada tipo `"Norte_ProductoA"`.

```python
df.groupby(["region", "producto"])["monto"].sum()
# region  producto
# Norte   A           1500
#         B            800
# Sur     A            600
# dtype: int64   <- esto YA es una Series con MultiIndex de 2 niveles
```

## Crear un `MultiIndex` explícitamente

```python
df = df.set_index(["region", "producto"])              # desde columnas existentes

arrays = [["Norte", "Norte", "Sur"], ["A", "B", "A"]]
idx = pd.MultiIndex.from_arrays(arrays, names=["region", "producto"])

idx = pd.MultiIndex.from_tuples([("Norte", "A"), ("Norte", "B"), ("Sur", "A")], names=["region", "producto"])

idx = pd.MultiIndex.from_product([["Norte", "Sur"], ["A", "B"]], names=["region", "producto"])   # todas las combinaciones
```

## Selección en un `MultiIndex`

```python
df.loc["Norte"]                        # todas las filas del primer nivel == "Norte"
df.loc[("Norte", "A")]                  # combinación exacta de ambos niveles
df.loc[("Norte", "A"):("Norte", "B")]   # rango (requiere índice ordenado con sort_index())

df.xs("A", level="producto")            # todas las filas donde el nivel 'producto' == "A", sin importar el otro nivel
df.xs(("Norte", "A"), level=["region", "producto"])
```

`xs()` (cross-section) es la herramienta específica para seleccionar por un nivel intermedio sin tener que especificar todos los niveles anteriores explícitamente en un `.loc[...]`.

## Reordenar y manipular niveles

```python
df.swaplevel("region", "producto")          # invierte el orden de dos niveles
df.sort_index(level="region")                # ordena por un nivel específico
df.reset_index()                              # convierte TODOS los niveles del índice de vuelta a columnas
df.reset_index(level="producto")              # convierte solo un nivel específico
```

## `unstack()` — de MultiIndex de filas a columnas

```python
serie_multiindex = df.groupby(["region", "producto"])["monto"].sum()
serie_multiindex.unstack()          # 'producto' pasa de nivel de índice a columnas -> tabla wide
serie_multiindex.unstack(level=0)   # especifica cuál nivel "sube" a columnas
```

Este es, en la práctica, el patrón más común para llegar de un `groupby` multi-clave a una tabla dinámica legible sin usar `pivot_table` directamente — ver también [[10 - Reshaping y Pivoting]].

## Rendimiento: `MultiIndex` ordenado (lexsorted)

```python
df = df.sort_index()          # requerido para que el slicing por rango en MultiIndex funcione correctamente y sea rápido
df.index.is_lexsorted()        # (o df.index.is_monotonic_increasing en versiones recientes)
```

Operar sobre un `MultiIndex` **no ordenado** (`sort_index()` pendiente) puede lanzar `UnsortedIndexError` en ciertos slices, y siempre es más lento en búsquedas — es buena práctica llamar `sort_index()` inmediatamente después de construir un `MultiIndex` complejo.

## Columnas también pueden ser `MultiIndex`

```python
df.groupby("region").agg({"monto": ["sum", "mean"], "stock": "max"})
# columnas resultantes: MultiIndex [('monto', 'sum'), ('monto', 'mean'), ('stock', 'max')]

df.columns = ["_".join(col).strip("_") for col in df.columns]   # aplanar a nombres simples: 'monto_sum', 'monto_mean'...
```

Aplanar columnas `MultiIndex` a strings simples después de un `agg()` con múltiples funciones es una práctica común para evitar tener que manejar tuplas al referenciar columnas después.

## Ver también

- [[04 - Selección e Indexación]]
- [[08 - Agrupación y Agregación (GroupBy)]]
- [[10 - Reshaping y Pivoting]]
