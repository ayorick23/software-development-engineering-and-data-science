---
tags: [mlflow, mlops, tracking, cheat-sheet]
---

# 04 — Tracking: Búsqueda, Comparación y Organización

> Continúa de [[03 - Tracking - Servidor, Backend Store y Artifact Store]].

Registrar experimentos sirve de poco si después no se pueden **encontrar, filtrar y comparar**. Este archivo cubre la API de búsqueda y la UI.

## `mlflow.search_runs` — la herramienta de consulta principal

Devuelve un `pandas.DataFrame` con una fila por run — ideal para análisis posterior en un notebook.

```python
import mlflow

runs_df = mlflow.search_runs(experiment_names=["claro-rd-demand-forecast"])
print(runs_df[["run_id", "metrics.rmse", "params.n_estimators"]])
```

### El filtro `filter_string` — sintaxis tipo SQL simplificado

```python
runs_df = mlflow.search_runs(
    experiment_names=["claro-rd-demand-forecast"],
    filter_string="metrics.rmse < 20 AND params.model_type = 'XGBoost'",
    order_by=["metrics.rmse ASC"],
    max_results=10,
)
```

Reglas de sintaxis del `filter_string`:

- Prefijos obligatorios: `metrics.`, `params.`, `tags.`, `attributes.` (para `status`, `run_id`, `start_time`, etc.)
- Operadores: `=`, `!=`, `<`, `<=`, `>`, `>=`, `LIKE`, `ILIKE`, `IN`
- Combinación solo con `AND` (no soporta `OR` directamente en versiones estándar)
- Nombres con espacios o caracteres especiales van entre backticks: `` params.`learning rate` ``

```python
# Buscar solo runs finalizados con éxito
mlflow.search_runs(filter_string="attributes.status = 'FINISHED'")

# Buscar por tag
mlflow.search_runs(filter_string="tags.release_candidate = 'true'")

# Buscar por rango de fechas (timestamp en milisegundos)
mlflow.search_runs(filter_string="attributes.start_time > 1700000000000")
```

### Buscar en múltiples experiments a la vez

```python
mlflow.search_runs(experiment_names=["proyecto-a", "proyecto-b"])

# o por ID:
mlflow.search_runs(experiment_ids=["1", "2"])
```

## `MlflowClient` — la API de bajo nivel

`mlflow.search_runs` es un atajo sobre `MlflowClient`, útil cuando se necesita más control (paginación, objetos `Run` en vez de DataFrame):

```python
from mlflow.tracking import MlflowClient

client = MlflowClient(tracking_uri="http://localhost:5000")

experiment = client.get_experiment_by_name("claro-rd-demand-forecast")

runs = client.search_runs(
    experiment_ids=[experiment.experiment_id],
    filter_string="metrics.rmse < 20",
    order_by=["metrics.rmse ASC"],
    max_results=5,
)

for run in runs:
    print(run.info.run_id, run.data.metrics["rmse"])
```

### Paginación manual

```python
results = client.search_runs(experiment_ids=["1"], max_results=50)
token = results.token  # si hay más resultados

while token:
    results = client.search_runs(experiment_ids=["1"], max_results=50, page_token=token)
    token = results.token
```

## Obtener el historial completo de una métrica

`search_runs` solo trae el **último** valor de cada métrica. Para la serie completa (ej. loss por época):

```python
history = client.get_metric_history(run_id="a1b2c3...", key="train_loss")
for m in history:
    print(m.step, m.value, m.timestamp)
```

## Comparar runs desde la UI

En `mlflow ui`, seleccionando varios runs (checkbox) y pulsando **"Compare"**:

- **Parallel Coordinates Plot**: útil para ver visualmente qué combinación de hiperparámetros correlaciona con mejores métricas.
- **Scatter Plot**: comparar dos métricas/params entre sí.
- **Box Plot**: distribución de una métrica agrupada por un parámetro categórico.
- Tabla comparativa lado a lado de params, metrics y tags.

## Organización — experiments, tags y nombres de run

Buenas prácticas de nomenclatura que evitan que el Tracking se vuelva un caos al crecer el equipo:

```python
# Un experiment por proyecto/caso de uso, NO uno por persona ni por día
mlflow.set_experiment("claro-rd-demand-forecast")

# run_name descriptivo del enfoque, no genérico
with mlflow.start_run(run_name="xgboost-90d-window-v2"):
    ...

# Tags para dimensiones que quieras filtrar después
mlflow.set_tags({
    "algorithm_family": "gradient_boosting",
    "data_version": "2026-08-01",
    "git_commit": "a1b2c3d",
    "triggered_by": "scheduled_retrain",
})
```

Los tags de sistema `mlflow.source.git.commit` y `mlflow.source.name` se capturan **automáticamente** si el código corre dentro de un repo git — no hace falta setearlos a mano.

## Renombrar, archivar y eliminar experiments/runs

```python
client.rename_experiment(experiment_id="1", new_name="claro-rd-demand-forecast-v2")

client.delete_run(run_id="a1b2c3...")     # soft-delete (recuperable)
client.restore_run(run_id="a1b2c3...")

client.delete_experiment(experiment_id="1")   # soft-delete
client.restore_experiment(experiment_id="1")
```

> Los `delete_*` son **soft deletes**: el registro se marca como eliminado pero sigue en la base de datos por un período de retención configurable (`mlflow gc` los purga definitivamente — ver [[13 - CLI Reference]]).

## Exportar resultados para reporting

```python
runs_df = mlflow.search_runs(experiment_names=["claro-rd-demand-forecast"])
runs_df.to_csv("experimentos_resumen.csv", index=False)

# Comparación rápida de las top-5 corridas por rmse
top5 = runs_df.sort_values("metrics.rmse").head(5)
print(top5[["tags.mlflow.runName", "metrics.rmse", "metrics.mae"]])
```

## Ver también

- [[02 - Tracking - Fundamentos y API de Logging]]
- [[07 - Model Registry]]
- [[10 - Hyperparameter Tuning con MLflow]]
