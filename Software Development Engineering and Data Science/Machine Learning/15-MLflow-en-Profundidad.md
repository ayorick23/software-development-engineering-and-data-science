---
tags: [mlflow, mlops, experiment-tracking, model-registry]
---

# 15 — MLflow en Profundidad

> Nota del mentor: en [[07-Librerias-de-Data-Science-y-ML]] y [[09-MLOps-en-Profundidad]] ya viste qué *es* MLflow a alto nivel. Aquí vamos a lo operativo: cómo usarlo bien en un proyecto real como el de Claro RD, y sobre todo, hasta dónde llega y dónde no.

## 1. Los cuatro componentes de MLflow

MLflow no es una sola cosa — es una suite con cuatro piezas independientes que puedes usar por separado:

1. **MLflow Tracking**: registra parámetros, métricas, artefactos y código de cada experimento.
2. **MLflow Projects**: empaqueta código de ML en un formato reproducible (similar en espíritu a `pyproject.toml` pero enfocado en reproducibilidad de ejecución, no solo dependencias).
3. **MLflow Models**: formato estándar para empaquetar modelos de forma que sean servibles desde múltiples plataformas (REST API, batch, Spark).
4. **MLflow Model Registry**: control de versiones y ciclo de vida de modelos (`Staging`, `Production`, `Archived`).

En un proyecto como el tuyo, probablemente ya usas o deberías usar los cuatro, aunque el Tracking y el Registry son los que dan más valor inmediato.

## 2. Tracking — la memoria de tus experimentos

```python
import mlflow

mlflow.set_tracking_uri("http://mlflow-server:5000")
mlflow.set_experiment("claro-rd-demand-forecast")

with mlflow.start_run(run_name="xgboost-90d-window"):
    mlflow.log_param("model_type", "XGBoost")
    mlflow.log_param("dias_atras", 90)
    mlflow.log_param("n_estimators", 300)

    model.fit(X_train, y_train)

    mae = mean_absolute_error(y_val, model.predict(X_val))
    rmse = mean_squared_error(y_val, model.predict(X_val), squared=False)
    mlflow.log_metric("mae", mae)
    mlflow.log_metric("rmse", rmse)

    mlflow.xgboost.log_model(model, "model")
    mlflow.log_artifact("feature_importance.png")
```

Esto reemplaza lo que probablemente hacías antes con archivos de Excel o notas sueltas comparando ventanas de -30, -60, -90 y -120 días: cada corrida queda registrada con sus parámetros exactos, sus métricas y el modelo resultante, todo consultable después desde la UI de MLflow sin tener que reconstruir nada.

## 3. Model Registry — el ciclo de vida formal de un modelo

```python
result = mlflow.register_model(
    model_uri="runs:/<run_id>/model",
    name="claro-rd-demand-model"
)

client = mlflow.tracking.MlflowClient()
client.transition_model_version_stage(
    name="claro-rd-demand-model",
    version=result.version,
    stage="Production",
    archive_existing_versions=True
)
```

Esto es exactamente la formalización de tu sistema champion/challenger: en vez de manejar `.bak` files a mano, el Registry te da un historial completo de qué versión estuvo en producción, cuándo, y con qué métricas — con la capacidad de hacer rollback a una versión anterior con una sola llamada si el modelo nuevo falla en producción.

## 4. Dónde vive el "servidor" de MLflow

Tres modos de despliegue, de menor a mayor madurez:

- **Local (`file:./mlruns`)**: para experimentación individual, no compartido con el equipo. Es donde probablemente empezaste.
- **Servidor centralizado con base de datos** (Postgres/MySQL como *backend store* + almacenamiento de artefactos en Azure Blob/S3): lo recomendado para un equipo — todos ven los mismos experimentos, y los artefactos (modelos, gráficas) no viven en el disco de una sola máquina.
- **Databricks Managed MLflow**: si la empresa ya usa Databricks (visto en [[08-Plataformas-de-Datos-y-ML]]), viene integrado sin que tengas que operar tu propio servidor.

## 5. Qué tan lejos llega MLflow (y dónde no)

Aquí es donde muchos juniors se confunden — MLflow **no** es:

- **Un orquestador de pipelines.** No reemplaza a Airflow o a los *scheduled pipelines* de GitLab CI para decidir *cuándo* correr el entrenamiento — solo registra *qué pasó* cuando corre.
- **Un sistema de feature store.** No gestiona features reutilizables entre modelos (eso es Feast, visto en [[07-Librerias-de-Data-Science-y-ML]]).
- **Un sistema de monitoreo de producción.** MLflow registra métricas de *entrenamiento y evaluación offline*, no drift ni performance en tiempo real sobre tráfico de producción (eso es Evidently, Prometheus/Grafana — ver [[18-Monitoreo-y-Observabilidad-de-Modelos]]).
- **Un servidor de inferencia de alto rendimiento por sí solo.** `mlflow models serve` es útil para pruebas, pero en producción de alto volumen normalmente se combina con algo como un contenedor Docker detrás de un balanceador, no se deja como el servidor final.

## 6. MLflow + tu flujo actual — integración concreta

Tu función `autoaprendizaje.py` con el gate de MAE/RMSE es el candidato perfecto para conectar con el Registry:

```
1. Entrena challenger → mlflow.start_run()
2. Evalúa contra ventana de validación → log de métricas
3. Compara métricas del challenger vs. la versión "Production" actual en el Registry
4. Si supera el gate → transition_model_version_stage(stage="Production")
5. Si no supera el gate → el run queda registrado igual (para auditoría) pero nunca se promueve
```

El beneficio real frente a tu implementación actual con `.bak` y JSON de estado: **trazabilidad completa y consultable por cualquiera del equipo**, sin depender de archivos locales en un servidor específico, y con capacidad de comparar visualmente N corridas históricas desde la UI.

## Ver también

- Cheat-sheet técnico completo de MLflow (sintaxis, API, ejemplos): `MLflow/01 - Introducción y Arquitectura General.md`
- [[07-Librerias-de-Data-Science-y-ML]]
- [[09-MLOps-en-Profundidad]]
- [[14-CICD-para-ML-con-GitLab]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]
