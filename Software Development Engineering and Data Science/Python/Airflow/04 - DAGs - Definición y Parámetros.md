---
tags: [airflow, dag, parametros, default-args, cheat-sheet]
---

# 04 — DAGs: Definición y Parámetros

> Continúa de [[03 - CLI de Airflow]]. Esta nota cubre **cómo se declara un DAG** (decorador, context manager o constructor), todos sus parámetros importantes, `default_args`, los `params` de ejecución y cómo descubre Airflow los archivos de DAGs.

## Tres formas de declarar un DAG

### 1. Decorador `@dag` (recomendado con TaskFlow)

```python
import pendulum
from airflow.sdk import dag, task

@dag(
    dag_id="ventas_diarias",               # opcional: por defecto, el nombre de la función
    schedule="0 6 * * *",
    start_date=pendulum.datetime(2026, 1, 1, tz="America/Lima"),
    catchup=False,
    tags=["ventas", "dwh"],
)
def ventas_diarias():
    @task
    def extraer(): ...
    extraer()

ventas_diarias()        # ⚠️ hay que LLAMAR a la función para registrar el DAG
```

### 2. Context manager `with DAG(...)`

```python
from airflow.sdk import DAG
from airflow.providers.standard.operators.bash import BashOperator

with DAG(
    dag_id="backup_nocturno",
    schedule="@daily",
    start_date=pendulum.datetime(2026, 1, 1, tz="UTC"),
    catchup=False,
) as dag:
    BashOperator(task_id="backup", bash_command="pg_dump ... > /backups/{{ ds }}.sql")
```

### 3. Constructor explícito (raro)

```python
dag = DAG(dag_id="x", schedule=None, start_date=...)
t1 = BashOperator(task_id="t1", bash_command="echo 1", dag=dag)
```

Las tres son equivalentes. El decorador es lo más natural con `@task`; el context manager es lo más común en código con operadores clásicos.

> **Airflow 3:** importa `dag`, `DAG` y `task` desde **`airflow.sdk`**. Las rutas viejas (`from airflow import DAG`, `airflow.decorators`) siguen funcionando con avisos de deprecación, pero el código nuevo debe usar el Task SDK.

---

## Parámetros del DAG

```python
@dag(
    # --- Identidad y documentación ---
    dag_id="ventas_diarias",
    description="Carga diaria de ventas a silver y gold",
    doc_md=__doc__,                          # Markdown visible en la UI
    tags=["ventas", "equipo-datos"],
    owner_links={"equipo-datos": "https://wiki.empresa.com/datos"},

    # --- Scheduling ---
    schedule="0 6 * * *",                    # cron, preset, timedelta, Asset, Timetable o None
    start_date=pendulum.datetime(2026, 1, 1, tz="America/Lima"),
    end_date=None,                           # opcional: deja de programarse después
    catchup=False,                           # no crear runs del pasado al activar el DAG

    # --- Concurrencia ---
    max_active_runs=1,                       # runs simultáneos de ESTE DAG
    max_active_tasks=8,                      # tareas simultáneas de ESTE DAG (antes "concurrency")
    max_consecutive_failed_dag_runs=5,       # se autopausa tras N runs fallidos seguidos

    # --- Ejecución ---
    default_args={...},                      # ver abajo
    dagrun_timeout=pendulum.duration(hours=2),   # el run falla si tarda más
    params={...},                            # parámetros de ejecución (ver abajo)

    # --- Templating ---
    template_searchpath=["/opt/airflow/include/sql"],
    user_defined_macros={"empresa": "ACME"},
    render_template_as_native_obj=False,     # ver nota 08

    # --- Callbacks a nivel DAG ---
    on_success_callback=None,
    on_failure_callback=None,

    # --- Otros ---
    is_paused_upon_creation=True,
    fail_fast=False,                         # True: al fallar una tarea, se detienen las demás
)
def ventas_diarias(): ...
```

| Parámetro | Error típico |
|---|---|
| `start_date` | Usar `datetime.now()`, que es dinámico: el DAG nunca se ejecuta o se comporta raro. **Siempre una fecha fija** |
| `start_date` sin zona horaria | Se interpreta en `core.default_timezone` (UTC por defecto); los horarios quedan desfasados |
| `catchup=True` con un `start_date` antiguo | Al activar el DAG se crean cientos de runs |
| Olvidar llamar a la función `@dag` | El DAG no aparece en la UI |
| `dag_id` duplicado en dos archivos | Uno pisa al otro sin aviso claro |
| Cambiar `dag_id` | Airflow lo trata como un DAG nuevo y se pierde el historial en la UI |

---

## `default_args`: valores por defecto para todas las tareas

`default_args` es un diccionario que se aplica a **cada tarea** del DAG. Cada tarea puede sobreescribir cualquier valor.

```python
from datetime import timedelta

default_args = {
    "owner": "equipo-datos",
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
    "retry_exponential_backoff": True,       # 5, 10, 20 min...
    "max_retry_delay": timedelta(hours=1),
    "execution_timeout": timedelta(minutes=30),
    "depends_on_past": False,                # True: espera a que la misma tarea del run anterior haya sido exitosa
    "email": ["datos@empresa.com"],
    "email_on_failure": False,               # mejor usar callbacks/notifiers (nota 17)
    "on_failure_callback": alerta_slack,
    "pool": "default_pool",
    "queue": "default",
    "priority_weight": 1,
}

@dag(default_args=default_args, ...)
def mi_dag():
    @task(retries=0)                          # sobreescribe: esta tarea no reintenta
    def tarea_no_idempotente(): ...
```

### Parámetros más útiles de las tareas (BaseOperator)

| Parámetro | Para qué |
|---|---|
| `task_id` | Identificador único dentro del DAG (en `@task`, por defecto el nombre de la función) |
| `retries`, `retry_delay`, `retry_exponential_backoff` | Reintentos |
| `execution_timeout` | Matar la tarea si tarda demasiado. **Defínelo siempre**: una tarea colgada ocupa un slot para siempre |
| `trigger_rule` | Cuándo se ejecuta según el estado de sus upstream (ver [[09 - Branching, Condicionales y Trigger Rules]]) |
| `depends_on_past` | Exige éxito de la misma tarea en el run anterior |
| `wait_for_downstream` | Como `depends_on_past`, pero exige también el éxito de los downstream del run anterior |
| `pool`, `pool_slots` | Limitar la concurrencia contra un recurso (ver [[16 - Executors, Escalado y Despliegue]]) |
| `priority_weight`, `weight_rule` | Prioridad en la cola |
| `queue` | Cola de Celery (por ejemplo, workers con GPU) |
| `executor` | Ejecutar esta tarea con otro executor (multi-executor) |
| `do_xcom_push` | Desactivar el push automático del valor de retorno |
| `doc_md` | Documentación visible en la UI |
| `on_failure_callback`, `on_success_callback`, `on_retry_callback`, `on_skipped_callback` | Hooks de eventos |
| `outlets`, `inlets` | Assets que produce o consume (ver [[11 - Assets y Scheduling Event-Driven]]) |
| `map_index_template` | Nombre legible de las instancias mapeadas (ver [[12 - Dynamic Task Mapping]]) |

---

## params: parámetros de ejecución con validación

Los `params` definen entradas del DAG con valores por defecto y **validación JSON Schema**. Al disparar el DAG manualmente, la UI muestra un **formulario** con esos campos.

```python
from airflow.sdk import dag, task, Param

@dag(
    schedule=None,
    start_date=pendulum.datetime(2026, 1, 1),
    params={
        "region": Param("LATAM", type="string", enum=["LATAM", "EMEA", "NA"]),
        "dias_atras": Param(7, type="integer", minimum=1, maximum=90,
                            description="Cuántos días reprocesar"),
        "full_refresh": Param(False, type="boolean"),
        "email": Param(None, type=["null", "string"], format="email"),
        "modelos": Param(["churn"], type="array", items={"type": "string"}),
    },
)
def reproceso_manual():

    @task
    def ejecutar(params: dict | None = None):
        # params llega por inyección de contexto (nota 08)
        print(params["region"], params["dias_atras"])

    ejecutar()

reproceso_manual()
```

```python
# En templates
BashOperator(task_id="t", bash_command="procesar.sh --region {{ params.region }}")
```

```bash
# Desde la CLI: --conf sobreescribe los params
airflow dags trigger reproceso_manual --conf '{"region": "EMEA", "dias_atras": 30}'
```

> `dag_run.conf` contiene la configuración cruda enviada al disparar; `params` contiene los valores **ya fusionados con los defaults y validados**. Usa `params`.

---

## Documentar el DAG

```python
"""
### Ventas diarias
Carga las ventas del día anterior desde el ERP a `silver.ventas` y recalcula `gold.ventas_mensuales`.

**Owner:** equipo-datos · **SLA:** 08:00 Lima
"""

@dag(doc_md=__doc__, ...)
def ventas_diarias():
    @task(doc_md="Extrae de Postgres réplica de lectura. Idempotente por fecha.")
    def extraer(): ...
```

## Cómo descubre Airflow los DAGs

- El **DAG Processor** recorre la carpeta de DAGs (o los **DAG bundles** configurados, por ejemplo un repositorio Git) y **ejecuta cada archivo `.py`** para encontrar objetos DAG.
- Por defecto, solo procesa archivos que contienen las palabras `airflow` y `dag` (modo seguro, `core.dag_discovery_safe_mode`).
- `.airflowignore` excluye rutas (sintaxis regex o glob):

```
# dags/.airflowignore
include/
.*_test\.py
sql/
```

- Cada archivo se reparsea periódicamente. Por eso **el código a nivel de módulo debe ser rápido y sin efectos secundarios**: nada de consultas a bases de datos, llamadas a APIs ni imports pesados fuera de las tareas. Ver [[17 - Testing, Observabilidad y Buenas Prácticas#Código top-level]].

```python
# ❌ MAL: se ejecuta en cada parseo (cada ~30-60 s), no cuando corre el DAG
import pandas as pd
clientes = pd.read_sql("SELECT * FROM clientes", engine)

# ✅ BIEN: dentro de la tarea, e import local
@task
def leer_clientes():
    import pandas as pd
    return ...
```

### DAG versioning (Airflow 3)

Cada vez que cambia la estructura de un DAG, Airflow 3 guarda una **nueva versión**. La UI muestra cada run con el grafo y el código **de la versión con la que corrió**, no del código actual. Con **DAG bundles versionados** (por ejemplo, `GitDagBundle`), un run en curso puede incluso terminar con la versión de código con la que empezó.

```bash
# airflow.cfg: [dag_processor] dag_bundle_config_list
export AIRFLOW__DAG_PROCESSOR__DAG_BUNDLE_CONFIG_LIST='[
  {"name": "repo_datos", "classpath": "airflow.providers.git.bundles.git.GitDagBundle",
   "kwargs": {"git_conn_id": "github_datos", "tracking_ref": "main", "subdir": "dags"}}
]'
```

## Generar DAGs dinámicamente

Para muchos DAGs casi idénticos (uno por cliente o por fuente), genéralos en un bucle a partir de una configuración **estática** (un YAML del repositorio, no una consulta a una base de datos):

```python
import yaml
from pathlib import Path

config = yaml.safe_load((Path(__file__).parent / "fuentes.yaml").read_text())

def crear_dag(fuente: dict):
    @dag(dag_id=f"ingesta_{fuente['nombre']}", schedule=fuente["schedule"],
         start_date=pendulum.datetime(2026, 1, 1), catchup=False, tags=["ingesta"])
    def _dag():
        @task
        def ingerir():
            print(f"Ingiriendo {fuente['url']}")
        ingerir()
    return _dag()

for f in config["fuentes"]:
    crear_dag(f)
```

Si la variación es **por ejecución** (por ejemplo, la lista de archivos de hoy) y no por estructura, usa [[12 - Dynamic Task Mapping]] en lugar de varios DAGs. Ver [[14 - JSON and YAML]] para YAML en Python.

## Ver también

- [[05 - TaskFlow API]]
- [[06 - Operadores Clásicos y Dependencias]]
- [[08 - Templating con Jinja y Contexto]]
- [[10 - Scheduling, Timetables y Backfills]]
- [[16 - Decorators]] (en `Python/`): cómo funcionan los decoradores como `@dag` y `@task`
