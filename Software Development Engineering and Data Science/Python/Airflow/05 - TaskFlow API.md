---
tags: [airflow, taskflow, task-decorator, python, cheat-sheet]
---

# 05 — TaskFlow API

> Continúa de [[04 - DAGs - Definición y Parámetros]]. La **TaskFlow API** (`@task`) es la forma moderna de escribir tareas: funciones de Python normales cuyo **valor de retorno** pasa automáticamente a la siguiente tarea vía XCom, y cuyas **dependencias se infieren** de las llamadas. Esta nota cubre `@task` y todas sus variantes (`virtualenv`, `docker`, `bash`, `kubernetes`...) y cómo mezclarlo con operadores clásicos.

## Lo básico

```python
import pendulum
from airflow.sdk import dag, task

@dag(schedule="@daily", start_date=pendulum.datetime(2026, 1, 1), catchup=False)
def etl_taskflow():

    @task
    def extraer() -> dict:
        return {"ventas": [120.5, 99.9, 300.0]}

    @task
    def transformar(datos: dict) -> float:
        return round(sum(datos["ventas"]), 2)

    @task
    def cargar(total: float) -> None:
        print(f"Total cargado: {total}")

    cargar(transformar(extraer()))      # define tareas Y dependencias: extraer >> transformar >> cargar

etl_taskflow()
```

Qué pasa por debajo:

1. Llamar a `extraer()` **dentro del DAG** no ejecuta la función. Crea la tarea y devuelve un **`XComArg`**, una referencia al resultado futuro.
2. Pasar ese `XComArg` a `transformar(...)` crea la dependencia `extraer >> transformar`.
3. En ejecución, el retorno de `extraer` se guarda como XCom y se pasa como argumento a `transformar`.

> Los valores de retorno deben ser **serializables** (JSON, dataclasses, tipos básicos, pendulum, `Asset`...) y **pequeños**. Para datos grandes, devuelve una **ruta** (por ejemplo, `s3://.../fecha=2026-11-14/`), nunca un DataFrame grande. Ver [[07 - XComs y Paso de Datos]].

## Parámetros de `@task`

```python
@task(
    task_id="extraer_ventas",          # por defecto: nombre de la función
    retries=3,
    retry_delay=pendulum.duration(minutes=2),
    execution_timeout=pendulum.duration(minutes=20),
    multiple_outputs=False,
    pool="api_crm",
    trigger_rule="all_success",
    outlets=[],                        # Assets que produce
    doc_md="Extrae ventas del ERP.",
)
def extraer(): ...
```

### `multiple_outputs`: un dict, varios XComs

```python
@task(multiple_outputs=True)
def calcular_metricas() -> dict[str, float]:
    return {"mae": 10.2, "rmse": 14.8}

m = calcular_metricas()
evaluar(m["mae"])          # cada clave es un XCom independiente
publicar(m["rmse"])
```

Si la función tiene anotación de retorno `dict[...]`, `multiple_outputs` se infiere como `True`.

### Reutilizar una tarea varias veces

```python
@task
def descargar(fuente: str) -> str:
    return f"s3://raw/{fuente}/"

a = descargar.override(task_id="descargar_crm")("crm")
b = descargar.override(task_id="descargar_erp", retries=5)("erp")
```

`.override()` cambia cualquier parámetro de `@task` para esa instancia concreta. Para N instancias definidas en tiempo de ejecución, usa `.expand()` (ver [[12 - Dynamic Task Mapping]]).

---

## Acceder al contexto de ejecución

Declara como parámetros con valor por defecto `None` las variables de contexto que necesites, y Airflow las **inyecta**:

```python
@task
def procesar(ds=None, logical_date=None, data_interval_start=None, ti=None,
             params=None, dag_run=None):
    print(ds, logical_date, params, dag_run.run_id)
    ti.xcom_push(key="filas", value=1234)

# O pide el contexto completo
from airflow.sdk import get_current_context

@task
def otra():
    ctx = get_current_context()
    print(ctx["ds"], ctx["task_instance"].try_number)

# O con **kwargs
@task
def con_kwargs(**context):
    print(context["data_interval_end"])
```

La lista completa de variables de contexto está en [[08 - Templating con Jinja y Contexto]].

---

## Variantes de `@task`

| Decorador | Dónde corre la función | Cuándo usarlo |
|---|---|---|
| `@task` / `@task.python` | En el proceso del worker | Lo normal |
| `@task.virtualenv` | En un **virtualenv nuevo** creado al ejecutar | Dependencias que chocan con las de Airflow |
| `@task.external_python` | En un **Python ya existente** (venv preinstalado) | Igual que virtualenv, pero sin el costo de crearlo |
| `@task.docker` | En un **contenedor Docker** | Aislamiento total, imagen propia |
| `@task.kubernetes` | En un **pod de Kubernetes** | Recursos dedicados (GPU, mucha RAM) |
| `@task.bash` | La función **devuelve un comando** bash, que se ejecuta | Scripts de shell con lógica Python para construirlos |
| `@task.branch` | Devuelve el/los `task_id` a seguir | Branching (nota 09) |
| `@task.short_circuit` | Devuelve un bool; si es False, salta el downstream | Condiciones (nota 09) |
| `@task.sensor` | Devuelve `PokeReturnValue` | Sensores propios (nota 14) |
| `@task.sql` (common.sql) | La función devuelve un SQL a ejecutar | SQL dinámico |
| `@task.pyspark` | Recibe una SparkSession | Spark |

### `@task.virtualenv`

```python
@task.virtualenv(
    requirements=["scikit-learn==1.5.2", "pandas==2.2.3"],
    python_version="3.11",
    system_site_packages=False,
    venv_cache_path="/tmp/venvs",     # reutiliza el venv entre ejecuciones
)
def entrenar(ruta_datos: str) -> float:
    # ⚠️ imports DENTRO de la función: corre en otro intérprete
    import pandas as pd
    from sklearn.linear_model import LogisticRegression
    df = pd.read_parquet(ruta_datos)
    ...
    return 0.87
```

> La función se **serializa y ejecuta en otro proceso**: no puede usar variables globales del archivo del DAG y todos los imports van dentro. Los argumentos y el retorno deben ser serializables.

### @task.docker y @task.kubernetes

```python
@task.docker(image="python:3.12-slim", auto_remove="success", mount_tmp_dir=False)
def en_contenedor(x: int) -> int:
    return x * 2

@task.kubernetes(
    image="miregistro/ml:1.4",
    namespace="airflow",
    container_resources={"limits": {"memory": "8Gi", "nvidia.com/gpu": "1"}},
    get_logs=True,
)
def entrenar_gpu(ruta: str) -> str:
    import torch
    ...
    return "s3://modelos/churn/v12"
```

Requieren los providers `docker` y `cncf.kubernetes`. Ver [[15 - Airflow con dbt, Docker, Kubernetes y MLflow]].

### `@task.bash`

```python
@task.bash
def exportar(tabla: str, ds=None) -> str:
    return f"duckdb analitica.duckdb -c \"COPY {tabla} TO '/exports/{tabla}_{ds}.parquet'\""

exportar("ventas")
```

`@task.bash` combina la flexibilidad de Python (construir el comando con lógica) con la ejecución en shell. Si el comando sale con código distinto de 0, la tarea falla; con el código 99, la tarea se marca como **skipped** (`skip_on_exit_code`).

---

## Mezclar TaskFlow con operadores clásicos

```python
from airflow.providers.standard.operators.bash import BashOperator
from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator

@dag(...)
def mixto():

    crear_tabla = SQLExecuteQueryOperator(
        task_id="crear_tabla",
        conn_id="postgres_dwh",
        sql="CREATE TABLE IF NOT EXISTS resumen (fecha date, total numeric)",
    )

    @task
    def calcular() -> float:
        return 1234.5

    total = calcular()

    insertar = SQLExecuteQueryOperator(
        task_id="insertar",
        conn_id="postgres_dwh",
        sql="INSERT INTO resumen VALUES ('{{ ds }}', {{ ti.xcom_pull(task_ids='calcular') }})",
    )

    # Dependencias: los XComArg también aceptan >>
    crear_tabla >> total >> insertar

    # Pasar la salida de un operador clásico a una tarea TaskFlow
    listar = BashOperator(task_id="listar", bash_command="ls /data | wc -l")   # push del último stdout

    @task
    def mostrar(n: str):
        print(f"Archivos: {n}")

    mostrar(listar.output)         # .output es el XComArg del operador clásico

mixto()
```

## TaskGroups con decorador

```python
from airflow.sdk import task_group

@task_group(group_id="limpieza")
def limpieza(ruta: str):
    @task
    def quitar_nulos(r): return r
    @task
    def dedupe(r): return r
    return dedupe(quitar_nulos(ruta))

@dag(...)
def pipeline():
    ruta = extraer()
    limpia = limpieza(ruta)
    cargar(limpia)
```

Más sobre TaskGroups en [[06 - Operadores Clásicos y Dependencias#TaskGroups]].

## TaskFlow vs. operadores: criterio

| Usa `@task` cuando | Usa un operador cuando |
|---|---|
| La lógica es Python propio | Ya existe un operador del provider (S3, Snowflake, dbt, K8s...) |
| Quieres pasar datos pequeños entre pasos sin boilerplate | Necesitas features del operador (deferrable, templates de archivos `.sql`) |
| Prototipos y DAGs de ML | Tareas declarativas: SQL, bash, transferencias |

Lo más común es **mezclar ambos**: operadores para las integraciones y `@task` para la lógica de negocio.

## Ver también

- [[04 - DAGs - Definición y Parámetros]]
- [[06 - Operadores Clásicos y Dependencias]]
- [[07 - XComs y Paso de Datos]]
- [[12 - Dynamic Task Mapping]]
- [[16 - Decorators]] y [[42 - Type Hints y el Módulo Typing]] (en `Python/`)
- [[50-Orquestacion-Prefect-y-Airflow]] (en `Machine Learning/`): TaskFlow acerca Airflow al estilo "pythónico" de Prefect
