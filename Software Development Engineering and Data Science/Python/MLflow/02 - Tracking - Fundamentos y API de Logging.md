---
tags: [mlflow, mlops, tracking, cheat-sheet]
---

# 02 — Tracking: Fundamentos y API de Logging

> Continúa de [[01 - Introducción y Arquitectura General]].

**MLflow Tracking** es el componente que más se usa en el día a día. Es una API + UI para registrar y consultar: parámetros, métricas, artefactos, código fuente y metadatos de cada ejecución de entrenamiento.

## Experiments y Runs — la jerarquía

```
Experiment ("claro-rd-demand-forecast")
 ├── Run 1 (run_id: a1b2c3...) — xgboost-90d-window
 ├── Run 2 (run_id: d4e5f6...) — xgboost-60d-window
 └── Run 3 (run_id: g7h8i9...) — lightgbm-90d-window
```

- **Experiment**: agrupación lógica. Se identifica por nombre único o `experiment_id`.
- **Run**: una ejecución concreta, con su propio `run_id` (UUID). Todo lo que se loguea (params, metrics, artifacts) queda asociado a un run.

### Crear/seleccionar un experiment

```python
import mlflow

mlflow.set_tracking_uri("http://localhost:5000")   # opcional; por defecto usa ./mlruns
mlflow.set_experiment("claro-rd-demand-forecast")   # lo crea si no existe
```

También se puede crear explícitamente y guardar metadata adicional:

```python
experiment_id = mlflow.create_experiment(
    name="claro-rd-demand-forecast",
    artifact_location="s3://mi-bucket/mlflow-artifacts",
    tags={"team": "data-science", "project": "demand-forecast"}
)
```

## El contexto `start_run`

Todo logging debe ocurrir dentro de un run activo. La forma idiomática es con el context manager:

```python
with mlflow.start_run(run_name="xgboost-90d-window") as run:
    print(run.info.run_id)      # UUID del run
    print(run.info.status)      # "RUNNING"
    # ... logging aquí ...
# Al salir del bloque `with`, MLflow marca el run como FINISHED automáticamente
# (o FAILED si hubo una excepción sin capturar)
```

Sin `with`, hay que cerrar el run manualmente:

```python
run = mlflow.start_run(run_name="manual-run")
try:
    mlflow.log_param("alpha", 0.5)
finally:
    mlflow.end_run()   # o mlflow.end_run(status="FAILED")
```

### Runs anidados (nested runs)

Útiles para búsquedas de hiperparámetros, donde cada combinación es un "hijo" de un run "padre" que agrupa todo el experimento — ver también [[10 - Hyperparameter Tuning con MLflow]].

```python
with mlflow.start_run(run_name="grid-search-parent") as parent:
    for n_estimators in [100, 200, 300]:
        with mlflow.start_run(run_name=f"n_est_{n_estimators}", nested=True):
            mlflow.log_param("n_estimators", n_estimators)
            # entrenar y loguear métricas...
```

## Logging de parámetros

Los **parámetros** son valores de entrada que no cambian durante el run (hiperparámetros, configuración). Son inmutables una vez logueados — intentar loguear el mismo param con otro valor lanza excepción.

```python
mlflow.log_param("model_type", "XGBoost")
mlflow.log_param("n_estimators", 300)
mlflow.log_param("learning_rate", 0.05)

# Loguear varios a la vez:
mlflow.log_params({
    "model_type": "XGBoost",
    "n_estimators": 300,
    "learning_rate": 0.05,
    "max_depth": 6,
})
```

## Logging de métricas

Las **métricas** son valores numéricos que sí pueden variar — típicamente se loguean varias veces dentro de un mismo run (por ejemplo, una vez por época).

```python
mlflow.log_metric("mae", 12.4)
mlflow.log_metric("rmse", 18.7)

# Con "step" para series temporales (ej. loss por época):
for epoch in range(10):
    train_loss = train_one_epoch(model)
    mlflow.log_metric("train_loss", train_loss, step=epoch)

# Varias a la vez:
mlflow.log_metrics({"mae": 12.4, "rmse": 18.7, "r2": 0.89})
```

Cuando se loguea la misma métrica varias veces con distinto `step`, MLflow guarda la **serie completa** (no solo el último valor) — la UI la grafica como línea de tiempo.

## Logging de artefactos

Un **artefacto** es cualquier archivo: modelos serializados, gráficas, datasets, archivos de configuración, reportes HTML.

```python
# Un solo archivo:
mlflow.log_artifact("feature_importance.png")

# Un directorio completo (recursivo):
mlflow.log_artifacts("output_dir/", artifact_path="reports")

# Guardar un dict/objeto directamente como artefacto (sin escribirlo a disco antes):
mlflow.log_dict({"threshold": 0.5, "classes": ["A", "B"]}, "config.json")

# Guardar una figura de matplotlib directamente:
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
ax.plot(y_true, y_pred, "o")
mlflow.log_figure(fig, "prediction_scatter.png")

# Guardar texto:
mlflow.log_text("Notas del experimento...", "notes.txt")
```

## Tags — metadata libre para búsqueda y organización

A diferencia de los params (inmutables, parte de la config del modelo), los **tags** son metadata mutable, pensada para clasificar y filtrar runs.

```python
mlflow.set_tag("release_candidate", "true")
mlflow.set_tags({"team": "forecasting", "reviewed": "yes"})

# Tags a nivel de sistema (reservados, empiezan con mlflow.):
# mlflow.runName, mlflow.user, mlflow.source.git.commit, mlflow.source.name
```

## Ejemplo completo — un run realista

```python
import mlflow
import mlflow.xgboost
from sklearn.metrics import mean_absolute_error, mean_squared_error

mlflow.set_tracking_uri("http://mlflow-server:5000")
mlflow.set_experiment("claro-rd-demand-forecast")

with mlflow.start_run(run_name="xgboost-90d-window"):
    params = {"model_type": "XGBoost", "dias_atras": 90, "n_estimators": 300}
    mlflow.log_params(params)

    model.fit(X_train, y_train)
    preds = model.predict(X_val)

    mlflow.log_metric("mae", mean_absolute_error(y_val, preds))
    mlflow.log_metric("rmse", mean_squared_error(y_val, preds, squared=False))

    mlflow.xgboost.log_model(model, "model")     # ver 06 - Model Format y Flavors
    mlflow.log_artifact("feature_importance.png")
    mlflow.set_tag("dataset_version", "v3")
```

## Obtener el run activo y consultar datos

```python
run = mlflow.active_run()          # el run actualmente abierto (o None)
info = mlflow.get_run(run_id="a1b2c3...")

print(info.data.params)     # dict de parámetros
print(info.data.metrics)    # dict con el ÚLTIMO valor de cada métrica
print(info.data.tags)       # dict de tags
print(info.info.artifact_uri)  # dónde están los artefactos de este run
```

## `mlflow.log_input` — trazabilidad de datasets

Desde MLflow 2.x se pueden loguear datasets como parte del linaje del run, útil para auditoría de qué datos entrenaron qué modelo:

```python
dataset = mlflow.data.from_pandas(df_train, source="s3://bucket/train.parquet", name="train_set")
mlflow.log_input(dataset, context="training")
```

## Ver también

- [[03 - Tracking - Servidor, Backend Store y Artifact Store]]
- [[04 - Tracking - Búsqueda, Comparación y Organización]]
- [[05 - Autologging en Profundidad]]
- [[10 - Hyperparameter Tuning con MLflow]]
