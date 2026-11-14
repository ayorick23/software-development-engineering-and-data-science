---
tags: [dbt, packages, dbt-utils, dbt-expectations, codegen, cheat-sheet]
---

# 10 — Packages

Un **package** es un proyecto dbt (macros, modelos, tests) que se instala dentro de otro. Es el equivalente a `pip install` para dbt. El catálogo público está en **hub.getdbt.com**.

## Instalación

```yaml
# packages.yml (raíz del proyecto)
packages:
  - package: dbt-labs/dbt_utils
    version: [">=1.3.0", "<2.0.0"]

  - package: metaplane/dbt_expectations      # el fork mantenido; calogica/dbt_expectations está archivado
    version: [">=0.10.0", "<0.11.0"]

  - package: dbt-labs/codegen
    version: [">=0.13.0", "<1.0.0"]

  - package: dbt-labs/audit_helper
    version: [">=0.12.0", "<1.0.0"]

  # Desde Git (paquete interno o versión no publicada)
  - git: "https://github.com/mi-org/dbt-shared-macros.git"
    revision: v1.4.0          # tag, rama o commit SHA
    subdirectory: "dbt"       # si el proyecto no está en la raíz del repo

  # Desde una ruta local (monorepo)
  - local: ../shared_dbt_package
```

```bash
dbt deps          # instala en dbt_packages/ y genera package-lock.yml
```

> **Versiona `package-lock.yml`** en Git: fija las versiones exactas resueltas (igual que `uv.lock` o `poetry.lock`). Y agrega `dbt_packages/` al `.gitignore`.

Desde dbt 1.8 también se pueden declarar en `dependencies.yml`, que además admite dependencias entre proyectos (dbt Mesh). `packages.yml` sigue siendo lo más común para packages con Jinja.

## dbt_utils — la navaja suiza

Mantenido por dbt Labs, presente en prácticamente todos los proyectos.

### Macros SQL

```sql
-- Clave surrogada (hash de varias columnas, null-safe)
select
    {{ dbt_utils.generate_surrogate_key(['order_id', 'line_number']) }} as order_line_sk,
    *
from {{ ref('stg_order_lines') }}
```

```sql
-- Todas las columnas excepto algunas
select
    {{ dbt_utils.star(from=ref('stg_customers'), except=['_loaded_at', 'raw_payload']) }}
from {{ ref('stg_customers') }}
```

```sql
-- Calendario: una fila por día
{{ dbt_utils.date_spine(
    datepart="day",
    start_date="cast('2024-01-01' as date)",
    end_date="cast('2026-12-31' as date)"
) }}
```

```sql
-- Valores distintos de una columna (para pivots dinámicos)
{% set methods = dbt_utils.get_column_values(ref('stg_payments'), 'payment_method') %}

select
    order_id,
    {{ dbt_utils.pivot('payment_method', methods, agg='sum', then_value='amount', prefix='amt_') }}
from {{ ref('stg_payments') }}
group by 1
```

```sql
-- Unir varias tablas con columnas distintas (rellena con null)
{{ dbt_utils.union_relations(
    relations=[ref('orders_peru'), ref('orders_chile'), ref('orders_mexico')],
    source_column_name='_source_table'
) }}
```

```sql
-- Desduplicar
{{ dbt_utils.deduplicate(
    relation=ref('stg_events'),
    partition_by='event_id',
    order_by='_loaded_at desc'
) }}
```

Otras útiles: `group_by(n)` (genera `group by 1,2,...,n`), `safe_divide(a, b)`, `get_relations_by_pattern()`, `unpivot()`, `width_bucket()`, `haversine_distance()`.

### Generic tests de dbt_utils

```yaml
models:
  - name: fct_orders
    data_tests:
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns: [order_id, line_number]
      - dbt_utils.expression_is_true:
          expression: "total_amount >= 0"
      - dbt_utils.recency:
          datepart: day
          field: order_date
          interval: 1           # falla si no hay datos de ayer
      - dbt_utils.equal_rowcount:
          compare_model: ref('stg_orders')
    columns:
      - name: discount_pct
        data_tests:
          - dbt_utils.accepted_range:
              min_value: 0
              max_value: 1
      - name: email
        data_tests:
          - dbt_utils.not_empty_string
```

Más: `at_least_one`, `not_constant`, `cardinality_equality`, `mutually_exclusive_ranges` (ideal para validar SCD2: que los rangos `valid_from/valid_to` no se solapen), `sequential_values`, `relationships_where`.

## dbt_expectations — Great Expectations en YAML

Port de las "expectations" de Great Expectations a tests de dbt, sin infraestructura extra.

```yaml
models:
  - name: stg_customers
    data_tests:
      - dbt_expectations.expect_table_row_count_to_be_between:
          min_value: 1000
      - dbt_expectations.expect_table_columns_to_match_ordered_list:
          column_list: [customer_id, first_name, last_name, email, created_at]
    columns:
      - name: email
        data_tests:
          - dbt_expectations.expect_column_values_to_match_regex:
              regex: "^[^@]+@[^@]+\\.[^@]+$"
      - name: created_at
        data_tests:
          - dbt_expectations.expect_column_values_to_be_of_type:
              column_type: timestamp
      - name: age
        data_tests:
          - dbt_expectations.expect_column_mean_to_be_between:
              min_value: 18
              max_value: 70
          - dbt_expectations.expect_column_quantile_values_to_be_between:
              quantile: .95
              max_value: 90
```

Es la forma más directa de llevar a dbt el tipo de validaciones estadísticas que en Python harías con [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]] (ver `Python/Pandera/`).

## codegen — no escribas YAML a mano

```bash
# YAML de sources a partir de un esquema existente
dbt run-operation generate_source \
  --args '{"schema_name": "raw", "generate_columns": true, "include_descriptions": true}'

# SQL base de staging para una tabla cruda
dbt run-operation generate_base_model \
  --args '{"source_name": "raw", "table_name": "orders"}'

# YAML de un modelo con todas sus columnas
dbt run-operation generate_model_yaml \
  --args '{"model_names": ["fct_orders", "dim_customers"]}'

# CTEs de importación para un modelo
dbt run-operation generate_model_import_ctes --args '{"model_name": "fct_orders"}'
```

## audit_helper — refactorizar sin miedo

Al migrar un stored procedure o reescribir un modelo, compara la salida vieja contra la nueva:

```sql
-- analyses/compare_fct_orders.sql
{{ audit_helper.compare_relations(
    a_relation=ref('fct_orders'),
    b_relation=api.Relation.create(database='analytics', schema='legacy', identifier='orders_report'),
    primary_key='order_id'
) }}
```

Devuelve el porcentaje de filas idénticas, solo en A y solo en B. `compare_column_values` muestra el detalle por columna.

## Otros packages populares

| Package | Para qué |
|---|---|
| `dbt-labs/dbt_project_evaluator` | Linter de buenas prácticas del DAG: modelos sin tests, fuentes referenciadas fuera de staging, fan-outs excesivos... |
| `elementary-data/elementary` | Observabilidad: detección de anomalías en volumen/frescura/distribuciones, reporte HTML, alertas a Slack |
| `dbt-labs/dbt_external_tables` | Crear tablas externas (S3/GCS) como sources |
| `dbt-labs/spark_utils`, `snowflake_spend` | Utilidades específicas por plataforma |
| `fivetran/*` | Modelos listos para fuentes de Fivetran (Salesforce, HubSpot, Stripe...) |
| `godatadriven/dbt_date` | Macros de fechas y dimensión calendario |

## Escribir tu propio package

Un package es simplemente un proyecto dbt sin `profiles.yml`:

```text
dbt-shared-macros/
├── dbt_project.yml       # name: shared_macros
├── macros/
│   ├── cents_to_dollars.sql
│   └── tests/
│       └── test_is_positive.sql
├── integration_tests/    # un proyecto dbt que instala el package con `local: ../` y lo prueba
└── README.md
```

```yaml
# dbt_project.yml del package
name: 'shared_macros'
version: '1.0.0'
config-version: 2
require-dbt-version: [">=1.8.0", "<2.0.0"]
```

Uso desde otro proyecto: `{{ shared_macros.cents_to_dollars('amount') }}`.

### Sobrescribir macros de un package

Si una macro de un package no hace lo que necesitas, configura el orden de búsqueda:

```yaml
# dbt_project.yml
dispatch:
  - macro_namespace: dbt_utils
    search_order: ['ecommerce', 'dbt_utils']   # tu proyecto primero
```

## Ver también

- [[09 - Jinja y Macros]]
- [[11 - Tests de Datos y Unit Tests]]
- [[08 - Snapshots y SCD Tipo 2]] — `mutually_exclusive_ranges` para validar SCD2
- `Python/Pandera/` — validación de datos en el lado Python
- [[13-Testing-en-Machine-Learning]] (en `Machine Learning/`) — Great Expectations en contexto
