---
tags: [duckdb, cli, instalacion, configuracion, cheat-sheet]
---

# 02 — Instalación, CLI y Bases de Datos

> Continúa de [[01 - Introducción y Arquitectura]]. Aquí está todo lo necesario para tener DuckDB funcionando: instalarlo, usar su CLI, crear o abrir bases y configurar el motor.

## Instalación

DuckDB no tiene dependencias externas: cada cliente es un binario o paquete autocontenido.

```bash
# Python (el caso más común)
pip install duckdb
# o con uv:  uv add duckdb

# CLI en Windows
winget install DuckDB.cli

# CLI en macOS
brew install duckdb

# CLI en Linux / macOS (script oficial)
curl https://install.duckdb.org | sh

# R
install.packages("duckdb")

# Node.js (cliente "neo", el recomendado actualmente)
npm install @duckdb/node-api

# Java: dependencia Maven org.duckdb:duckdb_jdbc
```

Verificar:

```bash
duckdb --version
python -c "import duckdb; print(duckdb.__version__)"
```

> **Compatibilidad entre versiones:** la CLI y el paquete de Python son independientes. Si creas un `.duckdb` con Python 1.4 y lo abres con una CLI 1.1, puede fallar. Mantén ambas en la misma versión (ver [[01 - Introducción y Arquitectura#Versiones y ciclo de releases]]).

---

## La CLI de DuckDB

### Arrancar

```bash
duckdb                       # base en memoria (se pierde al salir)
duckdb analitica.duckdb      # abre o crea una base persistente
duckdb -readonly prod.duckdb # solo lectura (permite varios procesos lectores)
duckdb -ui                   # abre la UI web local (notebooks SQL en el navegador)

# Ejecutar sin entrar al modo interactivo
duckdb -c "SELECT COUNT(*) FROM 'ventas.parquet'"
duckdb analitica.duckdb < script.sql
duckdb -csv -c "FROM 'datos.parquet' LIMIT 10" > muestra.csv
duckdb -json -c "SUMMARIZE 'datos.parquet'"
```

La CLI también sirve como **navaja suiza para archivos**: convertir CSV a Parquet, inspeccionar un JSON o contar filas sin abrir Python.

```bash
duckdb -c "COPY (FROM 'enorme.csv') TO 'enorme.parquet' (FORMAT parquet)"
```

### Dot-commands más útiles

Los comandos que empiezan con `.` son de la CLI, no SQL. No llevan `;` al final.

| Comando | Qué hace |
|---|---|
| `.help` | Lista todos los comandos |
| `.open archivo.duckdb` | Cierra la base actual y abre otra |
| `.tables` | Lista tablas y vistas |
| `.schema [tabla]` | Muestra el `CREATE` de las tablas |
| `.mode duckbox` | Formato por defecto (tabla con tipos) |
| `.mode csv` / `json` / `markdown` / `line` / `box` / `table` | Cambia el formato de salida |
| `.maxrows 50` / `.maxwidth 200` | Límites de impresión en modo duckbox |
| `.timer on` | Muestra el tiempo de cada consulta |
| `.read script.sql` | Ejecuta un archivo SQL |
| `.output salida.csv` | Redirige la salida a un archivo (`.output` solo vuelve a stdout) |
| `.once salida.md` | Redirige solo la próxima consulta |
| `.headers off` | Oculta encabezados (útil con `.mode csv`) |
| `.nullvalue NULL` | Cómo mostrar los nulos |
| `.shell cmd` / `.system cmd` | Ejecuta un comando del sistema |
| `.exit` o `Ctrl+D` | Salir |

Ejemplo de sesión típica:

```sql
.timer on
.mode markdown
SELECT region, COUNT(*) AS n FROM 'ventas.parquet' GROUP BY ALL;
.once top10.csv
.mode csv
FROM 'ventas.parquet' ORDER BY monto DESC LIMIT 10;
```

### Archivo de inicio `~/.duckdbrc`

La CLI ejecuta este archivo al arrancar. Sirve para dejar tus preferencias fijas:

```sql
-- ~/.duckdbrc
.timer on
.maxrows 40
SET threads = 8;
```

### La UI local

Desde la versión 1.2.1 existe una **UI web oficial**, servida localmente por la extensión `ui`:

```bash
duckdb -ui analitica.duckdb
```

```sql
-- o desde cualquier cliente (por ejemplo, Python):
CALL start_ui();
```

Ofrece notebooks SQL, un explorador de tablas con resúmenes por columna y autocompletado. Todo corre en local; solo se conecta a MotherDuck si tú inicias sesión.

---

## Bases de datos: memoria, archivo y `ATTACH`

### En memoria vs. persistente

| Modo | Cómo | Cuándo |
|---|---|---|
| **En memoria** | `duckdb` / `duckdb.connect()` / `':memory:'` | Exploración, tests, transformar archivos y escribir el resultado a Parquet |
| **Persistente** | `duckdb archivo.duckdb` / `duckdb.connect('archivo.duckdb')` | Guardar tablas entre sesiones, warehouse local, base de dbt |

El archivo `.duckdb` viene acompañado de un `.duckdb.wal` (write-ahead log) mientras hay cambios sin hacer *checkpoint*. **No borres el `.wal`** con la base abierta.

> Una base en memoria **sí puede hacer spilling a disco** si la consulta no cabe en RAM: usa un directorio temporal (ver [[15 - Rendimiento y Optimización]]).

### Varias bases a la vez con `ATTACH`

DuckDB puede tener **varias bases adjuntas** en la misma sesión y hacer joins entre ellas, incluidas bases de otros motores (ver [[12 - Extensiones]]).

```sql
ATTACH 'ventas.duckdb'       AS ventas;
ATTACH 'catalogo.duckdb'     AS cat (READ_ONLY);
ATTACH ':memory:'            AS scratch;
ATTACH 'legacy.sqlite'       AS lite (TYPE sqlite);          -- extensión sqlite
ATTACH 'dbname=crm host=localhost user=app' AS pg (TYPE postgres, READ_ONLY);

SHOW DATABASES;               -- lista bases adjuntas
USE ventas;                   -- base/esquema por defecto
SELECT v.*, p.nombre
FROM ventas.main.pedidos v
JOIN cat.productos p USING (producto_id);

-- Copiar una base entera a otra
COPY FROM DATABASE ventas TO scratch;

DETACH cat;
```

Nombres completamente calificados: `base.esquema.tabla`. El esquema por defecto es `main`.

```sql
CREATE SCHEMA IF NOT EXISTS staging;
CREATE TABLE staging.clientes AS FROM 'clientes.csv';
SELECT current_database(), current_schema();
```

### Exportar e importar una base completa

Para respaldos, migraciones entre versiones o para compartir datos en formato abierto:

```sql
EXPORT DATABASE 'respaldo/' (FORMAT parquet, COMPRESSION zstd);   -- schema.sql + load.sql + datos
IMPORT DATABASE 'respaldo/';
```

Esta es la vía recomendada para **migrar una base a una versión incompatible** de DuckDB: exportas con la vieja e importas con la nueva.

---

## Configuración del motor

### `SET`, `RESET` y `current_setting`

```sql
SET threads = 4;                        -- por defecto: todos los núcleos
SET memory_limit = '8GB';               -- por defecto: 80% de la RAM
SET temp_directory = '/tmp/duckdb_spill';
SET preserve_insertion_order = false;   -- menos memoria en cargas grandes
SET enable_progress_bar = true;
SET TimeZone = 'America/Lima';          -- requiere ICU (autocargado)

SELECT current_setting('memory_limit');
RESET threads;

-- Todas las opciones con su valor y descripción
SELECT name, value, description FROM duckdb_settings() WHERE name LIKE '%memory%';
```

`SET` aplica a la conexión o a la base según la opción. `SET GLOBAL` y `SET SESSION` permiten ser explícito.

### Configuración al conectar (Python)

```python
con = duckdb.connect(
    "analitica.duckdb",
    read_only=False,
    config={"threads": 4, "memory_limit": "6GB", "access_mode": "READ_WRITE"},
)
```

### `PRAGMA`: comandos de metadata

`PRAGMA` es la sintaxis heredada de SQLite. Muchas opciones existen también como función de tabla:

```sql
PRAGMA database_size;          -- tamaño, bloques usados, WAL
PRAGMA table_info('ventas');   -- columnas y tipos
PRAGMA show_tables;
PRAGMA storage_info('ventas'); -- compresión por columna y row group
PRAGMA version;
```

### Catálogo del sistema

Las funciones `duckdb_*()` exponen la metadata como tablas consultables:

```sql
SELECT * FROM duckdb_tables();
SELECT * FROM duckdb_columns() WHERE table_name = 'ventas';
SELECT * FROM duckdb_views();
SELECT * FROM duckdb_functions() WHERE function_name ILIKE '%list%';
SELECT * FROM duckdb_extensions();
SELECT * FROM duckdb_memory();     -- uso de memoria por componente
-- También existe information_schema estándar:
SELECT table_name FROM information_schema.tables;
```

Atajos de exploración:

```sql
SHOW TABLES;
SHOW ALL TABLES;          -- de todas las bases adjuntas
DESCRIBE ventas;          -- columnas y tipos
DESCRIBE SELECT * FROM 'archivo.parquet';
SUMMARIZE ventas;         -- min, max, nulos, distintos aproximados, cuartiles por columna
```

`SUMMARIZE` es el equivalente SQL de `df.describe()` en Pandas, y sirve para el primer contacto con un dataset.

---

## Checklist de instalación para un proyecto

1. Fija la versión en `requirements.txt` / `pyproject.toml` (`duckdb==1.x.y`). Ver [[20 - Package Managers]] y [[22 - Virtual Environments]].
2. Agrega `*.duckdb` y `*.duckdb.wal` al `.gitignore`. Las bases son artefactos, no código; si necesitas versionarlas, usa DVC.
3. Define `memory_limit` y `temp_directory` explícitos en contenedores, donde el 80% de RAM del host puede no coincidir con el límite del contenedor (ver [[15 - Rendimiento y Optimización]]).
4. Usa `-readonly` / `read_only=True` para procesos que solo consultan.

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[03 - API de Python]]
- [[10 - DDL, Transacciones y Concurrencia]]: concurrencia y `READ_ONLY`
- [[12 - Extensiones]]: `ATTACH` a Postgres, SQLite y MySQL
- [[02 - Instalación, Proyecto y profiles.yml|dbt/02]]: configurar DuckDB como target de dbt
- [[Python/DVC/01 - Introducción y Arquitectura Interna|DVC]]: versionar archivos `.duckdb` o Parquet grandes
