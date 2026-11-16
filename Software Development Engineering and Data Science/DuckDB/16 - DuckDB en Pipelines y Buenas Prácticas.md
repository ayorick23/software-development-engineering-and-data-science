---
tags: [duckdb, dbt, airflow, pytest, pipelines, buenas-practicas, cheat-sheet]
---

# 16 — DuckDB en Pipelines y Buenas Prácticas

> Continúa de [[15 - Rendimiento y Optimización]] y cierra el cheat-sheet. Aquí DuckDB se integra con el resto del stack: **dbt-duckdb**, **tareas de Airflow**, **tests con pytest**, notebooks, Streamlit y Docker. Al final hay una lista de errores comunes y un checklist de producción.

## dbt-duckdb

DuckDB es el adapter más cómodo para aprender dbt, correr CI sin warehouse o llevar proyectos pequeños y medianos a producción. La configuración básica de `profiles.yml` está en [[02 - Instalación, Proyecto y profiles.yml|dbt/02]]; aquí van las opciones avanzadas.

```yaml
# profiles.yml
mi_proyecto:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: dev.duckdb
      threads: 4
      extensions: [httpfs, parquet, spatial]
      settings:
        memory_limit: 4GB
        preserve_insertion_order: false
      secrets:
        - type: s3
          provider: credential_chain
      attach:
        - path: s3://lake/ref/catalogo.duckdb
          alias: catalogo
          read_only: true
      external_root: s3://lake/marts          # destino de los modelos 'external'
      plugins:
        - module: mi_proyecto.udfs             # registrar UDFs de Python (ver nota 13)
```

### Sources directamente sobre archivos

```yaml
# models/sources.yml
sources:
  - name: raw
    meta:
      external_location: "s3://lake/raw/{name}/*.parquet"   # {name} = nombre de la tabla
    tables:
      - name: ventas
      - name: clientes
        meta:
          external_location: "read_csv('s3://lake/raw/clientes/*.csv', header = true)"
```

```sql
-- models/staging/stg_ventas.sql: lee los Parquet sin cargarlos antes
SELECT * EXCLUDE (_ingest_ts), CAST(fecha AS DATE) AS fecha
FROM {{ source('raw', 'ventas') }}
```

### Materialización `external`: escribir el modelo como Parquet

```sql
-- models/marts/fct_ventas.sql
{{ config(
    materialized = 'external',
    location     = 's3://lake/marts/fct_ventas',
    format       = 'parquet',
    options      = {'partition_by': 'anio', 'compression': 'zstd'}
) }}
SELECT *, year(fecha) AS anio FROM {{ ref('int_ventas_enriquecidas') }}
```

Con esto, **dbt + DuckDB funciona como motor de transformación de un data lake**: lee Parquet, transforma con SQL y escribe Parquet, sin warehouse de por medio. Las materializaciones están en [[06 - Materializaciones|dbt/06]] y los incrementales en [[07 - Modelos Incrementales|dbt/07]].

---

## DuckDB dentro de Airflow

Un patrón muy común es usar DuckDB como **motor de cómputo efímero dentro de una tarea**: la tarea lee de S3 o Postgres, transforma y escribe Parquet. No hay estado compartido entre tareas, así que no hay problemas de locks.

```python
# dags/ventas_diarias.py  (Airflow 3, TaskFlow API)
from airflow.sdk import dag, task, Asset
import pendulum

ventas_silver = Asset("s3://lake/silver/ventas")

@dag(schedule="@daily", start_date=pendulum.datetime(2026, 1, 1, tz="America/Lima"), catchup=False)
def ventas_diarias():

    @task(outlets=[ventas_silver])
    def transformar(logical_date=None):
        import duckdb
        # En Airflow 3, un cron como "@daily" usa CronTriggerTimetable por defecto:
        # logical_date = momento del disparo (medianoche), así que procesamos el día anterior.
        dia = logical_date.subtract(days=1).strftime("%Y-%m-%d")
        with duckdb.connect() as con:
            con.execute("SET memory_limit = '2GB'")
            con.execute("CREATE SECRET (TYPE s3, PROVIDER credential_chain)")
            con.execute(f"""
                COPY (
                    SELECT * EXCLUDE (raw_payload), CAST(ts AS DATE) AS fecha
                    FROM read_json('s3://lake/raw/ventas/fecha={dia}/*.json')
                    WHERE monto IS NOT NULL
                ) TO 's3://lake/silver/ventas/fecha={dia}'
                  (FORMAT parquet, OVERWRITE_OR_IGNORE, FILENAME_PATTERN 'part_{{i}}')
            """)
            return con.sql(f"SELECT COUNT(*) FROM 's3://lake/silver/ventas/fecha={dia}/*.parquet'").fetchone()[0]

    @task
    def validar(n: int):
        if n == 0:
            raise ValueError("Partición vacía")

    validar(transformar())

ventas_diarias()
```

Claves del patrón:

- **Idempotencia**: cada ejecución escribe **su partición** (`fecha={dia}`) y la sobreescribe. Reintentar o hacer backfill no duplica datos. Con una base `.duckdb` se logra lo mismo con `DELETE` + `INSERT` en una transacción (ver [[10 - DDL, Transacciones y Concurrencia#Transacciones]]).
- **Pasa referencias, no datos**: la tarea devuelve un conteo o una ruta, nunca un DataFrame enorme por XCom.
- **Si varias tareas escriben el mismo `.duckdb`**: serialízalas con un **pool de tamaño 1** (`pool="duckdb_writer"`), o mejor, que escriban Parquet o DuckLake.
- **`memory_limit` explícito**: el worker de Airflow comparte la máquina con otras tareas.

La parte de Airflow se desarrolla en [[Python/Airflow/01 - Introducción y Arquitectura|Python/Airflow]]: pools en [[Python/Airflow/16 - Executors, Escalado y Despliegue#Pools: proteger recursos compartidos|Airflow/16]] y semántica de fechas en [[Python/Airflow/10 - Scheduling, Timetables y Backfills|Airflow/10]].

---

## Testing con pytest

DuckDB en memoria es ideal para **testear lógica SQL** rápido y sin infraestructura.

```python
# tests/conftest.py
import duckdb
import pytest

@pytest.fixture
def con():
    con = duckdb.connect()              # en memoria: aislado por test
    con.execute("""
        CREATE TABLE ventas AS SELECT * FROM (VALUES
            (1, 'LATAM', DATE '2026-01-01', 100.0),
            (2, 'LATAM', DATE '2026-01-01', 100.0),   -- duplicado de negocio
            (3, 'EMEA',  DATE '2026-01-02', -5.0)     -- monto inválido
        ) t(id, region, fecha, monto)
    """)
    yield con
    con.close()
```

```python
# tests/test_transformaciones.py
from pathlib import Path

SQL = Path("sql/ventas_limpias.sql").read_text()

def test_elimina_montos_negativos(con):
    res = con.sql(SQL).df()
    assert (res["monto"] >= 0).all()

def test_agrega_por_region(con):
    res = dict(con.sql(f"SELECT region, SUM(monto) FROM ({SQL}) GROUP BY ALL").fetchall())
    assert res == {"LATAM": 200.0}

def test_esquema_resultado(con):
    tipos = dict(zip(con.sql(SQL).columns, map(str, con.sql(SQL).types)))
    assert tipos["fecha"] == "DATE"
```

Usos típicos:

- Testear archivos `.sql` de un pipeline con datos sintéticos.
- **Validar datos reales** en un paso del pipeline (conteos, nulos, unicidad) con consultas `SELECT COUNT(*) ... WHERE condición_inválida`.
- Combinar con **Pandera** sobre `con.sql(...).pl()` o `.df()` para validar esquemas (ver [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]]).
- **Unit tests de dbt** sobre DuckDB en CI (ver [[11 - Tests de Datos y Unit Tests|dbt/11]]).

Ver [[Python/Pytest/01 - Introducción y Arquitectura|Pytest]] para fixtures y parametrización.

---

## Notebooks, Streamlit y BI

### Jupyter con JupySQL

```python
%pip install jupysql duckdb-engine
%load_ext sql
%sql duckdb:///analitica.duckdb
```

```sql
%%sql resultado <<
SELECT region, SUM(monto) AS total FROM ventas GROUP BY ALL
```

```python
resultado.DataFrame().plot.bar(x="region", y="total")
```

### Streamlit

```python
import duckdb
import streamlit as st

@st.cache_resource
def get_con():
    return duckdb.connect("analitica.duckdb", read_only=True)   # read_only: varios usuarios

@st.cache_data(ttl=600)
def consultar(region: str):
    return get_con().execute(
        "SELECT fecha, SUM(monto) AS total FROM ventas WHERE region = ? GROUP BY ALL ORDER BY fecha",
        [region],
    ).df()

region = st.selectbox("Región", ["LATAM", "EMEA", "NA"])
st.line_chart(consultar(region), x="fecha", y="total")
```

Usa `read_only=True` y una conexión cacheada con `@st.cache_resource`; en cada hilo usa `.cursor()` si hay concurrencia. Ver [[Python/Streamlit/01 - Introducción y Modelo de Ejecución|Streamlit]].

### BI

- **Power BI / Tableau**: conecta por ODBC (driver ODBC de DuckDB) o, más robusto, publica tablas *gold* en Parquet o en MotherDuck. Ver la carpeta `Power BI/` y `Tableau/`.
- **Evidence, Rill, Metabase, Superset**: soporte nativo de DuckDB.

---

## Docker

```dockerfile
FROM python:3.12-slim
RUN pip install --no-cache-dir duckdb==1.4.* pyarrow
# Pre-instalar extensiones para no depender de internet en runtime
RUN python -c "import duckdb; [duckdb.sql(f'INSTALL {e}') for e in ('httpfs','aws','postgres')]"
ENV DUCKDB_MEMORY_LIMIT=2GB
COPY pipeline/ /app/pipeline/
WORKDIR /app
CMD ["python", "-m", "pipeline.main"]
```

```python
# pipeline/db.py: punto único de creación de conexiones
import os
import duckdb

def get_connection(path: str = ":memory:", read_only: bool = False) -> duckdb.DuckDBPyConnection:
    con = duckdb.connect(path, read_only=read_only, config={
        "memory_limit": os.getenv("DUCKDB_MEMORY_LIMIT", "2GB"),
        "threads": int(os.getenv("DUCKDB_THREADS", "4")),
        "temp_directory": os.getenv("DUCKDB_TEMP", "/tmp/duckdb"),
    })
    con.execute("CREATE SECRET IF NOT EXISTS (TYPE s3, PROVIDER credential_chain)")
    # registrar UDFs y macros comunes aquí
    return con
```

Ver [[Docker/02 - Dockerfile en Profundidad|Docker/02]].

---

## Compactación de archivos pequeños

Las escrituras frecuentes (por hora, por evento) producen muchos Parquet pequeños que degradan las lecturas, sobre todo en S3. Conviene programar una compactación periódica:

```sql
COPY (
    FROM read_parquet('s3://lake/silver/eventos/fecha=2026-11-14/*.parquet')
    ORDER BY ts
) TO 's3://lake/silver_compact/eventos/fecha=2026-11-14'
  (FORMAT parquet, COMPRESSION zstd, FILE_SIZE_BYTES '256MB', OVERWRITE_OR_IGNORE);
-- luego intercambia prefijos y borra los originales en un paso controlado
```

Con DuckLake: `CALL ducklake_merge_adjacent_files('lake')` (ver [[14 - DuckLake, MotherDuck y el Lakehouse Ligero#Mantenimiento]]).

---

## Errores comunes

| Error / síntoma | Causa | Solución |
|---|---|---|
| `Could not set lock on file` | Otro proceso (DBeaver, notebook, UI) tiene la base abierta en escritura | Cierra el otro proceso; usa `read_only`; escribe Parquet |
| Base creada con una versión nueva no abre en otra vieja | Formato de almacenamiento más nuevo | Iguala las versiones o usa `EXPORT`/`IMPORT DATABASE` |
| Contenedor OOM-killed | `memory_limit` calculado sobre la RAM del host | Fija `memory_limit` |
| Tipos inferidos mal en CSV (códigos con ceros perdidos) | Sniffer con muestra pequeña | `types = {...}`, `all_varchar` o `sample_size = -1` |
| Filas perdidas en silencio | `ignore_errors = true` | Usa `store_rejects = true` y audita |
| Consultas lentas sobre S3 | Miles de archivos pequeños o egress entre regiones | Compacta y ejecuta en la misma región |
| Medición de tiempo engañosa en Python | `con.sql()` es lazy | Materializa con `.fetchall()` o `.df()` |
| Variable de Python "no encontrada" en SQL | El replacement scan no ve la variable (otro scope) | `con.register("nombre", df)` |
| El archivo `.duckdb` no se achica tras un `DELETE` | No hay compactación automática | `COPY FROM DATABASE` a una base nueva |
| Duplicados al reintentar una tarea | `INSERT` no idempotente | Delete + insert por partición en una transacción, u `OVERWRITE` de la partición |
| Resultado distinto entre DuckDB y el warehouse de prod | Friendly SQL o funciones no portables | Usa macros cross-database de dbt o prueba en ambos |

## Checklist de producción

- [ ] Versión de DuckDB **fijada** e igual en dev, CI y prod.
- [ ] **Una** función `get_connection()` con `memory_limit`, `threads`, `temp_directory`, secretos y UDFs.
- [ ] Credenciales vía `credential_chain` o variables de entorno; nunca en el código.
- [ ] Extensiones **pre-instaladas** en la imagen.
- [ ] Esquemas de entrada **explícitos** en CSV y JSON; rechazos auditados.
- [ ] Escrituras **idempotentes** (partición o transacción).
- [ ] **Un solo escritor** por archivo `.duckdb`; los demás en `read_only` o leyendo Parquet.
- [ ] Datos **ordenados y particionados** por las columnas de filtro; archivos de 100 MB a 1 GB.
- [ ] Compactación y expiración de snapshots programadas (si usas DuckLake).
- [ ] Tests de la lógica SQL con pytest y DuckDB en memoria; validación de datos en el pipeline.
- [ ] `*.duckdb` y `*.wal` fuera de Git; datos versionados con DVC o snapshots de DuckLake si importa la reproducibilidad.

## Ver también

- [[01 - Introducción y Arquitectura]]: mapa completo del cheat-sheet
- [[15 - dbt en Producción - CI-CD y Orquestación|dbt/15]] y [[16 - Python Models, Ecosistema y Buenas Prácticas|dbt/16]]
- [[50-Orquestacion-Prefect-y-Airflow]] (en `Machine Learning/`) y [[Python/Airflow/15 - Airflow con dbt, Docker, Kubernetes y MLflow|Airflow/15]]
- [[13-Testing-en-Machine-Learning]] (en `Machine Learning/`)
- [[Python/Pytest/01 - Introducción y Arquitectura|Pytest]], [[Python/Streamlit/01 - Introducción y Modelo de Ejecución|Streamlit]], [[Docker/02 - Dockerfile en Profundidad|Docker/02]]
- [[04-Ingenieria-de-Datos]] (en `Machine Learning/`)
