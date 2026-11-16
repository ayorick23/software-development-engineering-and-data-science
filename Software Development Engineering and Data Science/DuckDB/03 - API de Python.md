---
tags: [duckdb, python, api, relational-api, cheat-sheet]
---

# 03 — API de Python

> Continúa de [[02 - Instalación, CLI y Bases de Datos]]. Python es el cliente más usado de DuckDB. Esta nota cubre las conexiones, cómo ejecutar SQL, la **Relational API** (encadenar operaciones como en un DataFrame), cómo obtener resultados en distintos formatos y las consultas parametrizadas. La integración zero-copy con Pandas, Polars y Arrow tiene su propia nota: [[11 - Integración con Pandas, Polars y Arrow]].

## Conexiones

### La conexión por defecto (módulo)

Las funciones a nivel de módulo (`duckdb.sql`, `duckdb.read_csv`, etc.) usan una **conexión global en memoria** compartida por todo el proceso. Es cómoda en notebooks y scripts rápidos.

```python
import duckdb

duckdb.sql("SELECT 42 AS respuesta").show()
duckdb.sql("CREATE TABLE t AS SELECT range AS i FROM range(10)")
duckdb.sql("SELECT SUM(i) FROM t").fetchone()   # (45,)
```

### Conexiones explícitas

En scripts de producción, librerías y tests, crea **siempre una conexión explícita**. Así no hay estado global oculto.

```python
con = duckdb.connect()                          # en memoria, privada
con = duckdb.connect("analitica.duckdb")        # persistente
con = duckdb.connect("prod.duckdb", read_only=True)
con = duckdb.connect(config={"threads": 4, "memory_limit": "4GB"})

# Como context manager: cierra la conexión (y hace checkpoint) al salir
with duckdb.connect("analitica.duckdb") as con:
    con.sql("CREATE TABLE IF NOT EXISTS logs (ts TIMESTAMP, msg VARCHAR)")
```

Ver [[15 - Context Manager]] para el patrón `with`.

### Threads y cursores

Un objeto `DuckDBPyConnection` **no es thread-safe** si lo comparten varios hilos. Para concurrencia **dentro del mismo proceso**, cada hilo debe usar su propio cursor:

```python
from concurrent.futures import ThreadPoolExecutor

con = duckdb.connect("analitica.duckdb")

def contar(tabla: str) -> int:
    cur = con.cursor()               # conexión hija que comparte la misma base
    return cur.sql(f"SELECT COUNT(*) FROM {tabla}").fetchone()[0]

with ThreadPoolExecutor(4) as ex:
    print(list(ex.map(contar, ["ventas", "clientes", "productos"])))
```

`con.cursor()` devuelve una nueva conexión a la **misma** base de datos. Varios procesos distintos son otro tema: ver [[10 - DDL, Transacciones y Concurrencia#Concurrencia]]. Para fundamentos de hilos, ver [[23 - Concurrency]].

---

## `sql()` vs. `execute()`

| Método | Devuelve | Ejecución | Parámetros | Uso típico |
|---|---|---|---|---|
| `con.sql(query)` | `DuckDBPyRelation` (si es `SELECT`) | **Lazy**: se ejecuta al pedir el resultado | `params=` opcional | Consultas analíticas, encadenar con Relational API |
| `con.execute(query, params)` | La propia conexión (con resultado pendiente) | **Inmediata** | Sí (`?` o `$nombre`) | DML/DDL, consultas parametrizadas, `fetch*` |
| `con.executemany(query, lista)` | Conexión | Inmediata, una vez por fila de parámetros | Sí | Insertar pocas filas (es lento para muchas) |

```python
# sql(): relación lazy, se puede seguir encadenando o consultando
rel = con.sql("SELECT * FROM 'ventas.parquet' WHERE monto > 100")
print(rel.columns, rel.types)
rel.show()                      # imprime un preview (no materializa todo)

# execute(): se ejecuta ya
con.execute("INSERT INTO logs VALUES (now(), 'inicio')")
filas = con.execute("SELECT * FROM logs").fetchall()
```

> **Relaciones como tablas:** una relación asignada a una variable de Python se puede consultar por su nombre en SQL. DuckDB la encuentra mediante *replacement scan*, igual que hace con los DataFrames:
>
> ```python
> grandes = con.sql("SELECT * FROM 'ventas.parquet' WHERE monto > 1000")
> con.sql("SELECT region, COUNT(*) FROM grandes GROUP BY ALL").show()
> ```

---

## Obtener resultados

| Método | Resultado |
|---|---|
| `.fetchall()` | `list[tuple]` |
| `.fetchone()` | `tuple` o `None` |
| `.fetchmany(n)` | `list[tuple]` con hasta `n` filas (para iterar en bloques) |
| `.df()` / `.fetchdf()` | `pandas.DataFrame` |
| `.pl()` | `polars.DataFrame` |
| `.arrow()` / `.fetch_arrow_table()` | `pyarrow.Table` |
| `.fetch_record_batch(n)` | `pyarrow.RecordBatchReader` (streaming por lotes) |
| `.fetchnumpy()` | `dict[str, numpy.ndarray]` |
| `.torch()` / `.tf()` | Diccionario de tensores de PyTorch / TensorFlow |
| `.show()` | Imprime un preview, no devuelve nada |

```python
q = "SELECT region, SUM(monto) AS total FROM ventas GROUP BY ALL"

con.sql(q).df()          # Pandas
con.sql(q).pl()          # Polars
con.sql(q).fetchall()    # [('LATAM', 1234.5), ...]

# Procesar un resultado enorme sin cargarlo entero en memoria
reader = con.sql("SELECT * FROM 'gigante.parquet'").fetch_record_batch(100_000)
for batch in reader:
    procesar(batch.to_pandas())
```

---

## Consultas parametrizadas

**Nunca** construyas SQL con f-strings para *valores* que vienen del usuario o de fuera. Usa parámetros: evitas SQL injection y errores de comillas o tipos. Ver [[62 - Seguridad en Python]].

```python
# Posicionales con ?
con.execute("SELECT * FROM ventas WHERE region = ? AND monto > ?", ["LATAM", 500]).df()

# Con nombre usando $nombre
con.execute(
    "SELECT * FROM ventas WHERE fecha BETWEEN $ini AND $fin",
    {"ini": "2026-01-01", "fin": "2026-06-30"},
).df()

# También en sql()
con.sql("SELECT * FROM ventas WHERE region = $r", params={"r": "EMEA"})

# Prepared statements implícitos: executemany
con.executemany("INSERT INTO logs VALUES (?, ?)", [(ts1, "a"), (ts2, "b")])
```

> Los parámetros sirven para **valores**, no para **identificadores** (nombres de tabla o columna). Si el nombre de tabla es dinámico, valídalo contra una lista blanca antes de interpolarlo.

> **Rendimiento:** `executemany` con miles de filas es lento porque ejecuta fila por fila. Para cargas masivas, arma un DataFrame o tabla Arrow e inserta con `INSERT INTO t SELECT * FROM df`. Ver [[15 - Rendimiento y Optimización#Cargas masivas]].

---

## Relational API

La **Relational API** permite construir consultas encadenando métodos en Python, en lugar de escribir el SQL completo. Cada método devuelve una nueva `DuckDBPyRelation` lazy, y DuckDB optimiza todo el árbol junto al ejecutar. Es útil para construir consultas de forma programática, sin concatenar strings.

### Crear relaciones

```python
rel = con.table("ventas")                     # tabla existente
rel = con.view("v_ventas")
rel = con.read_parquet("ventas/*.parquet")
rel = con.read_csv("clientes.csv", header=True)
rel = con.read_json("eventos.ndjson")
rel = con.from_df(df_pandas)                  # o simplemente con.sql("FROM df_pandas")
rel = con.values([1, "a"])                    # una fila literal
rel = con.sql("SELECT * FROM range(100)")
```

### Transformar

```python
resultado = (
    con.read_parquet("ventas/*.parquet")
       .filter("fecha >= DATE '2026-01-01'")
       .project("region, producto_id, monto, monto * 0.18 AS igv")
       .aggregate("region, SUM(monto) AS total, SUM(igv) AS igv_total", "region")
       .order("total DESC")
       .limit(10)
)
resultado.show()
print(resultado.sql_query())   # SQL equivalente que generó
resultado.explain()            # plan de ejecución
```

| Método | Equivalente SQL |
|---|---|
| `.filter("cond")` | `WHERE` |
| `.project("exprs")` / `.select(...)` | `SELECT` |
| `.aggregate("exprs", "grupos")` | `SELECT ... GROUP BY` |
| `.order("col DESC")` | `ORDER BY` |
| `.limit(n, offset)` | `LIMIT / OFFSET` |
| `.distinct()` | `SELECT DISTINCT` |
| `.join(otra, "cond", how="left")` | `JOIN` (`inner`, `left`, `right`, `outer`, `semi`, `anti`) |
| `.union(otra)` / `.except_(otra)` / `.intersect(otra)` | Operaciones de conjuntos |
| `.set_alias("a")` | Alias para usar en joins |
| `.cross(otra)` | `CROSS JOIN` |
| `.count("*")`, `.sum("col")`, `.avg(...)`, `.max(...)` | Agregados rápidos |
| `.query("nombre_virtual", "SQL sobre nombre_virtual")` | Mezclar SQL arbitrario |

```python
pedidos  = con.table("pedidos").set_alias("p")
clientes = con.table("clientes").set_alias("c")
pedidos.join(clientes, "p.cliente_id = c.id", how="left") \
       .project("c.nombre, p.total") \
       .show()
```

### Materializar o escribir

```python
rel.create("ventas_resumen")             # CREATE TABLE ... AS
rel.create_view("v_resumen")             # CREATE VIEW
rel.insert_into("historico")             # INSERT INTO historico SELECT ...
rel.write_parquet("resumen.parquet", compression="zstd")
rel.write_csv("resumen.csv", header=True)
rel.to_table("t")                        # alias de create
```

### ¿SQL o Relational API?

- **SQL** cuando la consulta es fija y legible como texto. Es lo más común, y además es portable a dbt u otros motores.
- **Relational API** cuando la consulta se **arma dinámicamente**: filtros opcionales según parámetros, columnas elegidas en tiempo de ejecución o funciones que reciben y devuelven relaciones.

```python
def filtrar(rel, region=None, min_monto=None):
    if region:
        rel = rel.filter(f"region = '{region}'")   # valida 'region' antes (lista blanca)
    if min_monto is not None:
        rel = rel.filter(f"monto >= {float(min_monto)}")
    return rel
```

### Expression API (avanzado)

Para no escribir strings, existe una API de expresiones similar a la de Polars:

```python
from duckdb import ColumnExpression as col, ConstantExpression as lit, FunctionExpression as fn

rel = con.table("ventas")
rel.filter(col("monto") > lit(100)) \
   .select(col("region"), fn("upper", col("producto")).alias("producto")) \
   .show()
```

Es menos madura y menos usada que el SQL, pero útil en librerías que generan consultas.

---

## Atajos de lectura a nivel de módulo

```python
duckdb.read_csv("datos.csv").df()
duckdb.read_parquet(["a.parquet", "b.parquet"]).pl()
duckdb.sql("FROM 'datos.json'").show()
duckdb.query("SELECT 1")          # alias de sql()
```

## Manejo de errores

Las excepciones viven en el módulo `duckdb` y heredan de `duckdb.Error`:

```python
try:
    con.execute("SELECT * FROM tabla_inexistente")
except duckdb.CatalogException as e:      # tabla/columna no existe
    print("Catálogo:", e)
except duckdb.ParserException as e:       # error de sintaxis
    print("Sintaxis:", e)
except duckdb.ConversionException as e:   # cast inválido
    print("Conversión:", e)
except duckdb.ConstraintException as e:   # PK/UNIQUE/NOT NULL violado
    print("Constraint:", e)
except duckdb.TransactionException as e:  # conflicto de escritura concurrente
    print("Transacción:", e)
except duckdb.IOException as e:           # archivo no encontrado o bloqueado
    print("I/O:", e)
```

Ver [[11 - Exception Handling]] y [[49 - Excepciones Avanzadas]].

## Ejemplo integrador: mini-ETL en un script

```python
import duckdb
from pathlib import Path

def etl(origen: Path, destino: Path, desde: str) -> int:
    with duckdb.connect() as con:
        con.execute("SET memory_limit = '4GB'")
        rel = con.sql(
            """
            SELECT cliente_id,
                   CAST(fecha AS DATE)           AS fecha,
                   trim(lower(email))            AS email,
                   round(monto, 2)               AS monto
            FROM read_csv($src, header = true)
            WHERE fecha >= $desde
            """,
            params={"src": str(origen / "*.csv"), "desde": desde},
        )
        rel.write_parquet(str(destino), compression="zstd")
        return con.sql(f"SELECT COUNT(*) FROM '{destino}'").fetchone()[0]

print(etl(Path("raw"), Path("clean/ventas.parquet"), "2026-01-01"))
```

## Ver también

- [[02 - Instalación, CLI y Bases de Datos]]
- [[04 - Leer y Escribir CSV y JSON]]
- [[11 - Integración con Pandas, Polars y Arrow]]
- [[13 - UDFs en Python y Funciones Personalizadas]]
- [[24 - Databases]] (en `Python/`): DB-API y conectores de bases en Python
- [[Python/Polars/16 - Integración con el Ecosistema|Polars/16]]: el lado de Polars en la integración con DuckDB
