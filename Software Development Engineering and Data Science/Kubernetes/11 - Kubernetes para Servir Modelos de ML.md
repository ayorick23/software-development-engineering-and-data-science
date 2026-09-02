---
tags: [kubernetes, k8s, mlops, machine-learning, cheat-sheet, model-serving]
---

# 11 — Kubernetes para Servir Modelos de ML

Todo lo cubierto hasta ahora (Pods, Deployments, Services, escalado) aplica igual a servir un modelo de ML que a cualquier otra aplicación — un endpoint de inferencia es, ante todo, un servicio HTTP. Este archivo cubre lo específico de ML sobre esa base: el patrón manual con contenedores propios, y las plataformas especializadas que existen para no reinventarlo cada vez.

## El patrón base: FastAPI/MLflow + Docker + Deployment + Service

El caso más simple —y perfectamente válido en producción— es empaquetar el modelo servido con [[FastAPI/16 - Integración con el Ecosistema|FastAPI]] o `mlflow models serve` (ver `MLflow/09 - Model Serving y Despliegue.md`) en una imagen Docker, y desplegarla como cualquier otra API:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: clasificador-churn
spec:
  replicas: 3
  selector:
    matchLabels: { app: clasificador-churn }
  template:
    metadata:
      labels: { app: clasificador-churn }
    spec:
      containers:
        - name: api
          image: mi-registro/clasificador-churn:2.3.0
          ports: [{ containerPort: 8000 }]
          readinessProbe:
            httpGet: { path: /ready, port: 8000 }
          resources:
            requests: { cpu: "500m", memory: "1Gi" }
            limits: { cpu: "2", memory: "2Gi" }
---
apiVersion: v1
kind: Service
metadata:
  name: clasificador-churn-svc
spec:
  selector: { app: clasificador-churn }
  ports: [{ port: 80, targetPort: 8000 }]
```

Este patrón cubre la mayoría de los casos: un modelo de tamaño moderado, tráfico predecible, sin necesidad de GPU compartida entre modelos. Todo lo aprendido en [[03 - Pods]], [[04 - Deployments y ReplicaSets]], [[05 - Services y Networking]] y [[08 - Escalado - HPA, VPA y Cluster Autoscaler]] aplica directamente.

## Cuándo el patrón base no basta

A medida que crece el número de modelos servidos, o los requisitos se vuelven más específicos de ML, aparecen necesidades que el patrón base no resuelve por sí solo:

- Servir **decenas o cientos** de modelos distintos sin escribir un Deployment/Service por cada uno.
- **Scale-to-zero**: apagar completamente los Pods de un modelo poco usado y "despertarlo" solo cuando llega tráfico (el patrón base con HPA nunca escala a 0 réplicas, porque perdería el Service).
- Servir modelos en **GPU compartida** de forma eficiente entre varias cargas.
- **Canary releases y A/B testing** de versiones de modelo con reparto de tráfico por porcentaje, sin implementarlo a mano.
- **Explicabilidad e inferencia por lotes** integradas al mismo endpoint.

Para esto existen plataformas especializadas de *model serving* construidas sobre Kubernetes:

| Plataforma | Qué resuelve |
|---|---|
| **KServe** (antes KFServing) | Estándar de facto para servir modelos en K8s vía un CRD (`InferenceService`); soporta scale-to-zero, canary por tráfico, múltiples frameworks (sklearn, XGBoost, PyTorch, TensorFlow) con contenedores predefinidos |
| **Seldon Core** | Similar a KServe, con foco fuerte en explicabilidad, detección de outliers/drift y pipelines de inferencia complejos (pre/post-procesamiento como pasos separados) |
| **Kubeflow** | Plataforma de ML de punta a punta sobre Kubernetes (pipelines, notebooks, tuning, serving) — KServe es, de hecho, un componente de Kubeflow que también se puede instalar de forma independiente |
| **BentoML + Yatai** | Empaqueta modelos con su lógica de servicio en un formato estándar (`Bento`) y los despliega en Kubernetes con su propio operador |

```yaml
# ejemplo de InferenceService (KServe) — nótese cuánta infraestructura queda oculta
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: clasificador-churn
spec:
  predictor:
    sklearn:
      storageUri: "s3://mi-bucket/modelos/clasificador-churn/v3"
      resources:
        requests: { cpu: "500m", memory: "1Gi" }
```

Con esto, KServe genera automáticamente el Deployment, el Service, el Ingress y la lógica de scale-to-zero por debajo — el mismo objetivo que armar todo manualmente con los archivos [[03 - Pods|03]]–[[08 - Escalado - HPA, VPA y Cluster Autoscaler|08]], pero como una sola declaración de alto nivel.

## Canary deployments para modelos — por qué importa más aquí que en software genérico

En software tradicional, un canary valida que el código no tenga bugs. En ML, un canary valida algo adicional: que las **predicciones** del modelo nuevo sean buenas con tráfico real — una regresión de calidad silenciosa (el modelo "funciona" técnicamente, pero predice peor) no la detecta ningún health check HTTP.

```yaml
# ejemplo conceptual con KServe — reparto de tráfico entre dos revisiones
spec:
  predictor:
    canaryTrafficPercent: 10   # 10% del tráfico va a la nueva revisión; 90% sigue en la estable
```

Este reparto de tráfico, combinado con el monitoreo estadístico de predicciones (ver `Machine Learning/09-MLOps-en-Profundidad.md` sección Evidently), es lo que permite detectar una regresión de calidad **antes** de dirigirle el 100% del tráfico — el mismo principio del rolling update de [[04 - Deployments y ReplicaSets#Rollouts y Rollbacks]], aplicado al eje de "calidad de predicción" en vez de solo "el proceso no crashea".

## Inferencia por lotes vs. en tiempo real

No todo servicio de modelo necesita ser un endpoint HTTP siempre activo. Para inferencia por lotes (ej. recalcular un score para toda la base de clientes una vez al día), el patrón correcto es un **[[04 - Deployments y ReplicaSets#Otros controladores de carga de trabajo (para contraste)|CronJob]]**, no un Deployment con Service — evita mantener réplicas corriendo 24/7 para una carga que solo ocurre una vez al día.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scoring-diario-churn
spec:
  schedule: "0 3 * * *"   # 3am todos los días
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: scoring
              image: mi-registro/scoring-batch:1.0
              command: ["python", "score_batch.py"]
          restartPolicy: OnFailure
```

## Ver también

- [[03 - Pods]]
- [[04 - Deployments y ReplicaSets]]
- [[07 - Volumes y Almacenamiento Persistente]]
- [[08 - Escalado - HPA, VPA y Cluster Autoscaler]]
- `MLflow/09 - Model Serving y Despliegue.md`
- `FastAPI/15 - Despliegue y Producción.md`
- [[17-Arquitecturas-de-Despliegue-de-Modelos]] (en `Machine Learning/`)
