---
tags: [pandas, python, data-science, performance, cheat-sheet]
---

# 15 — Rendimiento y Optimización

> Continúa de [[14 - Exportación de Datos]]. El resumen práctico de "por qué esto es más lento" mencionado a lo largo de todo este cheat-sheet.

## Jerarquía de velocidad: vectorización > `apply` > loops

```python
# MÁS LENTO: loop explícito fila por fila
total = []
for i in range(len(df)):
    total.append(df.iloc[i]["precio"] * df.iloc[i]["cantidad"])
df["total"] = total

# LENTO: iterrows() — reconstruye una Series por fila, con overhead de tipos
for idx, fila in df.iterrows():
    df.at[idx, "total"] = fila["precio"] * fila["cantidad"]

# MEJOR: itertuples() — namedtuples, sin overhead de Series, pero sigue siendo un loop de Python
for fila in df.itertuples():
    ...

# ACEPTABLE cuando no hay alternativa vectorizada: apply
df["total"] = df.apply(lambda f: f["precio"] * f["cantidad"], axis=1)

# ÓPTIMO: operación vectorizada en C/NumPy por debajo
df["total"] = df["precio"] * df["cantidad"]
```

La diferencia de rendimiento entre la primera y la última forma puede ser de **órdenes de magnitud** (100x-1000x en DataFrames grandes) porque la versión vectorizada ejecuta el bucle en código C compilado de NumPy, no en el intérprete de Python.

## Diagnóstico de memoria

```python
df.memory_usage(deep=True)                    # bytes reales por columna, incluyendo contenido de objetos Python
df.memory_usage(deep=True).sum() / 1024**2    # total en MB

df.info(memory_usage="deep")
```

## Downcasting de dtypes

```python
df["stock"] = pd.to_numeric(df["stock"], downcast="integer")    # int64 -> int32/int16 si los valores caben
df["precio"] = pd.to_numeric(df["precio"], downcast="float")     # float64 -> float32

df["region"] = df["region"].astype("category")                    # ver 12 — ahorro grande en columnas de baja cardinalidad
```

Downcasting reduce memoria a costa de rango representable — verificar que ningún valor futuro exceda el rango del tipo más pequeño antes de aplicarlo en un pipeline de producción.

## Backend PyArrow (`dtype_backend`)

```python
df = pd.read_csv("datos.csv", dtype_backend="pyarrow", engine="pyarrow")
df = df.convert_dtypes(dtype_backend="pyarrow")   # convertir un DataFrame ya existente
```

El backend Arrow (disponible desde pandas 2.0) generalmente usa **menos memoria** que NumPy para strings y soporta nulos nativos en cualquier dtype (no solo `float`), además de interoperar sin copia con otras herramientas del ecosistema Arrow (Polars, DuckDB, Spark).

## `eval()` / `query()` y el motor `numexpr`

```python
df.eval("total = precio * cantidad", engine="numexpr")
df.query("precio > 100 and stock < 50", engine="numexpr")
```

En DataFrames grandes (cientos de miles de filas+), `eval`/`query` con el motor `numexpr` evitan la creación de arreglos intermedios completos en memoria para cada paso de una expresión compuesta — ganancia real solo se nota a partir de cierto volumen; en DataFrames pequeños el overhead de invocar `numexpr` puede hacerlo más lento que la sintaxis normal.

## Copy-on-Write (CoW)

```python
pd.set_option("mode.copy_on_write", True)   # pandas 2.x explícito — default desde pandas 3.0
```

Con CoW activo, cualquier operación que "parecería" devolver una vista en realidad copia de forma perezosa solo cuando se modifica — esto elimina la ambigüedad clásica de `SettingWithCopyWarning` (ver [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]) a costa de, en algunos casos, una copia adicional que no existía en el modo clásico. En la práctica, el código correcto (`.loc` en vez de indexación encadenada) se comporta igual con o sin CoW.

## Procesamiento por lotes (`chunksize`) para datasets más grandes que la RAM

```python
resultado_parcial = []
for chunk in pd.read_csv("archivo_enorme.csv", chunksize=200_000):
    resultado_parcial.append(chunk.groupby("categoria")["monto"].sum())
resultado_final = pd.concat(resultado_parcial).groupby(level=0).sum()
```

## Cuándo Pandas ya no es la herramienta correcta

| Síntoma | Alternativa |
|---|---|
| El dataset no cabe en RAM de una sola máquina | [[16 - Integración con el Ecosistema#Dask y PySpark|Dask o PySpark]] |
| Se necesita paralelismo real entre núcleos para transformaciones | Dask, o Polars (motor multi-hilo por defecto) |
| Pipeline con validación de esquema formal antes de modelar | [[Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]] |
| Solo lectura/agregación ad-hoc de archivos Parquet/CSV enormes, sin cargar todo | DuckDB (consultas SQL directas sobre archivos) |

Comparativa detallada Pandas vs Polars vs Dask en [[17 - Buenas Prácticas, Errores Comunes y Comparativa]].

## Ver también

- [[02 - Creación y Carga de Datos]] — `chunksize`, `dtype_backend` en la carga
- [[06 - Operaciones con Columnas]] — vectorización vs `apply`
- [[Python/Pandas/16 - Integración con el Ecosistema]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
