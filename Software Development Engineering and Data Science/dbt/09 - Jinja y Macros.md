---
tags: [dbt, jinja, macros, hooks, cheat-sheet]
---

# 09 — Jinja y Macros

**Jinja** es un motor de plantillas de Python (el mismo que usa Flask). dbt lo usa para convertir SQL estático en SQL **programable**: variables, condicionales, loops y funciones reutilizables (macros). Todo el Jinja se resuelve en la fase de **compile**: el warehouse solo recibe SQL puro.

## Sintaxis básica

| Delimitador | Uso | Ejemplo |
|---|---|---|
| `{{ ... }}` | **Expresión**: imprime un valor en el SQL | `{{ ref('stg_orders') }}` |
| `{% ... %}` | **Sentencia**: lógica (if, for, set, macro) sin imprimir | `{% if target.name == 'prod' %}` |
| `{# ... #}` | **Comentario** Jinja: no aparece en el SQL compilado | `{# TODO: revisar #}` |
| `{%- ... -%}` | Igual, pero **elimina espacios/saltos de línea** alrededor | `{%- set x = 1 -%}` |

```sql
{# Variables #}
{% set payment_methods = ['credit_card', 'bank_transfer', 'coupon', 'gift_card'] %}

select
    order_id,
    {# Loop: genera una columna por método de pago #}
    {% for method in payment_methods %}
    sum(case when payment_method = '{{ method }}' then amount else 0 end) as {{ method }}_amount
    {%- if not loop.last %},{% endif %}
    {% endfor %}
from {{ ref('stg_payments') }}
group by 1
```

SQL compilado:

```sql
select
    order_id,
    sum(case when payment_method = 'credit_card' then amount else 0 end) as credit_card_amount,
    sum(case when payment_method = 'bank_transfer' then amount else 0 end) as bank_transfer_amount,
    sum(case when payment_method = 'coupon' then amount else 0 end) as coupon_amount,
    sum(case when payment_method = 'gift_card' then amount else 0 end) as gift_card_amount
from "dev"."main_staging"."stg_payments"
group by 1
```

> **Depura siempre mirando el SQL compilado:** `dbt compile -s modelo` y abre `target/compiled/...`. Es la forma más rápida de entender qué hace un Jinja confuso.

### Condicionales y filtros

```sql
{% if target.name == 'dev' %}
  -- en dev, solo los últimos 30 días para iterar rápido
  where order_date >= current_date - interval '30 days'
{% elif target.name == 'ci' %}
  limit 1000
{% endif %}
```

Filtros de Jinja frecuentes: `| lower`, `| upper`, `| trim`, `| replace('a','b')`, `| join(', ')`, `| length`, `| default('x')`, `| tojson`. Concatenación de strings con `~`: `{{ 'stg_' ~ table_name }}`.

### Loop variables

`loop.index` (desde 1), `loop.index0`, `loop.first`, `loop.last`, `loop.length`.

## Objetos y funciones de contexto de dbt

| Objeto / función | Qué devuelve |
|---|---|
| `ref()`, `source()` | Relaciones (ver [[04 - Models, ref() y el DAG]]) |
| `this` | La relación del modelo actual (`this.database`, `this.schema`, `this.identifier`) |
| `target` | El target activo: `target.name`, `target.schema`, `target.type` (duckdb, snowflake...), `target.threads` |
| `var('nombre', default)` | Variable del proyecto o de `--vars` |
| `env_var('NOMBRE', default)` | Variable de entorno |
| `config.get('clave')` | Leer una config del modelo |
| `model` | Metadatos del nodo actual (`model.name`, `model.tags`, `model.config`) |
| `execute` | `false` durante el parse, `true` durante la ejecución |
| `run_query(sql)` | Ejecuta SQL y devuelve una tabla Agate |
| `log(msg, info=true)` | Imprime en la consola/logs |
| `exceptions.raise_compiler_error(msg)` | Aborta la compilación con un error |
| `adapter.get_columns_in_relation(rel)` | Columnas reales de una tabla |
| `modules.datetime`, `modules.re` | Módulos de Python permitidos |
| `invocation_id`, `run_started_at` | Identificador y timestamp de la ejecución |

### var() y env_var()

```yaml
# dbt_project.yml
vars:
  start_date: '2024-01-01'
  excluded_emails: ['test@empresa.com', 'qa@empresa.com']
```

```sql
select *
from {{ ref('stg_orders') }}
where order_date >= '{{ var("start_date") }}'
  and email not in (
    {%- for e in var('excluded_emails') %}'{{ e }}'{% if not loop.last %}, {% endif %}{% endfor -%}
  )
```

```bash
dbt run --vars '{"start_date": "2025-01-01"}'
```

`env_var()` se usa en `profiles.yml` y en configs; dbt **oculta en los logs** las variables cuyo nombre empieza con `DBT_ENV_SECRET_`.

## Macros

Una **macro** es una función Jinja definida en `macros/*.sql`. Es el mecanismo de reutilización de dbt (DRY).

```sql
-- macros/cents_to_dollars.sql
{% macro cents_to_dollars(column_name, scale=2) -%}
    round(({{ column_name }} / 100.0)::numeric, {{ scale }})
{%- endmacro %}
```

```sql
-- models/staging/stg_payments.sql
select
    id as payment_id,
    order_id,
    payment_method,
    {{ cents_to_dollars('amount') }} as amount
from {{ source('raw', 'payments') }}
```

Documentación y argumentos de macros en YAML:

```yaml
# macros/_macros.yml
version: 2
macros:
  - name: cents_to_dollars
    description: "Convierte una columna en centavos a dólares."
    arguments:
      - name: column_name
        type: string
        description: "Nombre de la columna en centavos."
      - name: scale
        type: integer
        description: "Decimales (default 2)."
```

### Macro que genera un bloque entero

```sql
-- macros/pivot_payments.sql
{% macro pivot_by(column, values, agg_column, agg='sum') %}
    {% for v in values %}
    {{ agg }}(case when {{ column }} = '{{ v }}' then {{ agg_column }} else 0 end) as {{ v | lower | replace(' ', '_') }}_{{ agg_column }}
    {%- if not loop.last %},{% endif %}
    {% endfor %}
{% endmacro %}
```

### Valores dinámicos desde el warehouse: run_query

En vez de hardcodear la lista de métodos de pago, consultarla:

```sql
{% set methods_query %}
    select distinct payment_method from {{ ref('stg_payments') }} order by 1
{% endset %}

{% set results = run_query(methods_query) %}

{% if execute %}
    {% set payment_methods = results.columns[0].values() %}
{% else %}
    {% set payment_methods = [] %}
{% endif %}

select
    order_id,
    {{ pivot_by('payment_method', payment_methods, 'amount') }}
from {{ ref('stg_payments') }}
group by 1
```

> **¿Por qué `if execute`?** Durante el parse, dbt aún no ha construido nada y `run_query` no devuelve resultados. Sin la guarda, `results.columns` falla. Es el error de Jinja más común en dbt.

`dbt_utils.get_column_values()` hace exactamente esto ya empaquetado (ver [[10 - Packages]]).

### Macros que se ejecutan solas: run-operation

```sql
-- macros/grant_select.sql
{% macro grant_select(role) %}
    {% set sql %}
        grant usage on schema {{ target.schema }} to role {{ role }};
        grant select on all tables in schema {{ target.schema }} to role {{ role }};
    {% endset %}
    {% do run_query(sql) %}
    {{ log("Permisos otorgados a " ~ role, info=true) }}
{% endmacro %}
```

```bash
dbt run-operation grant_select --args '{role: reporter}'
```

`{% do ... %}` ejecuta una expresión sin imprimir nada.

### Macros multi-warehouse: adapter.dispatch

Para que una macro genere SQL distinto según el warehouse:

```sql
{% macro datediff_days(start, end) %}
    {{ return(adapter.dispatch('datediff_days')(start, end)) }}
{% endmacro %}

{% macro default__datediff_days(start, end) %}
    datediff('day', {{ start }}, {{ end }})
{% endmacro %}

{% macro bigquery__datediff_days(start, end) %}
    date_diff({{ end }}, {{ start }}, day)
{% endmacro %}

{% macro postgres__datediff_days(start, end) %}
    ({{ end }}::date - {{ start }}::date)
{% endmacro %}
```

dbt elige `<adapter>__nombre` y cae a `default__nombre` si no existe. Así funcionan las macros cross-database de `dbt_utils` y del propio dbt (`dbt.datediff`, `dbt.dateadd`, `dbt.safe_cast`, `dbt.concat`, `dbt.date_trunc`...), que conviene usar antes de escribir las propias.

### Sobrescribir macros internas de dbt

Algunas macros del core existen precisamente para ser sobrescritas creando una con el mismo nombre en tu proyecto:

| Macro | Para qué |
|---|---|
| `generate_schema_name` | Cómo se forma el nombre del esquema (ver [[04 - Models, ref() y el DAG]]) |
| `generate_alias_name` | Cómo se forma el nombre de la tabla |
| `generate_database_name` | Cómo se forma el nombre de la base |
| `get_custom_alias`, materializaciones | Casos avanzados |

## Hooks

Los **hooks** ejecutan SQL antes/después de un modelo o de toda la ejecución.

```sql
-- a nivel modelo
{{ config(
    pre_hook="set timezone = 'UTC'",
    post_hook=[
        "grant select on {{ this }} to role reporter",
        "analyze {{ this }}"
    ]
) }}
```

```yaml
# dbt_project.yml — a nivel proyecto/carpeta y de ejecución
models:
  ecommerce:
    marts:
      +post-hook:
        - "grant select on {{ this }} to role bi_reader"

on-run-start:
  - "create schema if not exists {{ target.schema }}_audit"
on-run-end:
  - "{{ log_run_results(results) }}"
```

`on-run-end` recibe la variable `results` con el estado de cada nodo — útil para registrar auditoría:

```sql
-- macros/log_run_results.sql
{% macro log_run_results(results) %}
  {% if execute %}
    {% for r in results %}
      {{ log(r.node.unique_id ~ " -> " ~ r.status ~ " (" ~ r.execution_time | round(2) ~ "s)", info=true) }}
    {% endfor %}
  {% endif %}
{% endmacro %}
```

> Preferir `grants:` (config nativa) sobre `post_hook` con `GRANT` cuando el adapter lo soporte: es declarativo y dbt revoca lo sobrante.

## Cuándo NO usar Jinja

Jinja es poderoso y fácil de abusar. Un modelo que necesita leer `target/compiled` para entenderse es un mal modelo.

| Bien | Mal |
|---|---|
| Loops que eliminan repetición mecánica (pivots, 20 columnas iguales) | Loops que generan lógica de negocio que nadie entiende sin compilar |
| Macros para conversiones repetidas (`cents_to_dollars`) | Macros que esconden joins enteros |
| Diferencias por entorno (`target.name`) | Lógica de negocio distinta en dev y prod |
| Usar `dbt_utils`/`dbt.*` | Reimplementar `dbt_utils` |

Regla de dbt Labs: **"favor readability over DRY-ness"**. Si duplicar 5 líneas de SQL lo hace más claro que una macro, duplica.

## Ver también

- [[04 - Models, ref() y el DAG]]
- [[10 - Packages]]
- [[07 - Modelos Incrementales]] — `is_incremental()` y `this`
- [[15 - dbt en Producción - CI-CD y Orquestación]] — variables de entorno en CI
- `Python/14 - JSON and YAML.md` — YAML, el otro lenguaje de dbt
