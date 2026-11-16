---
tags: [duckdb, pandas, polars, arrow, numpy, integracion, cheat-sheet]
---

# 11 — Integración con Pandas, Polars y Arrow

> Continúa de [[10 - DDL, Transacciones y Concurrencia]]. DuckDB puede consultar con SQL los **DataFrames que ya tienes en memoria**, muchas veces **sin copiarlos**, y devolver resultados en el formato que prefieras. Esto lo convierte en un complemento natural de Pandas y Polars, no en un reemplazo. Desde el lado de Polars, ver [[Python/Polars/16 - Integración con el Ecosistema|Polars/16]]; desde el lado de Pandas, [[Python/Pandas/15 - Rendimiento y Optimización|Pandas/15]].

## Replacement scans: consultar variables de Python por su nombre

Cuando DuckDB no encuentra una tabla con un nombre dado, busca una **variable de Python** con ese nombre en el scope local y luego en el global. Si es un DataFrame de Pandas o Polars, una tabla o dataset de Arrow, un array de NumPy o una relación de DuckDB, la consulta directamente.

```python
import duckdb
import pandas as pd
import polars as pl

ventas_pd = pd.read_csv("ventas.csv")
clientes_pl = pl.read_parquet("clientes.parquet")

duckdb.sql("""
    SELECT c.segmento, SUM(v.monto) AS total
    FROM ventas_pd v
    JOIN clientes_pl c ON v.cliente_id = c.id
    GROUP BY ALL
""").pl()
```

Un join entre un DataFrame de Pandas y uno de Polars, en SQL, y con el resultado en Polars.

### Registro explícito

El replacement scan es cómodo, pero "mágico": depende de nombres de variables. En código de producción, sobre todo en funciones donde la variable no está en el scope esperado, conviene registrar el DataFrame explícitamente:

```python
con = duckdb.connect()
con.register("ventas", ventas_pd)            # vista virtual sobre el DataFrame (sin copia)
con.sql("SELECT COUNT(*) FROM ventas").fetchone()
con.unregister("ventas")

# Desde la relational API
rel = con.from_df(ventas_pd)
rel = con.from_arrow(tabla_arrow)
```

Los replacement scans se pueden desactivar con `SET python_enable_replacements = false`.

---

## Copia vs. zero-copy

| Fuente | ¿Copia al leer? | Detalle |
|---|---|---|
| **Arrow** (`pa.Table`, `RecordBatchReader`, `pyarrow.dataset`) | **No** | Formato columnar compatible; DuckDB lo escanea en sitio |
| **Polars** | **No** (en la práctica) | Polars usa Arrow internamente |
| **Pandas con dtypes NumPy** | Mayormente no para numéricos | Los strings `object` sí requieren conversión |
| **Pandas con dtypes Arrow** (`string[pyarrow]`, `pd.ArrowDtype`) | Muy barato | Recomendado desde Pandas 2.x |
| **NumPy** (dict de arrays) | Numéricos sin copia | |

Y al **devolver** resultados:

| Método | Costo |
|---|---|
| `.arrow()` | El más barato: el resultado ya es columnar |
| `.pl()` | Barato (pasa por Arrow) |
| `.df()` | Convierte a NumPy/Pandas: algo más caro, sobre todo con strings |
| `.fetchall()` | El más caro: crea objetos Python fila por fila |

Regla práctica: **mantén los datos en formatos Arrow el mayor tiempo posible** (DuckDB ↔ Polars ↔ PyArrow) y convierte a Pandas solo al final, cuando una librería lo exija.

---

## Pandas

```python
df = pd.read_parquet("ventas.parquet")

# Consultar
top = duckdb.sql("SELECT * FROM df ORDER BY monto DESC LIMIT 10").df()

# Crear una tabla persistente desde un DataFrame
con = duckdb.connect("analitica.duckdb")
con.execute("CREATE OR REPLACE TABLE ventas AS SELECT * FROM df")
con.execute("INSERT INTO ventas SELECT * FROM df_nuevas")      # carga masiva rápida

# Resultado a Pandas con dtypes Arrow (menos memoria, strings eficientes)
df2 = con.sql("FROM ventas").df()
df3 = con.sql("FROM ventas").arrow().to_pandas(types_mapper=pd.ArrowDtype)
```

### Cuándo DuckDB le gana a Pandas

| Operación | Pandas | DuckDB |
|---|---|---|
| `groupby` sobre 50M filas | Un solo hilo, mucha RAM | Paralelo, con spilling a disco |
| Join de dos DataFrames grandes | `merge` puede quedarse sin memoria | Hash join paralelo, larger-than-memory |
| Leer 200 CSVs y filtrar | Bucle + `concat` | `FROM read_csv('*.csv') WHERE ...` |
| Window functions complejas | `groupby().transform()` y `rolling` encadenados | Una sola consulta con `OVER` |
| Leer solo 3 columnas de un Parquet en S3 | Descarga completa | Range requests con pushdown |

Y cuándo **no**: manipulación fila a fila, `apply` con lógica de Python arbitraria, integración directa con sklearn o Matplotlib, o DataFrames pequeños donde Pandas ya es instantáneo.

### Patrón híbrido típico

```python
# 1. DuckDB hace el trabajo pesado (leer, filtrar, unir, agregar)
features = duckdb.sql("""
    SELECT cliente_id,
           count(*)                         AS n_compras,
           sum(monto)                       AS gasto_total,
           date_diff('day', max(fecha), current_date) AS recencia
    FROM read_parquet('s3://lake/ventas/**/*.parquet')
    GROUP BY ALL
""").df()

# 2. Pandas/sklearn hacen lo que saben hacer
from sklearn.cluster import KMeans
features["cluster"] = KMeans(n_clusters=4, n_init="auto").fit_predict(
    features[["n_compras", "gasto_total", "recencia"]]
)
```

Ver [[Python/Scikit-learn/01 - Introducción, Filosofía y la API Consistente|Scikit-learn]].

---

## Polars

```python
import polars as pl

lf = pl.scan_parquet("ventas/*.parquet")          # LazyFrame
df = lf.filter(pl.col("monto") > 0).collect()

# SQL de DuckDB sobre el DataFrame de Polars, resultado en Polars
res = duckdb.sql("""
    SELECT region, quantile_cont(monto, 0.9) AS p90
    FROM df GROUP BY ALL
""").pl()

# Resultado como LazyFrame para seguir encadenando en Polars (versiones recientes)
lazy = duckdb.sql("FROM df").pl(lazy=True)
```

DuckDB también puede consultar un `LazyFrame` directamente; en ese caso Polars lo materializa primero.

**¿Polars o DuckDB para cada paso?**

| Prefiere DuckDB | Prefiere Polars |
|---|---|
| Lógica compleja que se expresa mejor en SQL (window, PIVOT, ASOF, CTEs) | Transformaciones encadenadas con la API de expresiones |
| Persistir resultados (tablas, transacciones) | Pipelines puramente en memoria y en Python |
| Leer o escribir desde S3 con secretos, Iceberg, Postgres | Integración con otras librerías de DataFrames |
| Datos más grandes que la RAM (spilling robusto) | Streaming engine de Polars para casos similares |
| Equipo que sabe SQL | Equipo que prefiere código Python tipado |

Ver [[Python/Polars/17 - Buenas Prácticas, Errores Comunes y Comparativa|Polars/17]].

---

## Apache Arrow

```python
import pyarrow as pa
import pyarrow.dataset as ds
import pyarrow.parquet as pq

tabla = pq.read_table("ventas.parquet")
duckdb.sql("SELECT COUNT(*) FROM tabla").fetchone()

# Datasets de Arrow particionados: DuckDB empuja filtros y proyecciones al dataset
dataset = ds.dataset("lake/ventas", format="parquet", partitioning="hive")
duckdb.sql("SELECT SUM(monto) FROM dataset WHERE anio = 2026").fetchone()

# Streaming: RecordBatchReader entra y sale
reader = duckdb.sql("FROM 'grande.parquet'").fetch_record_batch(50_000)
pq.write_table(pa.Table.from_batches(reader), "copia.parquet")
```

Arrow es el "idioma común" que permite mover datos entre DuckDB, Polars, Pandas 2, Spark, Snowflake y otros sin serializar. Además, **ADBC** (*Arrow Database Connectivity*) es el estándar emergente para conectores que devuelven Arrow directamente, y DuckDB ofrece un driver ADBC.

---

## NumPy y tensores

```python
import numpy as np

arr = {"x": np.random.randn(1_000_000), "g": np.random.randint(0, 5, 1_000_000)}
duckdb.sql("SELECT g, avg(x), stddev(x) FROM arr GROUP BY g").show()   # dict de arrays

# Resultado como arrays de NumPy (por columna)
cols = duckdb.sql("SELECT x, g FROM arr").fetchnumpy()
X = np.column_stack([cols["x"], cols["g"]])

# Directo a PyTorch / TensorFlow
tensores = duckdb.sql("SELECT x, g FROM arr").torch()   # dict nombre -> torch.Tensor
```

Ver [[Python/NumPy/01 - Introducción y Arquitectura Interna|NumPy]].

## Otros puentes del ecosistema

| Herramienta | Integración |
|---|---|
| **Ibis** | API de DataFrames portable cuyo backend por defecto es DuckDB; el mismo código corre luego en BigQuery o Snowflake |
| **SQLAlchemy** | `duckdb-engine`: `create_engine("duckdb:///analitica.duckdb")` para usar `pd.read_sql` u ORMs |
| **JupySQL** | Magias `%sql` / `%%sql` en notebooks con DuckDB |
| **Narwhals** | Capa de compatibilidad entre librerías de DataFrames, con soporte para relaciones DuckDB |
| **Streamlit** | DuckDB como motor de consulta de dashboards (ver [[Python/Streamlit/01 - Introducción y Modelo de Ejecución|Streamlit]]) |

```python
# SQLAlchemy y Pandas
from sqlalchemy import create_engine
engine = create_engine("duckdb:///analitica.duckdb")
df = pd.read_sql("SELECT * FROM ventas LIMIT 100", engine)
```

```python
# Ibis: API de DataFrames compilada a SQL de DuckDB
import ibis
con = ibis.duckdb.connect("analitica.duckdb")
t = con.table("ventas")
expr = t.group_by("region").aggregate(total=t.monto.sum()).order_by(ibis.desc("total"))
print(ibis.to_sql(expr))
expr.to_pandas()
```

## Trampas comunes

- **Strings `object` en Pandas**: la conversión es lenta y consume memoria. Usa `dtype_backend="pyarrow"` al leer con Pandas.
- **Índice de Pandas**: DuckDB **ignora el índice**. Si contiene información (por ejemplo, una fecha), haz `df.reset_index()` antes.
- **Nombres de columna con espacios o mayúsculas**: en SQL hay que citarlos con comillas dobles: `SELECT "Monto Total" FROM df`.
- **Tipos mezclados en una columna `object`**: DuckDB infiere a partir de una muestra y puede fallar más adelante. Castea en Pandas antes de consultar.
- **Mutar el DataFrame tras registrarlo**: la vista apunta al objeto. Vuelve a registrarlo si lo reasignas.
- **Fechas con zona horaria**: `datetime64[ns, tz]` llega como `TIMESTAMPTZ`; revisa `SET TimeZone`.

## Ver también

- [[03 - API de Python]]
- [[Python/Polars/16 - Integración con el Ecosistema|Polars/16]] y [[Python/Pandas/16 - Integración con el Ecosistema|Pandas/16]]
- [[13 - UDFs en Python y Funciones Personalizadas]]
- [[15 - Rendimiento y Optimización]]
- [[07-Librerias-de-Data-Science-y-ML]] (en `Machine Learning/`)
