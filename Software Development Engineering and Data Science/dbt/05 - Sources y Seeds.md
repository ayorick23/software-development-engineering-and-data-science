---
tags: [dbt, sources, seeds, freshness, cheat-sheet]
---

# 05 — Sources y Seeds

## Sources: declarar los datos crudos

Un **source** es una tabla que **dbt no crea**: la cargó Fivetran, Airbyte, un script de Python o un `COPY INTO`. Declararla en YAML permite:

- Referenciarla con `{{ source() }}` en vez de un nombre hardcodeado.
- Que aparezca como nodo raíz en el **lineage**.
- Testearla y documentarla igual que un modelo.
- Monitorear su **frescura** (¿llegaron datos nuevos?).
- Seleccionar todo lo que depende de ella: `dbt build -s source:raw.orders+`.

### Declaración

```yaml
# models/staging/_sources.yml
version: 2

sources:
  - name: raw                      # nombre lógico usado en source('raw', ...)
    description: "Datos crudos de la app transaccional, cargados por Airbyte."
    database: dev                  # opcional; default: el del target
    schema: raw                    # default: igual a `name`
    loader: airbyte
    tables:
      - name: customers
        description: "Una fila por cliente registrado."
        columns:
          - name: id
            data_tests: [unique, not_null]

      - name: orders
        identifier: tbl_orders_v2  # nombre físico distinto del nombre lógico
        columns:
          - name: id
            data_tests: [unique, not_null]
          - name: status
            data_tests:
              - accepted_values:
                  values: ['placed', 'shipped', 'completed', 'return_pending', 'returned']

      - name: payments
```

### Uso

```sql
select * from {{ source('raw', 'orders') }}
-- compila a: "dev"."raw"."tbl_orders_v2"
```

> **Regla:** `source()` solo debería aparecer en los modelos de **staging**. Todo lo demás usa `ref()` hacia staging. Así, si la fuente cambia (nueva columna, renombre), solo tocas un archivo. Ver [[13 - Estructura de Proyecto y Modelado]].

### Sources por entorno

Si dev y prod leen de bases distintas, usa Jinja en el YAML:

```yaml
sources:
  - name: raw
    database: "{{ 'raw_prod' if target.name == 'prod' else 'raw_dev' }}"
    schema: app
```

## Source freshness

Detecta cuando la carga de datos se atrasa — por ejemplo, Airbyte falló anoche y el dashboard muestra datos de ayer sin que nadie lo note.

```yaml
sources:
  - name: raw
    config:
      freshness:                       # default para todas las tablas del source
        warn_after:  {count: 12, period: hour}
        error_after: {count: 24, period: hour}
      loaded_at_field: _airbyte_extracted_at
    tables:
      - name: orders
        config:
          loaded_at_field: updated_at   # sobreescribe a nivel tabla
          freshness:
            warn_after:  {count: 1, period: hour}
            error_after: {count: 6, period: hour}
            filter: "updated_at > current_date - interval '7 days'"   # limita el escaneo
      - name: customers
        config:
          freshness: null               # sin chequeo para esta tabla
```

> En versiones anteriores a dbt 1.10 `freshness` y `loaded_at_field` iban directamente bajo la tabla (sin `config:`). Ambas formas siguen apareciendo en proyectos reales.

```bash
dbt source freshness
dbt source freshness -s source:raw.orders
```

```text
1 of 2 START freshness of raw.orders ........ [RUN]
1 of 2 WARN freshness of raw.orders ......... [WARN in 0.12s]
2 of 2 PASS freshness of raw.customers ...... [PASS in 0.09s]
```

Qué hace por dentro: `select max(updated_at) from raw.orders` y compara contra la hora actual. El resultado queda en `target/sources.json`.

### Construir solo lo que tiene datos nuevos

Combinando freshness con state:

```bash
dbt source freshness
dbt build -s source_status:fresher+ --state ./prod-artifacts
```

Solo reconstruye lo que depende de sources que recibieron datos desde la última ejecución — ahorra mucho cómputo en proyectos grandes.

## Sources externas (archivos) con dbt-duckdb

DuckDB puede leer Parquet/CSV/JSON directamente, incluso desde S3. `dbt-duckdb` lo expone en la definición del source:

```yaml
sources:
  - name: lake
    meta:
      external_location: "s3://mi-bucket/raw/{name}/*.parquet"
    tables:
      - name: events        # lee s3://mi-bucket/raw/events/*.parquet
      - name: sessions
        meta:
          external_location: "read_csv_auto('data/sessions.csv', header=true)"
```

Es un patrón excelente para practicar un **lakehouse local**: Parquet como almacenamiento, DuckDB como motor, dbt como capa de transformación. Para escribir modelos de vuelta a Parquet, ver la materialización `external` en [[06 - Materializaciones]].

## Seeds: CSVs versionados

Un **seed** es un CSV en `seeds/` que `dbt seed` carga como tabla. Está pensado para **datos de referencia pequeños y estáticos** que cambian poco y tiene sentido versionar en Git:

- Mapeos de códigos (`country_code → country_name`).
- Listas de exclusión (cuentas de prueba, emails internos).
- Tablas de parámetros de negocio (metas mensuales, comisiones).

```csv
# seeds/payment_methods.csv
payment_method,display_name,is_digital
credit_card,Tarjeta de crédito,true
bank_transfer,Transferencia,true
coupon,Cupón,false
gift_card,Tarjeta regalo,false
```

```bash
dbt seed                            # carga todos
dbt seed -s payment_methods
dbt seed --full-refresh             # drop + recreate (necesario si cambian columnas)
```

```sql
-- se referencia igual que un modelo
select p.*, pm.display_name
from {{ ref('stg_payments') }} p
left join {{ ref('payment_methods') }} pm using (payment_method)
```

### Configurar tipos de columna

dbt infiere tipos, y a veces se equivoca (códigos postales con ceros a la izquierda, por ejemplo):

```yaml
# seeds/_seeds.yml
version: 2
seeds:
  - name: payment_methods
    description: "Catálogo de métodos de pago."
    config:
      column_types:
        payment_method: varchar
        is_digital: boolean
      delimiter: ","
      quote_columns: false
    columns:
      - name: payment_method
        data_tests: [unique, not_null]
```

### Cuándo NO usar seeds

| Situación | Por qué no | Alternativa |
|---|---|---|
| Archivos de > ~1 MB o miles de filas | La carga es lenta y llena el repo Git | Herramienta de ingesta o source externa |
| Datos que cambian a diario | Requiere un commit por cada cambio | Source con carga automatizada |
| Datos sensibles (PII) | Quedan en el historial de Git para siempre | Cargar en el warehouse con controles de acceso |
| Datos de producción | Los seeds son configuración, no datos | Pipeline de ingesta |

## Sources vs. seeds vs. models

| | Source | Seed | Model |
|---|---|---|---|
| ¿Quién crea la tabla? | Un proceso externo | `dbt seed` desde un CSV | `dbt run` desde un SELECT |
| Se referencia con | `source('x', 'y')` | `ref('y')` | `ref('y')` |
| Se puede testear | Sí | Sí | Sí |
| Vive en | El warehouse | `seeds/*.csv` en Git | `models/*.sql` en Git |
| Freshness | Sí | No | No (usa tests de recencia) |

## Generar staging automáticamente

El package `codegen` (ver [[10 - Packages]]) genera el YAML del source y el SQL base de staging leyendo el esquema real:

```bash
dbt run-operation generate_source --args '{"schema_name": "raw", "generate_columns": true}'
dbt run-operation generate_base_model --args '{"source_name": "raw", "table_name": "orders"}'
```

Copias la salida en los archivos correspondientes y solo ajustas renombres y casts.

## Ver también

- [[04 - Models, ref() y el DAG]]
- [[11 - Tests de Datos y Unit Tests]]
- [[13 - Estructura de Proyecto y Modelado]]
- [[44-Ingesta-Incremental-CDC-e-Idempotencia]] (en `Machine Learning/`) — la capa de ingesta que llena los sources
- [[04-Ingenieria-de-Datos#Parquet|Parquet]] (en `Machine Learning/04-Ingenieria-de-Datos`)
