---
tags: [duckdb, csv, json, ingesta, copy, cheat-sheet]
---

# 04 — Leer y Escribir CSV y JSON

> Continúa de [[03 - API de Python]]. Una de las razones principales para usar DuckDB es **consultar archivos sin importarlos**. Esta nota cubre los formatos de texto: CSV (con su detector automático) y JSON/NDJSON, además de la escritura con `COPY`. Parquet, el formato preferido para analítica, está en [[05 - Parquet, Globs y Particionado Hive]].

## Tres formas de leer un archivo

```sql
-- 1. Nombre del archivo como tabla (DuckDB infiere el formato por la extensión)
SELECT * FROM 'clientes.csv';
FROM 'eventos.json';               -- FROM-first, ver nota 07

-- 2. Función explícita con opciones
SELECT * FROM read_csv('clientes.csv', delim = ';', header = true);

-- 3. Materializar a tabla
CREATE TABLE clientes AS FROM read_csv('clientes.csv');
-- o insertar en una tabla ya definida
COPY clientes FROM 'clientes.csv' (HEADER, DELIMITER ',');
```

¿Leer directo o materializar? Si vas a consultar el archivo **muchas veces**, conviértelo en tabla o en Parquet, porque parsear CSV en cada consulta es caro. Para una exploración puntual, léelo directo.

---

## CSV

### El sniffer: detección automática

`read_csv` (alias `read_csv_auto`) analiza una muestra del archivo y detecta delimitador, comillas, escape, encabezado, tipos de cada columna y formatos de fecha.

```sql
-- Ver qué detectó el sniffer, sin leer el archivo completo
FROM sniff_csv('ventas.csv');
-- devuelve Delimiter, Quote, Escape, HasHeader, Columns, DateFormat, ... y un "Prompt"
-- con la llamada read_csv(...) exacta que puedes copiar y fijar en producción

DESCRIBE FROM 'ventas.csv';     -- solo el esquema inferido
```

> **Producción:** en pipelines conviene **fijar** el esquema y las opciones (copiando el `Prompt` de `sniff_csv`) en lugar de confiar en la auto-detección. Si un proveedor cambia el formato, prefieres que el pipeline falle de forma explícita y no que cambie un tipo en silencio. Complementa esto con validación de esquema (ver [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]]).

### Opciones más usadas

```sql
SELECT * FROM read_csv(
    'datos/ventas_*.csv',
    header          = true,
    delim           = ';',               -- alias: sep
    quote           = '"',
    escape          = '"',
    decimal_separator = ',',             -- CSVs latinoamericanos o europeos
    dateformat      = '%d/%m/%Y',
    timestampformat = '%d/%m/%Y %H:%M:%S',
    nullstr         = ['', 'NA', 'N/A', 'null'],
    skip            = 2,                 -- saltar líneas iniciales (títulos)
    encoding        = 'latin-1',         -- utf-8 por defecto; utf-16 y latin-1 soportados
    compression     = 'gzip',            -- auto-detectado por la extensión .gz/.zst
    filename        = true,              -- agrega columna con el archivo de origen
    union_by_name   = true,              -- varios archivos con columnas en distinto orden
    normalize_names = true,              -- "Fecha Venta" -> fecha_venta
    sample_size     = -1                 -- usar TODO el archivo para inferir tipos
);
```

| Opción | Para qué |
|---|---|
| `columns = {'id': 'INTEGER', 'nombre': 'VARCHAR'}` | Esquema completo explícito (desactiva la inferencia de columnas) |
| `types = {'codigo_postal': 'VARCHAR'}` | Forzar el tipo solo de algunas columnas. Ejemplo típico: códigos con ceros a la izquierda |
| `all_varchar = true` | Leer todo como texto (útil en la capa "raw/bronze") |
| `auto_type_candidates = ['BIGINT','DOUBLE','VARCHAR']` | Restringir los tipos que puede inferir |
| `null_padding = true` | Rellenar con NULL las filas con menos columnas |
| `max_line_size` | Líneas muy largas |
| `parallel = false` | Desactivar la lectura paralela (raro; solo con archivos problemáticos) |

### Archivos sucios: `ignore_errors` y tablas de rechazos

```sql
-- Opción 1: saltar filas inválidas en silencio (peligroso: pierdes datos sin saberlo)
SELECT * FROM read_csv('sucio.csv', ignore_errors = true);

-- Opción 2: saltarlas, pero guardarlas para auditar (RECOMENDADO)
SELECT * FROM read_csv('sucio.csv', store_rejects = true);

FROM reject_errors;   -- línea, columna, tipo de error, mensaje, contenido original
FROM reject_scans;    -- configuración del escaneo que generó los errores

-- Con nombres propios y un límite de errores
SELECT * FROM read_csv('sucio.csv',
    store_rejects = true,
    rejects_table = 'rechazos_ventas',
    rejects_scan  = 'scans_ventas',
    rejects_limit = 1000);
```

Patrón de calidad de datos en un pipeline:

```python
con.execute("CREATE TABLE ventas AS FROM read_csv('in/*.csv', store_rejects = true)")
n_err = con.sql("SELECT COUNT(*) FROM reject_errors").fetchone()[0]
if n_err > 0:
    con.sql("COPY reject_errors TO 'rechazos.csv' (HEADER)")
    if n_err > 100:
        raise ValueError(f"{n_err} filas rechazadas: revisar rechazos.csv")
```

### Varios archivos a la vez

```sql
FROM read_csv('logs/2026-*.csv');                         -- glob
FROM read_csv(['enero.csv', 'febrero.csv']);              -- lista
FROM read_csv('logs/**/*.csv', filename = true);          -- recursivo y con origen
FROM read_csv('logs/*.csv', union_by_name = true);        -- esquemas que evolucionan
```

Los globs y el particionado se explican a fondo en [[05 - Parquet, Globs y Particionado Hive]].

---

## JSON

La extensión `json` viene incluida y se autocarga.

### Formatos de JSON soportados

| Formato | Aspecto | Opción |
|---|---|---|
| **NDJSON / JSON Lines** | Un objeto por línea | `format = 'newline_delimited'` |
| **Array** | `[ {...}, {...} ]` | `format = 'array'` |
| **Unstructured** | Objetos concatenados, multilínea | `format = 'unstructured'` |
| Auto | DuckDB lo detecta | `format = 'auto'` (por defecto) |

```sql
SELECT * FROM 'eventos.ndjson';
SELECT * FROM read_json('api_dump.json', format = 'array');
SELECT * FROM read_ndjson('logs/*.jsonl.gz');

-- Esquema explícito
SELECT * FROM read_json('eventos.json',
    columns = {ts: 'TIMESTAMP', usuario: 'VARCHAR', payload: 'JSON'});

-- Inferencia más profunda o más superficial
SELECT * FROM read_json('anidado.json', maximum_depth = 2, sample_size = 50000);
```

### Del JSON anidado a columnas

DuckDB convierte objetos JSON en **STRUCT** y arrays en **LIST**, así que puedes navegar la estructura con notación de punto:

```sql
-- eventos.json:  {"id": 1, "usuario": {"nombre": "Ana", "pais": "PE"}, "items": [{"sku":"A","qty":2}]}
SELECT id,
       usuario.nombre,
       usuario.pais,
       unnest(items, recursive := true)     -- una fila por item, con columnas sku y qty
FROM 'eventos.json';
```

Para manipular el JSON como texto o cuando el esquema es muy irregular:

```sql
SELECT payload->'$.cliente.id'        AS cliente_json,   -- devuelve JSON
       payload->>'$.cliente.nombre'   AS cliente_txt,    -- devuelve VARCHAR
       json_extract_string(payload, '$.items[0].sku') AS primer_sku,
       json_array_length(payload, '$.items')          AS n_items,
       json_keys(payload)                              AS claves
FROM read_json_objects('raw.json');    -- cada objeto como una columna JSON sin parsear
```

El tipo JSON y sus funciones se detallan en [[08 - Tipos Anidados#JSON como tipo]].

---

## Escribir archivos con `COPY ... TO`

```sql
-- Tabla completa a CSV
COPY clientes TO 'clientes.csv' (HEADER, DELIMITER ',');

-- Resultado de una consulta
COPY (
    SELECT region, SUM(monto) AS total FROM ventas GROUP BY ALL
) TO 'resumen.csv' (HEADER, DELIMITER ';', QUOTE '"', DATEFORMAT '%d/%m/%Y');

-- Comprimido
COPY ventas TO 'ventas.csv.gz' (HEADER, COMPRESSION gzip);

-- JSON (NDJSON por defecto; ARRAY true para un array)
COPY ventas TO 'ventas.json';
COPY ventas TO 'ventas_array.json' (ARRAY true);

-- Excel (extensión excel)
INSTALL excel; LOAD excel;
COPY resumen TO 'reporte.xlsx' (FORMAT xlsx, HEADER true);
```

| Opción de `COPY TO` | Descripción |
|---|---|
| `FORMAT csv / parquet / json / xlsx` | Formato (inferido por la extensión si se omite) |
| `HEADER` | Escribir encabezado (CSV) |
| `DELIMITER`, `QUOTE`, `ESCAPE` | Formato CSV |
| `COMPRESSION gzip / zstd / snappy` | Compresión |
| `PARTITION_BY (col1, col2)` | Escribir carpetas particionadas estilo Hive (ver nota 05) |
| `OVERWRITE_OR_IGNORE` / `OVERWRITE` | Qué hacer si el destino existe |
| `PER_THREAD_OUTPUT` | Un archivo por thread (escritura más rápida) |
| `FILE_SIZE_BYTES '100MB'` | Partir la salida en varios archivos |
| `USE_TMP_FILE` | Escribir a un temporal y renombrar (escritura atómica) |

### CSV → Parquet: el caso más común

```sql
COPY (FROM read_csv('raw/*.csv', union_by_name = true, filename = true))
TO 'bronze/ventas.parquet' (FORMAT parquet, COMPRESSION zstd);
```

El Parquet resultante pesa típicamente entre 5 y 10 veces menos que el CSV y se consulta muchísimo más rápido.

## Excel como entrada

```sql
INSTALL excel; LOAD excel;
SELECT * FROM read_xlsx('presupuesto.xlsx', sheet = 'Q1', header = true, range = 'A1:F200');
```

Para Excel con formato complejo (celdas combinadas, varias tablas por hoja) es preferible Pandas con `openpyxl`, y luego consultar el DataFrame desde DuckDB (ver [[11 - Integración con Pandas, Polars y Arrow]]).

## Ver también

- [[05 - Parquet, Globs y Particionado Hive]]
- [[06 - Fuentes Remotas y Cloud]]: los mismos lectores sobre S3 y HTTP
- [[08 - Tipos Anidados]]: trabajar con el JSON una vez cargado
- [[14 - JSON and YAML]] (en `Python/`): JSON desde Python estándar
- [[13 - Files Handling]] (en `Python/`)
- [[05 - Sources y Seeds|dbt/05]]: los seeds de dbt cargan CSVs pequeños sobre DuckDB
