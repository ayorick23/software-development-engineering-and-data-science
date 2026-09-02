---
tags: [mlflow, mlops, serving, despliegue, cheat-sheet]
---

# 09 — Model Serving y Despliegue

> Continúa de [[07 - Model Registry]].

Una vez que un modelo está registrado, MLflow ofrece varias formas de "servirlo" — desde pruebas locales rápidas hasta despliegues en la nube.

## `mlflow models serve` — servidor REST local

```bash
mlflow models serve \
  -m "models:/claro-rd-demand-model@champion" \
  --port 5001 \
  --env-manager conda \
  --host 0.0.0.0
```

Esto levanta un servidor REST (por defecto usando MLServer o Flask internamente, según versión) con endpoints listos para inferencia.

### Hacer una petición de scoring

Formato `dataframe_split` (orientado a columnas, eficiente):

```bash
curl -X POST http://localhost:5001/invocations \
  -H "Content-Type: application/json" \
  -d '{
    "dataframe_split": {
      "columns": ["dias_atras", "region"],
      "data": [[90, "santo-domingo"], [60, "santiago"]]
    }
  }'
```

Formato `dataframe_records` (orientado a filas, más legible):

```bash
curl -X POST http://localhost:5001/invocations \
  -H "Content-Type: application/json" \
  -d '{
    "dataframe_records": [
      {"dias_atras": 90, "region": "santo-domingo"},
      {"dias_atras": 60, "region": "santiago"}
    ]
  }'
```

Formato `instances` (compatible con TensorFlow Serving) e `inputs` (para tensores):

```bash
curl -X POST http://localhost:5001/invocations \
  -H "Content-Type: application/json" \
  -d '{"instances": [[5.1, 3.5, 1.4, 0.2]]}'
```

### Endpoints adicionales

```bash
GET  /health        # health check
GET  /version        # versión de MLflow del servidor
GET  /ping           # liveness check
```

## Batch scoring — sin servidor persistente

Para scoring masivo offline (no requiere API REST corriendo):

```bash
mlflow models predict \
  -m "models:/claro-rd-demand-model@champion" \
  -i input.csv \
  -o predictions.csv \
  --env-manager conda
```

```python
import mlflow

model = mlflow.pyfunc.load_model("models:/claro-rd-demand-model@champion")
predictions = model.predict(df_batch)   # df_batch: pandas.DataFrame
```

## Construir una imagen Docker con el modelo embebido

```bash
mlflow models build-docker \
  -m "models:/claro-rd-demand-model@champion" \
  -n mi-registro.azurecr.io/claro-rd-model:v3 \
  --env-manager conda
```

```bash
docker run -p 5001:8080 mi-registro.azurecr.io/claro-rd-model:v3
```

Esto genera una imagen autocontenida (modelo + entorno + servidor REST) lista para desplegar en cualquier orquestador de contenedores — ver `Docker/Orchestration and Scalability.md`.

## Generar solo el Dockerfile (para personalizar)

```bash
mlflow models generate-dockerfile \
  -m "models:/claro-rd-demand-model@champion" \
  -d ./docker_output \
  --env-manager conda
```

Útil cuando necesitas agregar pasos custom al Dockerfile (certificados, agentes de monitoreo, hardening de seguridad) antes de construir la imagen final.

## Despliegue en plataformas cloud nativas

### Databricks Model Serving

```python
from mlflow.deployments import get_deploy_client

client = get_deploy_client("databricks")
client.create_endpoint(
    name="claro-rd-demand-endpoint",
    config={
        "served_entities": [{
            "name": "claro-rd-demand-model",
            "entity_name": "claro-rd-demand-model",
            "entity_version": "3",
            "workload_size": "Small",
            "scale_to_zero_enabled": True,
        }]
    },
)
```

### AWS SageMaker

```python
import mlflow.sagemaker as mfs

mfs.deploy(
    app_name="claro-rd-demand-endpoint",
    model_uri="models:/claro-rd-demand-model@champion",
    region_name="us-east-1",
    mode="create",
    instance_type="ml.m5.large",
    instance_count=2,
)
```

### Azure ML

```python
from mlflow.deployments import get_deploy_client

client = get_deploy_client("azureml")
client.create_deployment(
    name="claro-rd-demand-deployment",
    model_uri="models:/claro-rd-demand-model@champion",
    config={"instance_type": "Standard_DS3_v2", "instance_count": 2},
)
```

## Servir como Spark UDF — inferencia distribuida por lotes

Cuando el scoring debe correr sobre datasets masivos en Spark:

```python
import mlflow.pyfunc

predict_udf = mlflow.pyfunc.spark_udf(
    spark,
    model_uri="models:/claro-rd-demand-model@champion",
    result_type="double",
)

df_scored = df.withColumn("prediccion", predict_udf("dias_atras", "region"))
```

## Comparativa: cuándo usar cada mecanismo

| Escenario | Mecanismo recomendado |
|---|---|
| Prueba rápida local / demo | `mlflow models serve` |
| Scoring batch programado (cron, pipeline) | `mlflow models predict` o carga directa con `pyfunc.load_model` |
| Scoring batch sobre datasets masivos | `mlflow.pyfunc.spark_udf` |
| API de baja/media escala, control total de infraestructura | `mlflow models build-docker` + contenedor propio detrás de un balanceador |
| Alta escala, autoscaling, sin operar infraestructura | Databricks Model Serving / SageMaker / Azure ML endpoints |

## Limitación importante en producción de alto volumen

`mlflow models serve` usa por defecto un servidor de desarrollo (o MLServer en versiones recientes, más apto para producción, pero aún así diseñado para servir un modelo, no para las garantías de un API Gateway completo). Para tráfico productivo de alto volumen, el patrón estándar es: **imagen Docker generada por MLflow → desplegada detrás de un balanceador de carga con autoscaling** (Kubernetes, ECS, Azure Container Apps), en vez de exponer `mlflow models serve` directamente a internet.

## Ver también

- [[06 - Model Format y Flavors]]
- [[07 - Model Registry]]
- [[14 - Integraciones con el Ecosistema]]
- `Machine Learning/49-APIs-con-FastAPI-para-Servir-Modelos.md`
- `Machine Learning/17-Arquitecturas-de-Despliegue-de-Modelos.md`
