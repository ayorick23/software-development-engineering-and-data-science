---
tags: [duckdb, sql, friendly-sql, dialecto, cheat-sheet]
---

# 07 — Friendly SQL

> Continúa de [[06 - Fuentes Remotas y Cloud]]. El dialecto de DuckDB parte de PostgreSQL y le añade una larga lista de **extensiones ergonómicas** ("Friendly SQL") que eliminan el boilerplate del SQL estándar. Esta nota asume que ya dominas el SQL de la carpeta `SQL/` ([[DQL (Data Query Language)]], [[Joins]], [[SQL/Aggregate Functions|Aggregate Functions]], [[Window Functions]]) y muestra lo que DuckDB hace **distinto o mejor**.

> ⚠️ **Portabilidad:** casi todo lo de esta nota es específico de DuckDB (algunas cosas existen también en Snowflake, BigQuery o Databricks). Si tu SQL debe correr también en Postgres o SQL Server, úsalo con cuidado. En dbt sobre DuckDB puedes usarlo libremente.

## `FROM` primero y `SELECT` implícito

```sql
FROM ventas;                              -- = SELECT * FROM ventas
FROM ventas WHERE monto > 100 LIMIT 5;
FROM ventas SELECT region, monto;         -- el FROM puede ir antes que el SELECT
FROM 'archivo.parquet';                   -- ideal para explorar
```

## Selección de columnas: `EXCLUDE`, `REPLACE`, `RENAME`, `COLUMNS()`

### `* EXCLUDE` y `* REPLACE`

```sql
-- Todas las columnas menos algunas (adiós a listar 40 columnas a mano)
SELECT * EXCLUDE (password_hash, email) FROM usuarios;

-- Todas, pero transformando algunas en su lugar (mantiene nombre y posición)
SELECT * REPLACE (round(monto, 2) AS monto, upper(region) AS region) FROM ventas;

-- Todas, renombrando algunas
SELECT * RENAME (cli_id AS cliente_id) FROM pedidos;

-- Combinables, y con alias de tabla en joins
SELECT p.* EXCLUDE (created_at), c.nombre
FROM pedidos p JOIN clientes c ON p.cliente_id = c.id;
```

### `COLUMNS()`: aplicar una expresión a muchas columnas

`COLUMNS()` selecciona columnas por **regex**, **lista** o **lambda**, y la expresión que la envuelve se aplica a cada una:

```sql
-- Todas las columnas que empiezan con "monto_"
SELECT COLUMNS('monto_.*') FROM ventas;

-- Promedio de todas las columnas numéricas que matchean
SELECT avg(COLUMNS('^(precio|costo|margen)$')) FROM productos;

-- Lambda sobre el nombre de columna
SELECT COLUMNS(c -> c LIKE '%_id') FROM pedidos;

-- Excluir dentro de COLUMNS
SELECT max(COLUMNS(* EXCLUDE (id, fecha))) FROM metricas;

-- Renombrar con grupos de captura de la regex
SELECT COLUMNS('(\w+)_usd') AS '\1_dolares' FROM ventas;

-- Filtrar filas donde NINGUNA columna sea nula
SELECT * FROM t WHERE COLUMNS(*) IS NOT NULL;   -- se expande con AND

-- Conteo de nulos por columna (EDA en una línea)
SELECT count(*) - count(COLUMNS(*)) FROM ventas;

-- *COLUMNS(...) "desempaqueta" dentro de una función variádica
SELECT coalesce(*COLUMNS(['tel_movil', 'tel_casa', 'tel_trabajo'])) AS telefono FROM clientes;
```

`SELECT avg(COLUMNS(*))` sobre una tabla numérica equivale a `df.mean()` de Pandas.

## `GROUP BY ALL` y `ORDER BY ALL`

```sql
-- Agrupa automáticamente por todas las columnas que NO son agregados
SELECT region, date_trunc('month', fecha) AS mes, SUM(monto), COUNT(*)
FROM ventas
GROUP BY ALL;          -- = GROUP BY region, date_trunc('month', fecha)

ORDER BY ALL;          -- ordena por todas las columnas, de izquierda a derecha
ORDER BY ALL DESC;
```

Esto elimina el error clásico de agregar una columna al `SELECT` y olvidarla en el `GROUP BY`.

## Alias reutilizables en la misma consulta

```sql
SELECT monto * 0.18        AS igv,
       monto + igv         AS total          -- usa el alias definido arriba
FROM ventas
WHERE total > 1000                           -- alias en WHERE
GROUP BY ALL
HAVING total > 5000;

-- Alias con sintaxis de prefijo (1.2+)
SELECT total: monto + igv, cliente: upper(nombre) FROM ...;
```

## `QUALIFY`: filtrar por window functions

`QUALIFY` es a las window functions lo que `HAVING` es a los agregados: evita envolver la consulta en una subconsulta.

```sql
-- Última compra por cliente (dedupe clásico)
SELECT *
FROM pedidos
QUALIFY row_number() OVER (PARTITION BY cliente_id ORDER BY fecha DESC) = 1;

-- Top 3 productos por región
SELECT region, producto, SUM(monto) AS total
FROM ventas
GROUP BY ALL
QUALIFY rank() OVER (PARTITION BY region ORDER BY total DESC) <= 3;
```

Comparar con la versión estándar con CTE en [[Common Table Expressions]] y [[Window Functions]].

## `DISTINCT ON`

```sql
-- Una fila por cliente: la de fecha más reciente (sintaxis PostgreSQL)
SELECT DISTINCT ON (cliente_id) *
FROM pedidos
ORDER BY cliente_id, fecha DESC;
```

## Agregados prácticos

```sql
SELECT
    count_if(monto > 100)                    AS ventas_grandes,
    sum(monto) FILTER (WHERE region = 'LATAM') AS total_latam,   -- FILTER: agregado condicional
    arg_max(producto, monto)                 AS producto_mayor_venta,  -- alias max_by
    arg_min(fecha, monto)                    AS fecha_menor_venta,
    max(monto, 3)                            AS top3_montos,     -- lista con los 3 mayores
    list(DISTINCT region ORDER BY region)    AS regiones,
    string_agg(producto, ', ')               AS productos,
    median(monto), mode(region),
    quantile_cont(monto, [0.25, 0.5, 0.75])  AS cuartiles,
    approx_count_distinct(cliente_id)        AS clientes_aprox,
    bool_and(pagado)                         AS todo_pagado,
    histogram(region)                        AS conteo_por_region   -- MAP valor -> conteo
FROM ventas;
```

## Uniones por nombre

```sql
-- UNION por nombre de columna, no por posición; columnas faltantes se rellenan con NULL
SELECT * FROM ventas_2025
UNION ALL BY NAME
SELECT * FROM ventas_2026;

-- INSERT por nombre (el orden de columnas deja de importar)
INSERT INTO ventas BY NAME SELECT monto, fecha, region FROM staging;
```

## Joins extra

```sql
-- SEMI / ANTI JOIN explícitos (más claros que EXISTS / NOT EXISTS)
SELECT * FROM clientes SEMI JOIN pedidos USING (cliente_id);   -- clientes con pedidos
SELECT * FROM clientes ANTI JOIN pedidos USING (cliente_id);   -- clientes sin pedidos

-- POSITIONAL JOIN: une fila 1 con fila 1, fila 2 con fila 2 (como pd.concat(axis=1))
SELECT * FROM df_a POSITIONAL JOIN df_b;

-- LATERAL: subconsulta correlacionada por fila
SELECT c.nombre, ult.*
FROM clientes c,
LATERAL (SELECT * FROM pedidos p WHERE p.cliente_id = c.id ORDER BY fecha DESC LIMIT 2) ult;

-- ASOF JOIN: ver nota 09
```

Los joins estándar están en [[Joins]].

## Strings y encadenamiento de funciones

```sql
SELECT 'DuckDB'[1:4];                       -- 'Duck'  (slicing, índice desde 1)
SELECT 'DuckDB'[-2:];                       -- 'DB'
SELECT nombre.lower().trim() FROM clientes; -- sintaxis de punto (method chaining)
SELECT 'a' || 'b', concat_ws('-', 'a', 'b', 'c');
SELECT email SIMILAR TO '.*@gmail\.com' FROM clientes;
SELECT regexp_extract(sku, '([A-Z]+)-(\d+)', 2) FROM productos;
SELECT regexp_replace(tel, '[^0-9]', '', 'g') FROM clientes;
SELECT format('{} compró {:.2f}', nombre, monto) FROM ...;   -- estilo fmt/Python
SELECT printf('%s: %d', region, n) FROM ...;
SELECT levenshtein('duck', 'luck'), jaro_winkler_similarity('Pérez', 'Perez');
SELECT strip_accents('Méndez');             -- 'Mendez'
```

Las funciones de texto estándar están en [[Advanced Functions]].

## Más atajos

```sql
-- Comas al final permitidas
SELECT a, b, c, FROM t;

-- LIMIT porcentual
FROM ventas LIMIT 10%;

-- Muestreo
FROM ventas USING SAMPLE 1000;            -- 1000 filas
FROM ventas USING SAMPLE 5%;              -- 5% (reservoir/system)
FROM ventas USING SAMPLE 10% (bernoulli, 42);   -- método y seed

-- IS [NOT] DISTINCT FROM: comparación que trata NULL = NULL como verdadero
SELECT * FROM a JOIN b ON a.x IS NOT DISTINCT FROM b.x;

-- Literales e intervalos
SELECT DATE '2026-11-15' + INTERVAL 3 DAY, now() - INTERVAL '2 hours';

-- Generadores de filas
FROM range(5);                            -- 0..4
FROM generate_series(1, 10, 2);           -- 1,3,5,7,9 (incluye el fin)
SELECT * FROM range(DATE '2026-01-01', DATE '2026-02-01', INTERVAL 1 DAY);

-- CREATE OR REPLACE e IF NOT EXISTS
CREATE OR REPLACE TABLE t AS FROM 'x.parquet';
CREATE TABLE IF NOT EXISTS t (id INT);

-- Variables de sesión
SET VARIABLE corte = DATE '2026-06-30';
SELECT * FROM ventas WHERE fecha <= getvariable('corte');

-- Prefijo con nombre del archivo como tabla en joins
SELECT * FROM 'clientes.csv' c JOIN 'pedidos.parquet' p ON c.id = p.cliente_id;
```

## Tabla resumen

| Característica | SQL estándar | DuckDB |
|---|---|---|
| Todas menos X | Listar todas a mano | `SELECT * EXCLUDE (x)` |
| Transformar algunas | Listar todas a mano | `SELECT * REPLACE (f(x) AS x)` |
| Misma función a N columnas | N expresiones | `f(COLUMNS('regex'))` |
| Agrupar por lo no agregado | Repetir columnas | `GROUP BY ALL` |
| Filtrar por ventana | Subconsulta o CTE | `QUALIFY` |
| Reusar un alias | Repetir la expresión | Alias en `WHERE`/`GROUP BY`/`HAVING` |
| Union de esquemas distintos | Alinear columnas a mano | `UNION ALL BY NAME` |
| Explorar | `SELECT * FROM` | `FROM tabla` / `SUMMARIZE` |

## Ver también

- [[DQL (Data Query Language)]], [[SQL/Aggregate Functions|Aggregate Functions]], [[Window Functions]], [[Common Table Expressions]] (en `SQL/`)
- [[08 - Tipos Anidados]]
- [[09 - Analítica Avanzada]]
- [[26-SQL-Profesional-Nivel-1-Consultas-Avanzadas]] y [[27-SQL-Profesional-Nivel-2-Optimizacion]] (en `Machine Learning/`)
- [[04 - Models, ref() y el DAG|dbt/04]]: Friendly SQL dentro de modelos dbt sobre DuckDB
