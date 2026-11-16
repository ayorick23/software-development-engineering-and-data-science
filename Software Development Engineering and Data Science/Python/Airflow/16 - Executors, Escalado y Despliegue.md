---
tags: [airflow, executors, celery, kubernetes, helm, escalado, despliegue, cheat-sheet]
---

# 16 — Executors, Escalado y Despliegue

> Continúa de [[15 - Airflow con dbt, Docker, Kubernetes y MLflow]]. El **executor** decide **dónde y cómo** corren las tareas. Esta nota compara los executors, explica los niveles de concurrencia (parallelism, pools, colas, prioridades), cómo escalar cada componente y las opciones de despliegue: el **Helm chart** oficial en Kubernetes y los servicios gestionados (Astro, MWAA, Cloud Composer).

## Executors

| Executor | Dónde corren las tareas | Escala | Ideal para |
|---|---|---|---|
| **LocalExecutor** | Subprocesos en la máquina del scheduler | Vertical (una máquina) | Dev, instalaciones pequeñas o medianas |
| **CeleryExecutor** | Workers de Celery persistentes (cola Redis o RabbitMQ) | Horizontal (N workers) | Mucho volumen de tareas cortas, latencia baja |
| **KubernetesExecutor** | **Un pod por tarea**, creado y destruido | Elástica (según el cluster) | Cargas heterogéneas, aislamiento, escalar a cero |
| **EdgeExecutor** (provider `edge3`) | Workers remotos que se conectan por HTTP al API Server | Distribuida (otras redes o nubes, on-premise) | Tareas que deben correr cerca de los datos o detrás de un firewall |
| **AWS ECS / Batch / Lambda executors** (provider `amazon`) | Contenedores en ECS, Batch o Lambda | Elástica (serverless) | Instalaciones en AWS sin K8s |

> **SequentialExecutor** se eliminó en Airflow 3. `standalone` usa LocalExecutor.

### Configuración

```bash
export AIRFLOW__CORE__EXECUTOR=CeleryExecutor
export AIRFLOW__CELERY__BROKER_URL=redis://redis:6379/0
export AIRFLOW__CELERY__RESULT_BACKEND=db+postgresql://airflow:airflow@postgres/airflow
export AIRFLOW__CELERY__WORKER_CONCURRENCY=16          # tareas simultáneas por worker
```

### Multi-executor: lo mejor de cada uno

Desde 2.10 se pueden configurar **varios executors a la vez** y elegirlo por tarea. Este mecanismo reemplaza al antiguo `CeleryKubernetesExecutor`:

```bash
export AIRFLOW__CORE__EXECUTOR="CeleryExecutor,KubernetesExecutor"   # el primero es el default
```

```python
@task                                           # Celery (default): tareas ligeras y rápidas
def chequeo(): ...

@task(executor="KubernetesExecutor")            # un pod dedicado para lo pesado
def entrenar(): ...

# Con KubernetesExecutor, personalizar el pod de esa tarea:
from kubernetes.client import models as k8s

@task(executor="KubernetesExecutor", executor_config={
    "pod_override": k8s.V1Pod(spec=k8s.V1PodSpec(containers=[
        k8s.V1Container(name="base",
                        resources=k8s.V1ResourceRequirements(limits={"memory": "16Gi", "cpu": "4"}))
    ]))
})
def tarea_pesada(): ...
```

### Celery: colas para workers especializados

```bash
airflow celery worker --queues default          # workers generales
airflow celery worker --queues gpu              # workers en máquinas con GPU
```

```python
@task(queue="gpu")
def inferencia_gpu(): ...
```

---

## Niveles de concurrencia

```mermaid
flowchart TB
    P["core.parallelism (32)<br/>máx. tareas corriendo en TODA la instalación<br/>(por scheduler)"]
    P --> D["max_active_tasks (por DAG, 16)"]
    P --> R["max_active_runs (por DAG, 16)"]
    P --> POOL["Pools<br/>slots compartidos entre DAGs"]
    P --> TI["max_active_tis_per_dag / per_dagrun<br/>(por tarea)"]
    P --> W["worker_concurrency (Celery)<br/>capacidad física de cada worker"]
```

Una tarea corre solo si **todos** los límites que le aplican tienen espacio.

### Pools: proteger recursos compartidos

Un **pool** es un contador de slots compartido por todas las tareas que lo usan, aunque sean de DAGs distintos. Es la herramienta para no saturar una API, una base de datos o un archivo DuckDB.

```bash
airflow pools set postgres_erp 4 "Réplica del ERP: máx. 4 consultas a la vez"
airflow pools set duckdb_writer 1 "Un solo escritor sobre analitica.duckdb"
airflow pools set api_crm 10 "Rate limit del CRM"
```

```python
@task(pool="postgres_erp")
def extraer_erp(): ...

@task(pool="duckdb_writer")
def escribir_duckdb(): ...

@task(pool="cpu_pesado", pool_slots=4)          # esta tarea ocupa 4 slots
def entrenar(): ...
```

`default_pool` (128 slots) es el pool de todas las tareas que no declaran otro.

### Prioridades

Cuando hay más tareas listas que slots, el scheduler ordena por **`priority_weight`** efectivo:

| `weight_rule` | Peso efectivo de una tarea | Efecto |
|---|---|---|
| `downstream` (default) | Su peso + el de todas sus tareas downstream | Prioriza las tareas que desbloquean más trabajo |
| `upstream` | Su peso + el de sus upstream | Prioriza terminar los DAGs ya empezados |
| `absolute` | Solo su propio peso | Control total y explícito |

```python
@dag(default_args={"priority_weight": 10, "weight_rule": "absolute"})
def dag_critico(): ...        # se adelanta a los DAGs de prioridad 1
```

---

## Escalar cada componente

| Componente | ¿Cómo se escala? | Señales de que falta capacidad |
|---|---|---|
| **Scheduler** | Varias réplicas en alta disponibilidad (requiere PostgreSQL o MySQL 8 con row-level locking) | Tareas que tardan en pasar de `scheduled` a `queued` |
| **DAG Processor** | Más réplicas o más `parsing_processes`; separar DAG bundles | DAGs que tardan en reflejar cambios; `dag_processing.total_parse_time` alto |
| **API Server** | Réplicas detrás de un balanceador; `--workers` | UI lenta; workers con timeouts en la Task Execution API |
| **Triggerer** | Réplicas (cada una toma triggers) | Muchos triggers activos; event loop lento |
| **Workers** | Más workers o pods (Celery con KEDA; K8s automático) | Tareas en `queued` durante mucho rato |
| **Metadata DB** | Postgres gestionado de mayor tamaño + **PgBouncer** | Errores de conexión; lentitud general |

> El **cuello de botella más común** no es el número de workers sino el **parseo de DAGs** (código top-level pesado) y la **metadata DB** sin mantenimiento. Ver [[17 - Testing, Observabilidad y Buenas Prácticas#Código top-level]] y `airflow db clean`.

---

## Despliegue en Kubernetes con el Helm chart oficial

```bash
helm repo add apache-airflow https://airflow.apache.org
helm repo update
kubectl create namespace airflow
helm upgrade --install airflow apache-airflow/airflow -n airflow -f values.yaml
kubectl get pods -n airflow
kubectl port-forward svc/airflow-api-server 8080:8080 -n airflow
```

```yaml
# values.yaml (extracto)
executor: "CeleryExecutor,KubernetesExecutor"

images:
  airflow:
    repository: miregistro/airflow-empresa      # imagen propia con providers y dependencias
    tag: "3.1.0-2026.11.15"

# DAGs: una de estas estrategias
dags:
  gitSync:
    enabled: true
    repo: https://github.com/empresa/airflow-dags.git
    branch: main
    subPath: dags
    period: 60s

postgresql:
  enabled: false                                # usa una DB gestionada (RDS / Cloud SQL)
data:
  metadataSecretName: airflow-metadata-db       # Secret con la connection string

workers:
  replicas: 2
  keda:
    enabled: true                               # autoescalado de workers Celery según la cola
    minReplicaCount: 1
    maxReplicaCount: 10

triggerer:
  replicas: 2
scheduler:
  replicas: 2

logs:
  persistence:
    enabled: false                              # mejor remote logging a S3/GCS (nota 17)

config:
  core:
    load_examples: "False"
  logging:
    remote_logging: "True"
    remote_base_log_folder: "s3://empresa-airflow-logs"
    remote_log_conn_id: "aws_default"
```

Fundamentos de Helm y Kubernetes: [[Kubernetes/09 - Helm - Gestión de Paquetes para Kubernetes|Kubernetes/09]] y [[Kubernetes/08 - Escalado - HPA, VPA y Cluster Autoscaler|Kubernetes/08]].

### ¿Cómo llegan los DAGs a producción?

| Estrategia | Cómo | Pros | Contras |
|---|---|---|---|
| **Imagen con DAGs incluidos** | `COPY dags/` en el Dockerfile; un deploy = una imagen nueva | Inmutable, reproducible | Cada cambio implica build y rollout |
| **git-sync** | Sidecar que hace `git pull` periódico | Cambios rápidos sin rebuild | Las dependencias pip siguen requiriendo una imagen nueva |
| **DAG bundles (Airflow 3)** | `GitDagBundle` configurado en Airflow | Versionado nativo: cada run sabe qué commit usó | Más reciente; requiere configurar la connection de Git |
| **Volumen compartido / S3 sync** | NFS, EFS o sincronizar desde S3 | Simple en VMs | Sin atomicidad, condiciones de carrera |

### CI/CD de DAGs

```yaml
# .github/workflows/airflow.yml (esquema)
on: [pull_request, push]
jobs:
  test:
    steps:
      - uses: actions/checkout@v4
      - run: pip install "apache-airflow==3.1.0" -r requirements.txt --constraint "$CONSTRAINT_URL"
      - run: ruff check dags/ include/
      - run: pytest tests/                   # integridad de DAGs + tests unitarios (nota 17)
  deploy:
    if: github.ref == 'refs/heads/main'
    needs: test
    steps:
      - run: docker build -t miregistro/airflow-empresa:${{ github.sha }} .
      - run: docker push miregistro/airflow-empresa:${{ github.sha }}
      - run: helm upgrade airflow apache-airflow/airflow -n airflow --set images.airflow.tag=${{ github.sha }} -f values.yaml
```

Ver [[Kubernetes/13 - Managed Kubernetes, CI-CD y Buenas Prácticas|Kubernetes/13]] y [[Git/Remote|Git]].

---

## Servicios gestionados

| Servicio | Proveedor | Notas |
|---|---|---|
| **Astro** | Astronomer | El más cercano al proyecto open source (sus ingenieros son grandes contribuidores de Airflow); deploy con `astro deploy`; soporte rápido de versiones nuevas |
| **Amazon MWAA** | AWS | Integración IAM, S3 para los DAGs; las versiones nuevas llegan con algo de retraso |
| **Cloud Composer** | Google Cloud | Integración con GKE, BigQuery y Vertex AI |
| **Azure (Data Factory Workflow Orchestration Manager)** | Microsoft | Airflow gestionado dentro de ADF; revisar disponibilidad y versión |

**¿Autogestionado o gestionado?** Un servicio gestionado te ahorra operar la DB, las actualizaciones, la alta disponibilidad y la seguridad, a cambio de costo fijo y menos control. Para equipos pequeños, normalmente compensa. Autogestiona solo si ya tienes un equipo de plataforma en Kubernetes.

## Ver también

- [[02 - Instalación y Entorno Local]]
- [[14 - Sensores, Deferrable Operators y el Triggerer]]
- [[17 - Testing, Observabilidad y Buenas Prácticas]]
- [[Docker/11 - Orquestación y Escalabilidad|Docker/11]]
- [[Kubernetes/01 - Introducción y Arquitectura del Clúster|Kubernetes/01]]
- [[08-Plataformas-de-Datos-y-ML]] (en `Machine Learning/`)
