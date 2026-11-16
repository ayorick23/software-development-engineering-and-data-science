---
tags: [duckdb, extensiones, postgres, sqlite, spatial, iceberg, delta, vss, fts, cheat-sheet]
---

# 12 — Extensiones

> Continúa de [[11 - Integración con Pandas, Polars y Arrow]]. El núcleo de DuckDB es pequeño a propósito, y casi todo lo "extra" vive en **extensiones**: acceso a S3, otras bases de datos, formatos de tabla (Iceberg, Delta), datos geoespaciales, búsqueda de texto completo, búsqueda vectorial, Excel y mucho más. Esta nota explica cómo gestionarlas y recorre las más útiles.

## Gestión de extensiones

```sql
INSTALL spatial;          -- descarga el binario (una vez por versión de DuckDB, a ~/.duckdb/extensions)
LOAD spatial;             -- lo carga en la sesión actual

FROM duckdb_extensions(); -- nombre, loaded, installed, versión, descripción
SELECT extension_name, loaded, installed FROM duckdb_extensions() WHERE installed;

UPDATE EXTENSIONS;        -- actualizar las instaladas
FORCE INSTALL spatial;    -- reinstalar

-- Community extensions (repositorio comunitario)
INSTALL h3 FROM community;
LOAD h3;
```

### Autoloading

Muchas extensiones **core** se instalan y cargan solas cuando usas una función suya: `httpfs` con `s3://`, `json` con `read_json`, `parquet`, `icu` con zonas horarias, `excel` con `read_xlsx`, etc. Otras, como `spatial` o `vss`, requieren `INSTALL`/`LOAD` explícitos.

```sql
SET autoinstall_known_extensions = false;   -- desactivar en entornos controlados
SET autoload_known_extensions = false;
```

> **Entornos sin internet (producción, Docker):** instala las extensiones al construir la imagen para que no se descarguen en runtime:
>
> ```dockerfile
> RUN python -c "import duckdb; [duckdb.sql(f'INSTALL {e}') for e in ['httpfs','aws','spatial','postgres']]"
> ```
>
> Ver [[Docker/02 - Dockerfile en Profundidad|Docker/02]]. También puedes apuntar a un directorio propio con `SET extension_directory = '/opt/duckdb_ext'`.

### Mapa de extensiones

| Extensión | Para qué | Tipo |
|---|---|---|
| `httpfs`, `aws`, `azure` | HTTP, S3, GCS, R2, Azure (ver [[06 - Fuentes Remotas y Cloud]]) | core |
| `parquet`, `json` | Formatos (incluidas por defecto) | core |
| `icu` | Zonas horarias, collations | core |
| `excel` | `read_xlsx`, `COPY ... (FORMAT xlsx)` | core |
| `postgres`, `mysql`, `sqlite` | `ATTACH` a otras bases | core |
| `iceberg`, `delta` | Formatos de tabla de lakehouse | core |
| `ducklake` | Formato de lakehouse de DuckDB (ver [[14 - DuckLake, MotherDuck y el Lakehouse Ligero]]) | core |
| `spatial` | GIS: geometrías, `ST_*`, shapefiles, GeoJSON | core |
| `fts` | Búsqueda de texto completo (BM25) | core |
| `vss` | Índice HNSW para búsqueda vectorial | core |
| `inet` | Tipos de IP | core |
| `tpch`, `tpcds` | Generar datasets de benchmark | core |
| `ui` | UI web local | core |
| `motherduck` | Conexión a MotherDuck | core |
| `h3`, `prql`, `gsheets`, `bigquery`, `chsql`... | Comunidad | community |

---

## Otras bases de datos: `postgres`, `mysql` y `sqlite`

Con estas extensiones DuckDB funciona como **motor de federación**: consulta otras bases como si fueran propias, empuja filtros hacia ellas y cruza sus tablas con archivos locales.

### PostgreSQL

```sql
INSTALL postgres; LOAD postgres;

-- Credenciales vía secret
CREATE SECRET pg_prod (
    TYPE postgres, HOST 'db.empresa.com', PORT 5432,
    DATABASE 'erp', USER 'analista', PASSWORD '...'
);
ATTACH '' AS erp (TYPE postgres, SECRET pg_prod, READ_ONLY);

-- o con connection string
ATTACH 'dbname=erp host=localhost user=postgres' AS erp (TYPE postgres);

SHOW ALL TABLES;
SELECT * FROM erp.public.facturas WHERE fecha >= '2026-11-01';   -- el filtro se empuja a Postgres

-- Ejecutar SQL nativo de Postgres (con funciones propias de Postgres)
SELECT * FROM postgres_query('erp', 'SELECT id, data->>''x'' FROM eventos LIMIT 10');

-- Extraer a Parquet (ETL en una línea)
COPY (SELECT * FROM erp.public.facturas) TO 'facturas.parquet';

-- Copiar a DuckDB local para analizar rápido
CREATE TABLE facturas AS FROM erp.public.facturas;

-- También se puede ESCRIBIR en Postgres (sin READ_ONLY)
INSERT INTO erp.public.resumen SELECT region, sum(monto) FROM ventas GROUP BY ALL;
```

**Patrón muy útil:** analítica sobre una réplica de lectura de la base OLTP de producción sin montar un pipeline de extracción. Cuidado: las consultas pesadas pueden cargar esa base, así que úsalo sobre réplicas.

### MySQL y SQLite

```sql
INSTALL mysql; LOAD mysql;
ATTACH 'host=localhost user=root database=tienda' AS my (TYPE mysql, READ_ONLY);

INSTALL sqlite; LOAD sqlite;
ATTACH 'app.sqlite' AS app (TYPE sqlite);
SELECT * FROM app.usuarios;
-- o sin ATTACH
SELECT * FROM sqlite_scan('app.sqlite', 'usuarios');
```

---

## Iceberg y Delta Lake

Los **formatos de tabla abiertos** añaden, sobre archivos Parquet, transacciones, esquema y *time travel*. DuckDB los lee y (en versiones recientes) también los escribe.

```sql
-- Delta Lake (por ejemplo, tablas escritas por Databricks o Spark)
INSTALL delta; LOAD delta;
SELECT * FROM delta_scan('s3://lake/silver/ventas');

-- Iceberg: lectura directa de una tabla
INSTALL iceberg; LOAD iceberg;
SELECT * FROM iceberg_scan('s3://lake/warehouse/db/ventas');
FROM iceberg_snapshots('s3://lake/warehouse/db/ventas');
SELECT * FROM iceberg_scan('...', snapshot_from_id := 123456789);   -- time travel

-- Iceberg vía catálogo REST (AWS Glue, Polaris, S3 Tables, Lakekeeper...)
CREATE SECRET iceberg_cat (TYPE iceberg, CLIENT_ID '...', CLIENT_SECRET '...',
                           OAUTH2_SERVER_URI 'https://.../v1/oauth/tokens');
ATTACH 'warehouse' AS ice (TYPE iceberg, ENDPOINT 'https://catalogo.empresa.com', SECRET iceberg_cat);
SELECT * FROM ice.ventas.pedidos;
```

Así DuckDB sirve como **motor de consulta ligero para un lakehouse corporativo**: lees las mismas tablas que usa Spark o Databricks, pero desde tu laptop o una tarea pequeña, sin levantar un cluster. Ver [[08-Plataformas-de-Datos-y-ML]] para Databricks y Snowflake.

---

## `spatial`: datos geoespaciales

```sql
INSTALL spatial; LOAD spatial;

-- Leer cualquier formato GDAL: Shapefile, GeoJSON, GeoPackage, KML...
CREATE TABLE distritos AS FROM ST_Read('distritos_lima.geojson');

SELECT ST_Point(-77.0428, -12.0464)                          AS plaza_mayor,
       ST_Distance_Sphere(ST_Point(-77.04, -12.04), ST_Point(-77.03, -12.12)) AS metros,
       ST_Area(geom), ST_Centroid(geom), ST_AsText(geom), ST_AsGeoJSON(geom)
FROM distritos;

-- Spatial join: ¿en qué distrito cae cada tienda?
SELECT t.tienda_id, d.nombre AS distrito
FROM tiendas t
JOIN distritos d ON ST_Contains(d.geom, ST_Point(t.lon, t.lat));

-- Exportar
COPY (FROM distritos) TO 'distritos.gpkg' WITH (FORMAT gdal, DRIVER 'GPKG');
```

Alternativa ligera a PostGIS o GeoPandas para análisis espacial, por ejemplo cobertura de tiendas o geocercas en People Analytics. Para mapas, ver `Tableau/` y Power BI.

---

## `fts`: búsqueda de texto completo

```sql
INSTALL fts; LOAD fts;

CREATE TABLE docs (id INTEGER, titulo VARCHAR, cuerpo VARCHAR);
-- Crea el índice (esquema fts_main_docs) con stemming en español
PRAGMA create_fts_index('docs', 'id', 'titulo', 'cuerpo',
                        stemmer = 'spanish', stopwords = 'none', ignore = '(\.|[^a-z])+',
                        strip_accents = 1, lower = 1, overwrite = 1);

SELECT id, titulo, score
FROM (
    SELECT *, fts_main_docs.match_bm25(id, 'contrato laboral indefinido') AS score
    FROM docs
)
WHERE score IS NOT NULL
ORDER BY score DESC
LIMIT 10;
```

> El índice FTS **no se actualiza solo**: si cambias la tabla, vuelve a crearlo con `overwrite = 1`.

## Búsqueda vectorial con vss

```sql
INSTALL vss; LOAD vss;

CREATE TABLE chunks (id INT, texto VARCHAR, emb FLOAT[768]);
-- ... insertar embeddings calculados en Python (sentence-transformers, OpenAI, etc.)

CREATE INDEX idx_emb ON chunks USING HNSW (emb) WITH (metric = 'cosine');

SELECT id, texto
FROM chunks
ORDER BY array_cosine_distance(emb, $query_emb::FLOAT[768])
LIMIT 5;     -- el optimizador usa el índice HNSW en ORDER BY ... LIMIT
```

> Para persistir índices HNSW en un archivo `.duckdb` hay que activar `SET hnsw_enable_experimental_persistence = true`. Todavía es experimental, así que en producción considera recrear el índice al cargar.

**Búsqueda híbrida** (combina relevancia léxica y semántica, típico en RAG):

```sql
WITH lexico AS (
    SELECT id, fts_main_chunks.match_bm25(id, $q) AS bm25 FROM chunks
),
semantico AS (
    SELECT id, 1 - array_cosine_distance(emb, $q_emb::FLOAT[768]) AS sim FROM chunks
)
SELECT c.id, c.texto, 0.3 * coalesce(l.bm25, 0) + 0.7 * s.sim AS score
FROM chunks c JOIN semantico s USING (id) LEFT JOIN lexico l USING (id)
ORDER BY score DESC LIMIT 5;
```

Ver [[10-IA-Generativa-LLMs-y-Agentes]] para el contexto de RAG.

## Otras extensiones útiles

```sql
-- Datasets de benchmark al instante
INSTALL tpch; LOAD tpch;
CALL dbgen(sf = 1);         -- ~1 GB de tablas TPC-H para practicar
PRAGMA tpch(1);             -- ejecuta la consulta 1

-- Excel
SELECT * FROM read_xlsx('reporte.xlsx', sheet = 'Datos');

-- Google Sheets (community)
INSTALL gsheets FROM community; LOAD gsheets;
```

## Ver también

- [[06 - Fuentes Remotas y Cloud]]
- [[14 - DuckLake, MotherDuck y el Lakehouse Ligero]]
- [[08 - Tipos Anidados#ARRAY: longitud fija y vectores]]
- [[04-Ingenieria-de-Datos]] y [[08-Plataformas-de-Datos-y-ML]] (en `Machine Learning/`)
- [[24 - Databases]] (en `Python/`)
