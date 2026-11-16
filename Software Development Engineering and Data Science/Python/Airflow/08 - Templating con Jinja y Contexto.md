---
tags: [airflow, jinja, templating, contexto, macros, cheat-sheet]
---

# 08 — Templating con Jinja y Contexto

> Continúa de [[07 - XComs y Paso de Datos]]. Airflow renderiza los **campos templated** de los operadores con **Jinja** justo antes de ejecutar cada tarea. Así puedes escribir `WHERE fecha = '{{ ds }}'` y que cada ejecución procese **su** fecha. Esta nota cubre las variables de contexto, las macros, los archivos `.sql` templated, los `params` y `render_template_as_native_obj`. La sintaxis de Jinja es la misma que usa dbt (ver [[09 - Jinja y Macros|dbt/09]]).

## ¿Qué se renderiza?

Solo los atributos listados en `template_fields` de cada operador. Algunos ejemplos:

| Operador | `template_fields` |
|---|---|
| `BashOperator` | `bash_command`, `env`, `cwd` |
| `PythonOperator` | `op_args`, `op_kwargs`, `templates_dict` |
| `SQLExecuteQueryOperator` | `sql`, `parameters`, `conn_id`... |
| `KubernetesPodOperator` | `image`, `cmds`, `arguments`, `env_vars`... |
| `TriggerDagRunOperator` | `trigger_dag_id`, `conf`, `logical_date` |

```python
# Consultarlo por código
from airflow.providers.standard.operators.bash import BashOperator
print(BashOperator.template_fields)
```

En `@task`, el cuerpo de la función **no** se renderiza (es Python), así que usa las variables de contexto como parámetros (ver [[05 - TaskFlow API#Acceder al contexto de ejecución]]). Los **argumentos** que pasas al llamar la tarea sí pueden ser strings templated.

```bash
airflow tasks render mi_dag mi_tarea 2026-11-14     # ver el resultado renderizado
```

En la UI, cada Task Instance tiene una pestaña **"Rendered Template"**.

---

## Variables de contexto

| Variable | Ejemplo (run diario del 2026-11-14) | Descripción |
|---|---|---|
| `{{ ds }}` | `2026-11-14` | Logical date en formato `YYYY-MM-DD` |
| `{{ ds_nodash }}` | `20261114` | Sin guiones |
| `{{ ts }}` | `2026-11-14T00:00:00+00:00` | Timestamp ISO |
| `{{ ts_nodash }}` | `20261114T000000` | |
| `{{ logical_date }}` | objeto pendulum | Fecha lógica del run (antes `execution_date`) |
| `{{ data_interval_start }}` | objeto pendulum | Inicio del intervalo de datos |
| `{{ data_interval_end }}` | objeto pendulum | Fin del intervalo de datos |
| `{{ prev_data_interval_start_success }}` | pendulum o None | Inicio del intervalo del último run exitoso |
| `{{ prev_start_date_success }}` | pendulum o None | Cuándo arrancó el último run exitoso |
| `{{ run_id }}` | `scheduled__2026-11-14T00:00:00+00:00` | ID del DAG Run |
| `{{ dag }}`, `{{ task }}` | objetos | DAG y tarea actuales |
| `{{ dag_run }}` | objeto | `dag_run.conf`, `dag_run.run_type`... |
| `{{ ti }}` / `{{ task_instance }}` | objeto | `ti.xcom_pull(...)`, `ti.try_number`, `ti.map_index` |
| `{{ params }}` | dict | Parámetros del DAG validados (nota 04) |
| `{{ var.value.mi_var }}` | str | Variable de Airflow |
| `{{ var.json.config.region }}` | valor | Variable JSON navegada |
| `{{ conn.mi_conn.host }}` | str | Atributos de una Connection (`host`, `login`, `password`, `schema`, `port`, `extra_dejson`) |
| `{{ macros }}` | módulo | Funciones auxiliares (ver abajo) |
| `{{ triggering_asset_events }}` | dict | Eventos de assets que dispararon el run (nota 11) |
| `{{ outlet_events }}` / `{{ inlet_events }}` | accesores | Metadata de assets |

> **Eliminadas en Airflow 3:** `execution_date`, `next_execution_date`, `prev_execution_date`, `next_ds`, `prev_ds`, `yesterday_ds`, `tomorrow_ds` y sus variantes `_nodash`. Usa `logical_date`, `data_interval_start`/`data_interval_end` y aritmética con pendulum o macros.

> ⚠️ **`logical_date` puede ser `None`** en Airflow 3 para runs disparados manualmente sin fecha o por assets. En esos casos, `ds` y los data intervals tampoco existen. Si tu tarea depende de la fecha, asegúrate de que el DAG tenga un horario de tiempo o maneja el caso `None`.

### logical_date vs. data interval: la semántica que más confunde

La relación depende del **timetable** (ver [[10 - Scheduling, Timetables y Backfills]]):

| Timetable | Run que se ejecuta el 15/11 a las 00:00 | `logical_date` | `data_interval_start` → `end` |
|---|---|---|---|
| `CronTriggerTimetable` (**default en 3.x** para cron y presets) | Se dispara el 15/11 00:00 | 15/11 00:00 | 15/11 00:00 → 15/11 00:00 (intervalo vacío) |
| `CronDataIntervalTimetable` (default en 2.x) | Se ejecuta **al final** del intervalo | 14/11 00:00 | 14/11 00:00 → 15/11 00:00 |

Consecuencia práctica: en Airflow 3 con `schedule="@daily"`, **`{{ ds }}` es el día de ejecución**, no "ayer". Para procesar el día anterior:

```sql
WHERE fecha = '{{ macros.ds_add(ds, -1) }}'
-- o
WHERE fecha = '{{ logical_date.subtract(days=1) | ds }}'
```

O bien configura el comportamiento de 2.x: `create_cron_data_intervals = True` en `[scheduler]`, o usa explícitamente `CronDataIntervalTimetable` en el DAG.

---

## Macros

```jinja
{{ macros.ds_add(ds, 7) }}                                   {# 2026-11-21 #}
{{ macros.ds_add(ds, -1) }}                                  {# ayer #}
{{ macros.ds_format(ds, "%Y-%m-%d", "%d/%m/%Y") }}            {# 14/11/2026 #}
{{ macros.datetime.now() }}
{{ macros.timedelta(days=3) }}
{{ macros.dateutil.relativedelta.relativedelta(months=1) }}
{{ macros.uuid.uuid4() }}
{{ macros.random() }}
{{ (logical_date - macros.timedelta(days=30)) | ds }}        {# filtro ds sobre un datetime #}
{{ data_interval_end.in_timezone("America/Lima").strftime("%Y-%m") }}
{{ logical_date.start_of("month") | ds }}                    {# primer día del mes (pendulum) #}
```

Filtros incluidos: `ds`, `ds_nodash`, `ts`, `ts_nodash`, `ts_nodash_with_tz`.

### Macros y filtros propios

```python
def trimestre(fecha) -> str:
    return f"{fecha.year}-Q{(fecha.month - 1) // 3 + 1}"

@dag(
    user_defined_macros={"trimestre": trimestre, "empresa": "ACME"},
    user_defined_filters={"mayus": lambda s: s.upper()},
    ...
)
def reportes():
    BashOperator(task_id="r", bash_command="echo {{ empresa | mayus }} {{ trimestre(logical_date) }}")
```

---

## Archivos `.sql` (y `.sh`) templated

Si un campo templated termina en una extensión de `template_ext` (por ejemplo, `.sql`), Airflow **lee el archivo y lo renderiza**:

```
dags/
└── ventas/
    ├── ventas_diarias.py
    └── sql/
        └── upsert_ventas.sql
```

```sql
-- sql/upsert_ventas.sql
DELETE FROM silver.ventas WHERE fecha = '{{ ds }}';

INSERT INTO silver.ventas
SELECT *
FROM bronze.ventas
WHERE fecha = '{{ ds }}'
  AND region IN ({% for r in params.regiones %}'{{ r }}'{% if not loop.last %}, {% endif %}{% endfor %});
```

```python
from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator

@dag(template_searchpath=["/opt/airflow/dags/ventas/sql"],
     params={"regiones": ["LATAM", "EMEA"]}, ...)
def ventas_diarias():
    SQLExecuteQueryOperator(
        task_id="upsert",
        conn_id="postgres_dwh",
        sql="upsert_ventas.sql",          # busca en template_searchpath o junto al DAG
    )
```

Ventajas: SQL con resaltado en el editor, testeable aparte y diffs limpios en Git.

### Parámetros SQL vs. templating

Para **valores** que vienen de fuera (`dag_run.conf`, entradas del usuario), usa **parámetros de la base** en lugar de interpolar con Jinja, para evitar SQL injection:

```python
SQLExecuteQueryOperator(
    task_id="seguro",
    conn_id="postgres_dwh",
    sql="SELECT * FROM ventas WHERE region = %(region)s AND fecha = %(fecha)s",
    parameters={"region": "{{ params.region }}", "fecha": "{{ ds }}"},
)
```

---

## `render_template_as_native_obj`

Por defecto, Jinja **siempre devuelve strings**. Si un campo templated debe ser una lista, un dict o un número, activa los objetos nativos:

```python
@dag(render_template_as_native_obj=True, ...)
def nativo():
    PythonOperator(
        task_id="t",
        python_callable=lambda cfg: print(type(cfg), cfg),
        op_args=["{{ dag_run.conf['regiones'] }}"],   # llega como list, no como "['LATAM']"
    )
```

## Escapar llaves

```python
# Bash con variables de shell ${...}: no hay conflicto. Para llaves dobles literales:
bash_command="echo '{{ '{{' }} literal {{ '}}' }}'"
# o un bloque raw
bash_command="{% raw %}echo {{ esto_no_se_renderiza }}{% endraw %}"
```

Ojo con las f-strings de Python que generan templates: `f"{{{{ ds }}}}"` produce `{{ ds }}`. Ver [[09 - String Formatting]].

## Ver también

- [[04 - DAGs - Definición y Parámetros#params: parámetros de ejecución con validación]]
- [[07 - XComs y Paso de Datos]]
- [[10 - Scheduling, Timetables y Backfills]]: la semántica de `logical_date`
- [[13 - Connections, Variables, Hooks y Providers]]: `var` y `conn` en templates
- [[09 - Jinja y Macros|dbt/09]]: el mismo Jinja aplicado a SQL en dbt
- [[53 - Fechas y Zonas Horarias]] (en `Python/`)
