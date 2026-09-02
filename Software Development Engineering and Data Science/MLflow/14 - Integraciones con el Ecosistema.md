---
tags: [mlflow, mlops, integraciones, cicd, docker, kubernetes, cheat-sheet]
---

# 14 — Integraciones con el Ecosistema

> Consolida ideas de [[03 - Tracking - Servidor, Backend Store y Artifact Store]], [[08 - Projects - Empaquetado Reproducible]] y [[09 - Model Serving y Despliegue]].

MLflow rara vez vive solo — normalmente es una pieza dentro de una cadena de herramientas más amplia. Este archivo cubre los puntos de integración más comunes.

## Docker

Dos usos distintos, no confundir:

1. **MLflow corriendo dentro de Docker** (el servidor de Tracking en un contenedor) — ver [[03 - Tracking - Servidor, Backend Store y Artifact Store]] para el `docker-compose.yml` completo.
2. **Un modelo empaquetado por MLflow como imagen Docker** (`mlflow models build-docker`) — ver [[09 - Model Serving y Despliegue]].

```bash
# Reconstruir la imagen de un modelo cada vez que se promueve una nueva versión al Registry:
mlflow models build-docker -m "models:/mi-modelo@champion" -n registro/mi-modelo:$(date +%Y%m%d)
docker push registro/mi-modelo:$(date +%Y%m%d)
```

Ver `Docker/Introduction to Docker.md` y `Docker/Docker Compose.md` para fundamentos de Docker en sí.

## Kubernetes

El patrón estándar es desplegar la imagen generada por `mlflow models build-docker` como un `Deployment` + `Service` de Kubernetes, con autoscaling horizontal según carga de inferencia:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: claro-rd-model-serving
spec:
  replicas: 2
  selector:
    matchLabels: {app: claro-rd-model}
  template:
    metadata:
      labels: {app: claro-rd-model}
    spec:
      containers:
        - name: model-server
          image: registro/claro-rd-model:v3
          ports: [{containerPort: 8080}]
          resources:
            requests: {cpu: "500m", memory: "1Gi"}
            limits: {cpu: "1", memory: "2Gi"}
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: claro-rd-model-hpa
spec:
  scaleTargetRef: {apiVersion: apps/v1, kind: Deployment, name: claro-rd-model-serving}
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource: {name: cpu, target: {type: Utilization, averageUtilization: 70}}
```

Ver `Docker/Orchestration and Scalability.md`.

## Orquestadores de pipelines: Airflow y Prefect

MLflow no orquesta *cuándo* correr un pipeline — eso lo hace Airflow o Prefect, que a su vez invocan pasos que internamente usan la API de MLflow.

### Airflow

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
import mlflow

def entrenar_y_evaluar(**context):
    mlflow.set_tracking_uri("http://mlflow-server:5000")
    mlflow.set_experiment("claro-rd-demand-forecast")
    with mlflow.start_run():
        # entrenar, evaluar, registrar el modelo si supera el gate
        ...

with DAG("reentrenamiento_semanal", schedule="@weekly") as dag:
    entrenar_task = PythonOperator(task_id="entrenar", python_callable=entrenar_y_evaluar)
```

### Prefect

```python
from prefect import flow, task
import mlflow

@task
def entrenar_modelo():
    with mlflow.start_run():
        ...
        return run_id

@flow(name="reentrenamiento-semanal")
def pipeline_reentrenamiento():
    run_id = entrenar_modelo()
```

Ver `Machine Learning/50-Orquestacion-Prefect-y-Airflow.md`.

## CI/CD — GitLab CI / GitHub Actions

Patrón para automatizar el gate de promoción (entrenar → evaluar → promover solo si supera al champion):

```yaml
# .gitlab-ci.yml
entrenar_y_validar:
  stage: train
  script:
    - python train.py
    - python validar_gate.py --model-uri "$CI_COMMIT_SHA" --registry-model claro-rd-demand-model
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
```

```python
# validar_gate.py — se apoya en mlflow.evaluate, ver 11 - Evaluación de Modelos
if evaluar_y_promover(challenger_uri, champion_uri, df_val):
    print("Modelo promovido a champion")
else:
    sys.exit(1)   # falla el pipeline si el modelo no supera el gate
```

Ver `Machine Learning/14-CICD-para-ML-con-GitLab.md` y `Machine Learning/48-CICD-con-GitHub-Actions.md`.

## FastAPI — servir un modelo con más control que `mlflow models serve`

Cuando se necesita lógica de negocio adicional alrededor de la inferencia (autenticación custom, logging estructurado, rate limiting específico):

```python
from fastapi import FastAPI
import mlflow.pyfunc

app = FastAPI()
model = mlflow.pyfunc.load_model("models:/claro-rd-demand-model@champion")

@app.post("/predecir")
def predecir(payload: dict):
    df = pd.DataFrame([payload])
    prediccion = model.predict(df)
    return {"prediccion": float(prediccion[0])}
```

Ver `Machine Learning/49-APIs-con-FastAPI-para-Servir-Modelos.md`.

## Spark

Además de `mlflow.spark.log_model` y `mlflow.pyfunc.spark_udf` (ver [[09 - Model Serving y Despliegue]]), MLflow puede correr como Tracking server compartido entre notebooks de Databricks/Spark y jobs batch, unificando el registro de experimentos sin importar dónde corre el código.

## Feature Store (Feast)

MLflow y un feature store resuelven problemas distintos y se complementan: Feast gestiona *qué features existen y cómo se sirven de forma consistente entre entrenamiento e inferencia*; MLflow registra *qué modelo se entrenó con cuáles de esas features y con qué resultado*. Es común loguear como parámetro/tag qué versión del feature set se usó:

```python
mlflow.set_tag("feature_view_version", "demanda_features_v4")
```

Ver `Machine Learning/43-Feature-Store-en-Profundidad.md`.

## Monitoreo de producción (Evidently, Prometheus/Grafana)

MLflow **no** monitorea drift ni performance en producción — solo métricas de entrenamiento/evaluación offline. La integración típica es: el sistema de monitoreo detecta degradación → dispara un pipeline de reentrenamiento → ese pipeline usa MLflow para registrar el nuevo modelo y decidir si lo promueve. Son sistemas complementarios, no superpuestos.

Ver `Machine Learning/18-Monitoreo-y-Observabilidad-de-Modelos.md`.

## DVC (versionado de datos)

MLflow versiona modelos y experimentos; DVC versiona los datasets de entrenamiento. Se combinan logueando el hash/versión de DVC como parámetro del run, de forma que un run de MLflow sea trazable hasta el dataset exacto usado:

```python
import subprocess
dvc_hash = subprocess.check_output(["dvc", "get-url", "data/train.csv"]).decode().strip()
mlflow.log_param("dvc_data_version", dvc_hash)
```

Ver `Machine Learning/46-Reproducibilidad-con-DVC.md`.

## Ver también

- [[03 - Tracking - Servidor, Backend Store y Artifact Store]]
- [[09 - Model Serving y Despliegue]]
- [[15 - Buenas Prácticas, Seguridad y Comparativa]]
