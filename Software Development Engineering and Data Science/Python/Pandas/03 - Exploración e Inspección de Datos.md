---
tags: [pandas, python, data-science, eda, cheat-sheet]
---

# 03 — Exploración e Inspección de Datos

> Continúa de [[02 - Creación y Carga de Datos]]. El primer paso ante cualquier DataFrame nuevo, antes de filtrar o transformar nada.

## Vista rápida de la forma y contenido

```python
df.head(10)          # primeras 10 filas
df.tail(5)            # últimas 5 filas
df.sample(5, random_state=42)   # muestra aleatoria — mejor que head() para detectar patrones no visibles al inicio
df.shape               # (n_filas, n_columnas)
len(df)                 # número de filas
df.columns              # Index con nombres de columna
df.T                    # transponer (útil para ver muchas columnas en pantallas angostas)
```

## `info()` — dtypes, nulos y memoria de un vistazo

```python
df.info()
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 10000 entries, 0 to 9999
# Data columns (total 4 columns):
#  #   Column     Non-Null Count  Dtype
# ---  ------     --------------  -----
#  0   fecha      10000 non-null  datetime64[ns]
#  1   producto   9850 non-null   object     <- 150 nulos aquí
#  2   monto      10000 non-null  float64
#  3   region     10000 non-null  category
# memory usage: 312.6+ KB

df.info(memory_usage="deep")   # memoria REAL incluyendo objetos Python (más lento, más preciso)
```

`memory_usage="deep"` es necesario para columnas `object` (strings): sin `deep=True`, Pandas solo cuenta el tamaño de los punteros, no el contenido real de los strings — subestima brutalmente la memoria usada. Ver optimización de memoria en [[15 - Rendimiento y Optimización]].

## `describe()` — estadísticas descriptivas

```python
df.describe()                          # solo columnas numéricas por default
df.describe(include="all")             # incluye columnas object/category también
df.describe(include=[np.number])       # explícito
df.describe(percentiles=[.1, .5, .9])  # percentiles custom en vez de 25/50/75

df["monto"].describe()                 # sobre una sola Series
```

## Tipos de dato y selección por tipo

```python
df.dtypes
df.select_dtypes(include="number")           # solo columnas numéricas
df.select_dtypes(include=["object", "category"])
df.select_dtypes(exclude="datetime64[ns]")
```

## Conteos y valores únicos

```python
df["region"].value_counts()                  # frecuencia de cada valor, ordenado descendente
df["region"].value_counts(normalize=True)    # como proporción (0-1) en vez de conteo
df["region"].nunique()                        # número de valores únicos
df["region"].unique()                         # arreglo con los valores únicos (sin ordenar por frecuencia)
df.nunique()                                   # nunique por columna, para todo el DataFrame
```

## Correlaciones y resúmenes numéricos

```python
df.corr(numeric_only=True)                    # matriz de correlación de Pearson
df.corr(method="spearman")
df["monto"].mean(), df["monto"].median(), df["monto"].std()
df.agg(["mean", "std", "min", "max"])         # varias estadísticas de una sola vez
```

## Detección rápida de calidad de datos

```python
df.isna().sum()                    # nulos por columna
df.isna().mean() * 100             # % de nulos por columna
df.duplicated().sum()               # filas duplicadas exactas
df[df.duplicated(keep=False)]       # ver TODAS las filas involucradas en duplicados (no solo las repetidas)
```

Este bloque de chequeos es, en la práctica, el primer paso antes de decidir estrategia de nulos ([[07 - Datos Nulos y Duplicados]]) o de validar el esquema con [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]] antes de que el DataFrame entre a un pipeline de producción.

## Ver también

- [[02 - Creación y Carga de Datos]]
- [[04 - Selección e Indexación]]
- [[07 - Datos Nulos y Duplicados]]
- [[15 - Rendimiento y Optimización]]
- [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]] — validación formal de esquema, más allá de la exploración manual
