---
tags: [duckdb, s3, httpfs, cloud, secrets, cheat-sheet]
---

# 06 — Fuentes Remotas y Cloud

> Continúa de [[05 - Parquet, Globs y Particionado Hive]]. Los mismos `read_parquet`, `read_csv` y `COPY ... TO` funcionan sobre **HTTP(S), S3, Google Cloud Storage, Azure Blob/ADLS, Cloudflare R2 y Hugging Face**. Esta nota cubre la extensión `httpfs`, el **Secrets Manager** para credenciales y cuánto cuesta (en tiempo y dinero) consultar datos remotos.

## La extensión `httpfs`

`httpfs` se **autocarga** la primera vez que usas una URL `http://`, `https://`, `s3://`, `gcs://`, `r2://` o `hf://`. Si el entorno no tiene internet al momento de la consulta (por ejemplo, un contenedor *air-gapped*), instálala antes:

```sql
INSTALL httpfs;
LOAD httpfs;
```

Azure usa su propia extensión (`azure`), que también se autocarga con `az://` o `abfss://`.

## HTTP(S): archivos públicos

```sql
SELECT * FROM 'https://raw.githubusercontent.com/org/repo/main/data/titanic.csv';

SELECT COUNT(*)
FROM read_parquet('https://datos.ejemplo.org/viajes/2026-01.parquet');
```

Con Parquet sobre HTTP, DuckDB usa **HTTP range requests**: descarga solo el footer y las columnas y row groups que necesita, no el archivo entero. Por eso un `SELECT COUNT(*)` sobre un Parquet remoto de 2 GB puede tardar menos de un segundo.

---

## Secrets Manager: credenciales

Desde la versión 0.10, las credenciales se gestionan con **`CREATE SECRET`**. Es mejor que los viejos `SET s3_access_key_id=...`, porque permite tener varias credenciales a la vez, cada una asociada a un *scope* (prefijo de bucket).

### S3 con claves explícitas

```sql
CREATE SECRET s3_prod (
    TYPE s3,
    KEY_ID 'AKIA...',
    SECRET '...',
    REGION 'us-east-1'
);

SELECT * FROM 's3://mi-bucket/ventas/*.parquet';
```

### S3 con la cadena de credenciales de AWS (recomendado)

En lugar de escribir claves, usa las del entorno: variables `AWS_*`, `~/.aws/credentials`, perfiles SSO, el rol IAM de la instancia o del pod. Requiere la extensión `aws`, que se autocarga.

```sql
CREATE SECRET aws_default (
    TYPE s3,
    PROVIDER credential_chain
);

-- Perfil específico
CREATE SECRET aws_analytics (
    TYPE s3,
    PROVIDER credential_chain,
    CHAIN 'sso;config',
    PROFILE 'analytics'
);
```

Este es el patrón correcto en producción: el contenedor o el pod obtiene credenciales temporales de su rol IAM y **ningún secreto vive en el código**. Ver [[13 - Managed Kubernetes, CI-CD y Buenas Prácticas|Kubernetes/13]] para IRSA y Workload Identity.

### Varias credenciales con `SCOPE`

```sql
CREATE SECRET lake_lectura (
    TYPE s3, PROVIDER credential_chain, PROFILE 'lector',
    SCOPE 's3://data-lake'
);
CREATE SECRET sandbox_escritura (
    TYPE s3, KEY_ID '...', SECRET '...', REGION 'us-west-2',
    SCOPE 's3://sandbox-equipo'
);

-- DuckDB elige el secreto cuyo SCOPE es el prefijo más largo que coincide
FROM which_secret('s3://sandbox-equipo/x.parquet', 's3');
```

### Otros proveedores

```sql
-- Google Cloud Storage (HMAC keys o credential_chain)
CREATE SECRET gcs (TYPE gcs, KEY_ID '...', SECRET '...');
SELECT * FROM 'gs://bucket/datos/*.parquet';

-- Cloudflare R2
CREATE SECRET r2 (TYPE r2, KEY_ID '...', SECRET '...', ACCOUNT_ID '...');
SELECT * FROM 'r2://bucket/archivo.parquet';

-- S3 compatible (MinIO, Ceph...) con endpoint propio
CREATE SECRET minio (
    TYPE s3, KEY_ID 'minio', SECRET 'minio123',
    ENDPOINT 'localhost:9000', URL_STYLE 'path', USE_SSL false
);

-- Azure
CREATE SECRET az (TYPE azure, CONNECTION_STRING 'DefaultEndpointsProtocol=...');
-- o PROVIDER credential_chain / service_principal
SELECT * FROM 'az://contenedor/datos/*.parquet';
SELECT * FROM 'abfss://contenedor@cuenta.dfs.core.windows.net/datos/*.parquet';

-- Hugging Face datasets
SELECT * FROM 'hf://datasets/usuario/dataset/data/train-*.parquet';
CREATE SECRET hf (TYPE huggingface, TOKEN 'hf_...');   -- para datasets privados
```

### Secretos temporales vs. persistentes

```sql
CREATE SECRET ...            -- TEMPORARY por defecto: vive solo en la sesión
CREATE PERSISTENT SECRET s3_dev (TYPE s3, PROVIDER credential_chain);
-- Se guarda en ~/.duckdb/stored_secrets SIN cifrar: úsalo solo en tu máquina

FROM duckdb_secrets();       -- lista secretos (los valores salen redactados)
DROP SECRET s3_dev;
DROP PERSISTENT SECRET s3_dev;
```

> **Buenas prácticas de seguridad:** no escribas claves en notebooks que se suben a Git. Usa `credential_chain`, o lee las claves desde variables de entorno en Python y pásalas al crear el secreto:
>
> ```python
> import os
> con.execute(f"""
>   CREATE SECRET (TYPE s3, KEY_ID '{os.environ["AWS_ACCESS_KEY_ID"]}',
>                  SECRET '{os.environ["AWS_SECRET_ACCESS_KEY"]}', REGION 'us-east-1')
> """)
> ```
>
> Ver [[62 - Seguridad en Python]].

---

## Escribir a la nube

```sql
COPY (SELECT * FROM ventas WHERE fecha = current_date - 1)
TO 's3://data-lake/silver/ventas'
   (FORMAT parquet, PARTITION_BY (fecha), COMPRESSION zstd, APPEND);
```

- S3 no tiene "renombrar" atómico, así que una escritura interrumpida puede dejar archivos parciales. Usa `FILENAME_PATTERN '{uuid}'` y escribe primero en un prefijo temporal, o bien un formato de tabla con transacciones (Iceberg, Delta, DuckLake).
- DuckDB sube los archivos grandes en *multipart uploads* automáticamente.

## Listar y explorar buckets

```sql
FROM glob('s3://data-lake/silver/ventas/**/*.parquet');      -- lista objetos

SELECT filename, COUNT(*) AS filas
FROM read_parquet('s3://data-lake/silver/ventas/**/*.parquet', filename = true)
GROUP BY ALL ORDER BY filename;
```

---

## Rendimiento y costo de leer datos remotos

| Factor | Impacto | Qué hacer |
|---|---|---|
| **Latencia por request** | Cada archivo implica varios requests (footer y luego columnas) | Menos archivos y más grandes (100 MB–1 GB) |
| **Listado de objetos** (`**`) | Listar miles de prefijos es lento | Usa particionado y filtra por la columna de partición |
| **Egress** | La nube cobra los datos que salen de la región | Corre DuckDB **en la misma región** que el bucket (EC2, Lambda, pod) |
| **Requests GET** | S3 cobra por request | Evita miles de archivos diminutos |
| **Releer lo mismo** | Repetir la descarga en cada consulta | Materializa localmente: `CREATE TABLE t AS FROM 's3://...'`, o activa el caché |

```sql
-- Paralelismo de I/O: con datos remotos conviene más threads que núcleos
SET threads = 16;

-- Caché HTTP de metadata y objetos dentro de la sesión
SET enable_http_metadata_cache = true;
SET enable_external_file_cache = true;   -- caché en memoria de bloques remotos (1.3+)

-- Reintentos ante errores transitorios
SET http_retries = 5;
SET http_timeout = 60000;   -- en ms
```

### Patrón típico: "traer y trabajar local"

```python
con = duckdb.connect("trabajo.duckdb")
con.execute("CREATE SECRET (TYPE s3, PROVIDER credential_chain)")
con.execute("""
    CREATE OR REPLACE TABLE ventas_2026 AS
    SELECT * FROM read_parquet('s3://data-lake/silver/ventas/anio=2026/**/*.parquet')
""")
# A partir de aquí todas las consultas son locales y rápidas
```

## DuckDB-Wasm: en el navegador

DuckDB también compila a **WebAssembly**. El motor corre dentro del navegador y lee Parquet remotos por HTTP. Así funcionan muchas herramientas de BI ligeras y playgrounds (por ejemplo, el shell en `shell.duckdb.org`). Es útil para dashboards estáticos sin backend.

## Ver también

- [[05 - Parquet, Globs y Particionado Hive]]
- [[12 - Extensiones]]: `httpfs`, `aws` y `azure` son extensiones
- [[14 - DuckLake, MotherDuck y el Lakehouse Ligero]]: un data lake con transacciones en S3
- [[16 - DuckDB en Pipelines y Buenas Prácticas]]
- [[08-Plataformas-de-Datos-y-ML]] (en `Machine Learning/`): almacenamiento cloud (S3, GCS, ADLS)
- [[Python/DVC/01 - Introducción y Arquitectura Interna|DVC]]: remotos S3 para versionar datos
