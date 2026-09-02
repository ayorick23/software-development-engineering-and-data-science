---
tags: [mlflow, mlops, hyperparameter-tuning, optuna, cheat-sheet]
---

# 10 — Hyperparameter Tuning con MLflow

> Se apoya en [[02 - Tracking - Fundamentos y API de Logging]] (runs anidados) y [[04 - Tracking - Búsqueda, Comparación y Organización]].

MLflow no incluye un optimizador de hiperparámetros propio — se integra con librerías especializadas (Optuna, Hyperopt, Ray Tune) usando el patrón de **runs anidados**: un run padre agrupa la búsqueda completa, y cada combinación probada es un run hijo.

## El patrón general: parent run + nested runs

```python
import mlflow

with mlflow.start_run(run_name="busqueda-hiperparametros") as parent:
    for params in combinaciones_a_probar:
        with mlflow.start_run(run_name=f"trial-{params}", nested=True):
            mlflow.log_params(params)
            model = entrenar(params)
            score = evaluar(model)
            mlflow.log_metric("rmse", score)
```

En la UI, el run padre muestra todos sus hijos agrupados, lo que permite comparar visualmente decenas o cientos de intentos sin perder la organización del experimento.

## Integración con Optuna

```python
import optuna
import mlflow

mlflow.set_experiment("claro-rd-demand-forecast-tuning")

def objective(trial):
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 500),
        "max_depth": trial.suggest_int("max_depth", 3, 10),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
    }

    with mlflow.start_run(nested=True):
        mlflow.log_params(params)
        model = XGBRegressor(**params)
        model.fit(X_train, y_train)
        rmse = mean_squared_error(y_val, model.predict(X_val), squared=False)
        mlflow.log_metric("rmse", rmse)

    return rmse

with mlflow.start_run(run_name="optuna-study"):
    study = optuna.create_study(direction="minimize")
    study.optimize(objective, n_trials=50)

    mlflow.log_params(study.best_params)
    mlflow.log_metric("best_rmse", study.best_value)
```

### Callback nativo de Optuna para MLflow

Optuna incluye un callback que automatiza parte de este logging:

```python
from optuna.integration.mlflow import MLflowCallback

mlflow_callback = MLflowCallback(
    tracking_uri=mlflow.get_tracking_uri(),
    metric_name="rmse",
)

study = optuna.create_study(direction="minimize")
study.optimize(objective, n_trials=50, callbacks=[mlflow_callback])
```

## Integración con Hyperopt

```python
from hyperopt import fmin, tpe, hp, Trials
import mlflow

space = {
    "n_estimators": hp.quniform("n_estimators", 100, 500, 50),
    "max_depth": hp.quniform("max_depth", 3, 10, 1),
    "learning_rate": hp.loguniform("learning_rate", -4.6, -1.2),
}

def objective(params):
    params["n_estimators"] = int(params["n_estimators"])
    params["max_depth"] = int(params["max_depth"])

    with mlflow.start_run(nested=True):
        mlflow.log_params(params)
        model = XGBRegressor(**params)
        model.fit(X_train, y_train)
        rmse = mean_squared_error(y_val, model.predict(X_val), squared=False)
        mlflow.log_metric("rmse", rmse)

    return {"loss": rmse, "status": "ok"}

with mlflow.start_run(run_name="hyperopt-search"):
    best = fmin(fn=objective, space=space, algo=tpe.suggest, max_evals=50, trials=Trials())
```

## Analizar resultados de la búsqueda

```python
runs_df = mlflow.search_runs(
    experiment_names=["claro-rd-demand-forecast-tuning"],
    filter_string="tags.mlflow.parentRunId = 'a1b2c3...'",   # solo los hijos de un padre específico
    order_by=["metrics.rmse ASC"],
)

mejor_trial = runs_df.iloc[0]
print(mejor_trial[["params.n_estimators", "params.max_depth", "metrics.rmse"]])
```

Visualización recomendada en la UI: **Parallel Coordinates Plot** sobre el run padre — permite ver de un vistazo qué rango de hiperparámetros correlaciona con mejor `rmse`, más rápido que leer una tabla.

## Registrar solo el mejor modelo tras la búsqueda

Patrón habitual: no registrar cada trial en el Model Registry (serían demasiadas versiones ruido), solo el ganador final:

```python
best_run_id = runs_df.iloc[0]["run_id"]

mejor_params = {
    "n_estimators": int(runs_df.iloc[0]["params.n_estimators"]),
    "max_depth": int(runs_df.iloc[0]["params.max_depth"]),
}

with mlflow.start_run(run_name="modelo-final-tuning"):
    mlflow.log_params(mejor_params)
    modelo_final = XGBRegressor(**mejor_params)
    modelo_final.fit(X_train_full, y_train_full)   # reentrenar con todos los datos disponibles
    mlflow.xgboost.log_model(
        modelo_final, "model",
        registered_model_name="claro-rd-demand-model",
    )
```

## Integración con Ray Tune (búsquedas distribuidas a gran escala)

```python
from ray import tune
from ray.air.integrations.mlflow import MLflowLoggerCallback

tuner = tune.Tuner(
    trainable_function,
    param_space={
        "n_estimators": tune.randint(100, 500),
        "max_depth": tune.randint(3, 10),
    },
    run_config=tune.RunConfig(
        callbacks=[MLflowLoggerCallback(
            tracking_uri=mlflow.get_tracking_uri(),
            experiment_name="claro-rd-demand-forecast-tuning",
        )]
    ),
)
results = tuner.fit()
```

## Ver también

- [[02 - Tracking - Fundamentos y API de Logging]]
- [[04 - Tracking - Búsqueda, Comparación y Organización]]
- [[07 - Model Registry]]
- `Machine Learning/45-Optimizacion-de-Hiperparametros.md`
