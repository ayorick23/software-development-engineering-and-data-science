---
tags: [polars, python, data-science, query-optimization, cheat-sheet]
---

# 13 — Optimización de Queries Lazy

> Continúa de [[12 - Texto y Categóricos]]. El detalle técnico detrás de por qué [[02 - Eager vs Lazy API|la API lazy]] es más rápida, no solo "diferente".

## El optimizador de consultas

Cuando se llama `.collect()` sobre un `LazyFrame`, Polars no ejecuta el plan tal cual fue escrito — primero lo pasa por un **optimizador de consultas** que reordena y simplifica las operaciones antes de ejecutarlas, de forma similar a como un motor de bases de datos relacional optimiza una consulta SQL antes de ejecutarla.

```python
lf = pl.scan_csv("ventas.csv").select(["fecha", "monto"]).filter(pl.col("monto") > 100)
print(lf.explain())      # muestra el plan YA optimizado
```

## Predicate Pushdown — empujar filtros lo antes posible

```python
(
    pl.scan_parquet("ventas.parquet")
    .filter(pl.col("region") == "Norte")     # el optimizador mueve este filtro para aplicarlo DURANTE la lectura del archivo
    .group_by("producto")
    .agg(pl.col("monto").sum())
    .collect()
)
```

Aunque el `.filter()` esté escrito **después** de la lectura en el código, el optimizador detecta que puede aplicarse antes (durante la lectura del archivo Parquet, que soporta filtrado a nivel de row-group) — reduciendo drásticamente cuántos datos hay que leer y procesar realmente.

## Projection Pushdown — leer solo las columnas necesarias

```python
(
    pl.scan_parquet("ventas.parquet")     # el archivo tiene 50 columnas
    .select(["fecha", "monto"])              # el optimizador detecta que SOLO estas 2 se necesitan
    .collect()
)                                              # y lee solo esas 2 columnas del archivo Parquet, ignorando las otras 48
```

Esto es posible porque Parquet es un formato **columnar** — leer columnas específicas sin tocar las demás es una operación nativa del formato, no algo que Polars tenga que simular después de cargar todo. Con CSV (formato de fila), esta optimización es más limitada porque hay que leer la fila completa de todas formas.

## Eliminación de operaciones redundantes

```python
lf = (
    pl.scan_csv("ventas.csv")
    .select(["fecha", "monto", "region"])
    .select(["fecha", "monto"])          # esta segunda selección hace que 'region' nunca se haya necesitado
)
print(lf.explain())     # el optimizador fusiona ambos select() en uno solo, leyendo solo fecha y monto desde el inicio
```

## Comparar el plan optimizado vs sin optimizar

```python
lf = pl.scan_csv("ventas.csv").filter(pl.col("monto") > 100).select(["fecha", "monto"])

print("--- SIN optimizar ---")
print(lf.explain(optimized=False))

print("--- Optimizado ---")
print(lf.explain(optimized=True))
```

Comparar ambos plans es la forma más directa de entender **qué** está optimizando Polars en un caso concreto — útil al depurar por qué una consulta lazy es más rápida (o, en casos raros, para detectar cuando el optimizador no pudo aplicar una optimización esperada).

## Slice Pushdown — limitar filas leídas cuando se usa `.head()`/`.limit()`

```python
pl.scan_csv("archivo_enorme.csv").head(100).collect()     # el optimizador puede dejar de leer el archivo tras las primeras 100 filas
```

Relevante para exploración rápida de archivos gigantes: `.head(n)` combinado con lazy evita procesar el archivo completo solo para mostrar una vista previa — algo que `pd.read_csv().head()` de Pandas no puede hacer (Pandas ya cargó todo el archivo antes de que `.head()` actuara).

## `collect_schema()` — inspeccionar el esquema resultante sin ejecutar

```python
lf.collect_schema()     # devuelve los nombres y dtypes de las columnas del resultado, SIN materializar ningún dato
```

Útil para validar que una cadena larga de transformaciones produce el esquema esperado antes de pagar el costo de ejecutar el `.collect()` completo sobre datos reales.

## Ver también

- [[02 - Eager vs Lazy API]]
- [[14 - Streaming y Datasets Grandes]]
- [[03 - Creación y Carga de Datos]]
