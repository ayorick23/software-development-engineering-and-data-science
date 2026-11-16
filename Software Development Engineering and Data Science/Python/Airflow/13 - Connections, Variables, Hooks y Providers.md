---
tags: [airflow, connections, variables, hooks, providers, secrets, cheat-sheet]
---

# 13 — Connections, Variables, Hooks y Providers

> Continúa de [[12 - Dynamic Task Mapping]]. Para hablar con sistemas externos, Airflow ofrece **Connections** (credenciales y endpoints), **Variables** (configuración global), **Hooks** (clientes reutilizables que usan las Connections) y **Providers** (paquetes con todo lo anterior para cada integración). Esta nota cubre los cuatro, más los **secrets backends** para no guardar secretos en la metadata DB.

## Connections

Una **Connection** guarda cómo conectarse a un sistema: tipo, host, puerto, login, password, schema y un JSON `extra` con opciones adicionales. Se identifica por un `conn_id`.

### Formas de definirlas

**1. Variables de entorno** (ideal para contenedores y CI): `AIRFLOW_CONN_<CONN_ID_EN_MAYÚSCULAS>`

```bash
# Formato URI
export AIRFLOW_CONN_POSTGRES_DWH='postgresql://analista:s3cr3t@dwh.empresa.com:5432/analytics'

# Formato JSON (más legible con extras)
export AIRFLOW_CONN_AWS_DEFAULT='{"conn_type": "aws", "extra": {"region_name": "us-east-1"}}'
export AIRFLOW_CONN_API_CRM='{"conn_type": "http", "host": "https://api.crm.com", "password": "token123",
                              "extra": {"timeout": 30}}'
```

Las Connections definidas por variable de entorno **no aparecen en la UI** ni se guardan en la DB.

**2. CLI:** `airflow connections add ...` (ver [[03 - CLI de Airflow#Connections, Variables y Pools]]).

**3. UI:** *Admin → Connections*. Los passwords se guardan **cifrados con la Fernet key**.

**4. Secrets backend** (producción): ver más abajo.

### Usarlas

```python
# En un operador: casi todos aceptan un *_conn_id
SQLExecuteQueryOperator(task_id="q", conn_id="postgres_dwh", sql="SELECT 1")
S3ListOperator(task_id="ls", bucket="raw", aws_conn_id="aws_default")

# En templates
"{{ conn.postgres_dwh.host }}"
"{{ conn.api_crm.password }}"
"{{ conn.aws_default.extra_dejson.region_name }}"

# En código de una tarea (Task SDK)
from airflow.sdk import Connection

@task
def usar():
    c = Connection.get("api_crm")
    print(c.host, c.login, c.extra_dejson.get("timeout"))
    uri = c.get_uri()
```

> **Airflow 3:** las tareas obtienen las Connections **a través de la Task Execution API**, no de la DB. No hagas consultas directas a la metadata DB desde las tareas: en 3.x está bloqueado.

---

## Variables

Configuración **global** clave-valor (string o JSON), visible en *Admin → Variables*.

```python
from airflow.sdk import Variable

@task
def configurar():
    umbral = float(Variable.get("umbral_mae", default="15"))
    cfg = Variable.get("config_ventas", deserialize_json=True)     # dict
    print(cfg["regiones"])
    Variable.set("ultima_ejecucion_ok", "2026-11-14")              # escribir (con moderación)
```

```python
# En templates: se resuelven al ejecutar (no en el parseo)
BashOperator(task_id="t", bash_command="procesar.sh --umbral {{ var.value.umbral_mae }}")
"{{ var.json.config_ventas.regiones }}"
```

```bash
export AIRFLOW_VAR_UMBRAL_MAE=12.5                                  # por entorno
export AIRFLOW_VAR_CONFIG_VENTAS='{"regiones": ["LATAM", "EMEA"]}'
```

> ⚠️ **No leas Variables en el código top-level del DAG** (fuera de las tareas). Se ejecutaría en **cada parseo** del archivo (cada pocos segundos por cada DAG) y saturaría la DB o el secrets backend. Léelas dentro de las tareas o usa templates `{{ var.value.x }}`.

```python
# ❌ MAL
UMBRAL = Variable.get("umbral_mae")          # top-level

# ✅ BIEN
@task
def evaluar():
    umbral = Variable.get("umbral_mae")
```

### Connections vs. Variables vs. Params vs. config en el repo

| Qué | Dónde |
|---|---|
| Credenciales, hosts, tokens | **Connections** (con secrets backend) |
| Config que cambia sin desplegar código (umbrales, feature flags) | **Variables** |
| Entradas por ejecución (reprocesar región X) | **Params** (ver [[04 - DAGs - Definición y Parámetros#params: parámetros de ejecución con validación]]) |
| Config estable y versionable | Un **YAML en el repo** junto al DAG |

### Enmascaramiento de secretos

Airflow **oculta en los logs y en la UI** los valores de passwords de Connections y de Variables cuyo nombre contiene `password`, `secret`, `token`, `api_key`, `private_key`, etc. Para enmascarar otros valores:

```python
from airflow.sdk.log import mask_secret      # en 2.x: airflow.utils.log.secrets_masker
mask_secret(mi_token_dinamico)
```

---

## Secrets backends

En producción, los secretos deben vivir en un gestor dedicado, no en la metadata DB:

```bash
# AWS Secrets Manager
export AIRFLOW__SECRETS__BACKEND=airflow.providers.amazon.aws.secrets.secrets_manager.SecretsManagerBackend
export AIRFLOW__SECRETS__BACKEND_KWARGS='{"connections_prefix": "airflow/connections",
                                         "variables_prefix": "airflow/variables"}'
# El secreto "airflow/connections/postgres_dwh" se resuelve como conn_id "postgres_dwh"
```

| Backend | Clase |
|---|---|
| AWS Secrets Manager | `airflow.providers.amazon.aws.secrets.secrets_manager.SecretsManagerBackend` |
| AWS SSM Parameter Store | `airflow.providers.amazon.aws.secrets.systems_manager.SystemsManagerParameterStoreBackend` |
| GCP Secret Manager | `airflow.providers.google.cloud.secrets.secret_manager.CloudSecretManagerBackend` |
| Azure Key Vault | `airflow.providers.microsoft.azure.secrets.key_vault.AzureKeyVaultBackend` |
| HashiCorp Vault | `airflow.providers.hashicorp.secrets.vault.VaultBackend` |

**Orden de búsqueda:** secrets backend → variables de entorno → metadata DB. El primero que encuentra el `conn_id` o la variable gana.

Ver [[62 - Seguridad en Python]] (en `Python/`), [[24-Configuracion-Profesional-de-Proyectos]] (en `Machine Learning/`) y [[Kubernetes/06 - ConfigMaps y Secrets|Kubernetes/06]].

---

## Hooks

Un **Hook** es un cliente de alto nivel para un sistema externo que **lee la Connection** por su `conn_id` y expone métodos útiles. Los operadores usan hooks por dentro, y tú puedes usarlos en tus tareas.

### SQL (PostgresHook, MySqlHook, SnowflakeHook... comparten la interfaz `DbApiHook`)

```python
from airflow.providers.postgres.hooks.postgres import PostgresHook

@task
def leer_y_escribir(ds=None):
    hook = PostgresHook(postgres_conn_id="postgres_dwh")

    filas = hook.get_records("SELECT id, total FROM ventas WHERE fecha = %s", parameters=[ds])
    primera = hook.get_first("SELECT COUNT(*) FROM ventas")
    df = hook.get_df("SELECT * FROM ventas WHERE fecha = %s", parameters=[ds])   # Pandas (df_type="polars" para Polars)

    hook.run("DELETE FROM resumen WHERE fecha = %s", parameters=[ds])
    hook.insert_rows(table="resumen", rows=[(ds, 1234.5)], target_fields=["fecha", "total"])

    engine = hook.get_sqlalchemy_engine()      # para pandas.to_sql, etc.
    with hook.get_conn() as conn:              # conexión DB-API cruda (psycopg2)
        ...
```

### S3

```python
from airflow.providers.amazon.aws.hooks.s3 import S3Hook

@task
def mover(ds=None):
    s3 = S3Hook(aws_conn_id="aws_default")
    keys = s3.list_keys(bucket_name="raw", prefix=f"ventas/{ds}/")
    contenido = s3.read_key(key=keys[0], bucket_name="raw")
    s3.load_string("ok", key=f"control/{ds}/_SUCCESS", bucket_name="lake", replace=True)
    s3.load_file("/tmp/reporte.pdf", key=f"reportes/{ds}.pdf", bucket_name="lake", replace=True)
    existe = s3.check_for_key(f"ventas/{ds}/data.parquet", bucket_name="raw")
    cliente_boto3 = s3.get_conn()              # cliente boto3 para todo lo demás
```

### HTTP

```python
from airflow.providers.http.hooks.http import HttpHook

@task
def llamar_api():
    http = HttpHook(method="GET", http_conn_id="api_crm")
    resp = http.run(endpoint="/v1/clientes", data={"activo": "true"},
                    headers={"Authorization": "Bearer ..."})
    return resp.json()["total"]
```

### Hook propio

```python
# include/hooks/crm_hook.py
from airflow.sdk import BaseHook
import requests

class CrmHook(BaseHook):
    conn_name_attr = "crm_conn_id"
    default_conn_name = "crm_default"
    conn_type = "http"
    hook_name = "CRM Interno"

    def __init__(self, crm_conn_id: str = default_conn_name):
        super().__init__()
        self.crm_conn_id = crm_conn_id

    def get_conn(self) -> requests.Session:
        conn = self.get_connection(self.crm_conn_id)
        s = requests.Session()
        s.headers["Authorization"] = f"Bearer {conn.password}"
        s.base_url = conn.host
        return s

    def clientes_activos(self) -> list[dict]:
        s = self.get_conn()
        return s.get(f"{s.base_url}/clientes?activo=true", timeout=30).json()
```

Patrón recomendado: el **Hook** encapsula la conexión y el **Operator** (o `@task`) encapsula la acción. Ver [[06 - Operadores Clásicos y Dependencias#Escribir un operador propio]].

---

## Providers

Un **provider** es un paquete pip independiente con operadores, hooks, sensores, triggers, tipos de conexión y a veces secrets backends o executors. Sus versiones son independientes de la de Airflow.

```bash
pip install apache-airflow-providers-amazon apache-airflow-providers-postgres --constraint "$CONSTRAINT_URL"
airflow providers list
```

| Provider | Contenido destacado |
|---|---|
| `standard` (preinstalado) | Bash, Python, Empty, sensores de fecha, archivo y tareas externas |
| `common-sql` | `SQLExecuteQueryOperator`, `SQLCheckOperator`, `SQLValueCheckOperator`, `SQLColumnCheckOperator`, `@task.sql` |
| `common-io` | `ObjectStoragePath`, XCom backend en object storage |
| `amazon` | S3, Redshift, Athena, Glue, EMR, SageMaker, Lambda, SQS, Secrets Manager |
| `google` | GCS, BigQuery, Dataproc, Vertex AI, Cloud Run |
| `microsoft-azure` | Blob/ADLS, Data Factory, Synapse, Azure ML, Key Vault |
| `postgres`, `mysql`, `snowflake`, `databricks` | Hooks y operadores de cada plataforma |
| `cncf-kubernetes` | `KubernetesPodOperator`, `@task.kubernetes`, KubernetesExecutor |
| `docker` | `DockerOperator`, `@task.docker` |
| `dbt-cloud` | Disparar y monitorear jobs de dbt Cloud |
| `slack`, `smtp` | Notificaciones (notifiers) |
| `http` | `HttpOperator`, `HttpSensor`, `HttpHook` |
| `openlineage` | Emitir linaje de datos (OpenLineage/Marquez) |

El **Astronomer Registry** (registry.astronomer.io) es el catálogo navegable de providers, módulos y ejemplos.

### Checks de calidad de datos con `common-sql`

```python
from airflow.providers.common.sql.operators.sql import SQLColumnCheckOperator, SQLTableCheckOperator

checks = SQLColumnCheckOperator(
    task_id="checks_columnas",
    conn_id="postgres_dwh",
    table="silver.ventas",
    partition_clause="fecha = '{{ ds }}'",
    column_mapping={
        "id":    {"null_check": {"equal_to": 0}, "unique_check": {"equal_to": 0}},
        "monto": {"min": {"geq_to": 0}},
    },
)

tabla = SQLTableCheckOperator(
    task_id="checks_tabla",
    conn_id="postgres_dwh",
    table="silver.ventas",
    checks={"filas_minimas": {"check_statement": "COUNT(*) > 1000"}},
)
```

Para validaciones más ricas, usa Pandera o Great Expectations dentro de una tarea (ver [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]]) o tests de dbt (ver [[11 - Tests de Datos y Unit Tests|dbt/11]]).

## Ver también

- [[06 - Operadores Clásicos y Dependencias]]
- [[14 - Sensores, Deferrable Operators y el Triggerer]]
- [[15 - Airflow con dbt, Docker, Kubernetes y MLflow]]
- [[24 - Databases]] y [[25 - APIs]] (en `Python/`)
- [[DuckDB/06 - Fuentes Remotas y Cloud|DuckDB/06]]: credenciales S3 con `credential_chain`, el mismo principio de "sin secretos en el código"
