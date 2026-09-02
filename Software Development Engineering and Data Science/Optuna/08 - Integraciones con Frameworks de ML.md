---
tags: [optuna, hiperparametros, integraciones, mlflow, sklearn, cheat-sheet]
---

# 08 — Integraciones con Frameworks de ML

> Se apoya en [[01 - Introducción y Conceptos Fundamentales]] y [[04 - Pruners en Profundidad]].

El paquete `optuna-integration` (separado del core desde Optuna 3.3) contiene adaptadores listos para usar con los frameworks de ML más comunes, evitando escribir el loop de `trial.report()`/`should_prune()` a mano en cada uno.

```bash
pip install optuna-integration
```

## `OptunaSearchCV` — reemplazo directo de `GridSearchCV`/`RandomizedSearchCV`

Para quien ya tiene código basado en la API de sklearn y quiere el beneficio de TPE sin reescribir la función objetivo manualmente:

```python
from optuna.integration import OptunaSearchCV
from optuna.distributions import IntDistribution, FloatDistribution

param_distributions = {
    "n_estimators": IntDistribution(100, 600),
    "max_depth": IntDistribution(3, 15),
    "learning_rate": FloatDistribution(0.01, 0.3, log=True),
}

search = OptunaSearchCV(
    estimator=XGBRegressor(),
    param_distributions=param_distributions,
    n_trials=100,
    cv=5,
    scoring="neg_mean_absolute_error",
)
search.fit(X_train, y_train)

print(search.best_params_)   # misma API que GridSearchCV/RandomizedSearchCV
print(search.best_estimator_)
```

Ventaja sobre `GridSearchCV`/`RandomizedSearchCV`: usa TPE en vez de grilla exhaustiva o muestreo puramente aleatorio, con la misma interfaz `.fit()`/`.best_params_`/`.best_estimator_` que ya conoces de sklearn — migración casi directa.

> **Precaución con series de tiempo**: `OptunaSearchCV` usa `cv` de sklearn, que por defecto no respeta el orden temporal. Para datos secuenciales, pasar explícitamente `cv=TimeSeriesSplit(n_splits=5)` — de lo contrario se reintroduce el leakage temporal que [[45-Optimizacion-de-Hiperparametros]] advierte evitar con walk-forward validation manual.

## LightGBM Tuner — tuning automático especializado

```python
import optuna.integration.lightgbm as lgb_tuner

dtrain = lgb_tuner.Dataset(X_train, label=y_train)
dval = lgb_tuner.Dataset(X_val, label=y_val)

params = {
    "objective": "regression",
    "metric": "mae",
    "verbosity": -1,
}

modelo = lgb_tuner.train(
    params, dtrain,
    valid_sets=[dval],
    num_boost_round=1000,
)
print(modelo.params)   # hiperparámetros óptimos encontrados
```

`LightGBMTuner` no es un `objective` genérico — es un afinador **especializado** que conoce de antemano qué hiperparámetros de LightGBM importan más y en qué orden ajustarlos (usa una estrategia de tuning secuencial por grupos de hiperparámetros, más eficiente que una búsqueda genérica para este caso específico).

## Integración con PyTorch — pruning por época

```python
import optuna
from optuna.integration import PyTorchLightningPruningCallback

def objective(trial):
    modelo = ModeloLightning(
        lr=trial.suggest_float("lr", 1e-5, 1e-1, log=True),
        n_layers=trial.suggest_int("n_layers", 1, 4),
    )
    trainer = pl.Trainer(
        max_epochs=30,
        callbacks=[PyTorchLightningPruningCallback(trial, monitor="val_loss")],
    )
    trainer.fit(modelo)
    return trainer.callback_metrics["val_loss"].item()

study = optuna.create_study(direction="minimize", pruner=optuna.pruners.HyperbandPruner())
study.optimize(objective, n_trials=50)
```

El callback reporta automáticamente `val_loss` a Optuna después de cada época (equivalente a llamar `trial.report()` manualmente en cada `on_epoch_end`) y lanza `TrialPruned` si corresponde — evita escribir el loop de entrenamiento manual solo para insertar el pruning.

## Integración con Keras/TensorFlow

```python
from optuna.integration import TFKerasPruningCallback

def objective(trial):
    modelo = construir_modelo(
        n_units=trial.suggest_int("n_units", 16, 256, log=True),
        dropout=trial.suggest_float("dropout", 0.0, 0.5),
    )
    modelo.fit(
        X_train, y_train,
        validation_data=(X_val, y_val),
        epochs=30,
        callbacks=[TFKerasPruningCallback(trial, "val_loss")],
        verbose=0,
    )
    return modelo.evaluate(X_val, y_val, verbose=0)[0]
```

## Integración con XGBoost — pruning por ronda de boosting

```python
from optuna.integration import XGBoostPruningCallback

def objective(trial):
    params = {
        "max_depth": trial.suggest_int("max_depth", 3, 10),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
    }
    pruning_callback = XGBoostPruningCallback(trial, "validation_0-mae")

    modelo = XGBRegressor(**params, n_estimators=500)
    modelo.fit(
        X_train, y_train,
        eval_set=[(X_val, y_val)],
        callbacks=[pruning_callback],
        verbose=False,
    )
    return mean_absolute_error(y_val, modelo.predict(X_val))
```

## Integración con MLflow — trazabilidad completa de la búsqueda

Ya cubierto en detalle en [[45-Optimizacion-de-Hiperparametros]] (patrón manual con `nested=True`) y en `MLflow/10 - Hyperparameter Tuning con MLflow.md`. `optuna-integration` también ofrece un callback dedicado que automatiza el logging sin escribirlo trial por trial:

```python
from optuna.integration.mlflow import MLflowCallback
import mlflow

mlflow.set_tracking_uri("http://mlflow-server:5000")
mlflow.set_experiment("claro-rd-demand-forecast-tuning")

mlflc = MLflowCallback(
    tracking_uri=mlflow.get_tracking_uri(),
    metric_name="mae",
    create_experiment=False,   # usa el experiment ya seteado con mlflow.set_experiment
)

@mlflc.track_in_mlflow()   # decorador: cada trial se loguea automáticamente como un run
def objective(trial):
    params = sugerir_parametros(trial)
    modelo = entrenar(params)
    return evaluar(modelo)

study = optuna.create_study(direction="minimize")
study.optimize(objective, n_trials=100, callbacks=[mlflc])
```

## Integración con Weights & Biases

```python
from optuna.integration.wandb import WeightsAndBiasesCallback

wandb_kwargs = {"project": "claro-rd-demand-forecast"}
wandbc = WeightsAndBiasesCallback(wandb_kwargs=wandb_kwargs, as_multirun=True)

@wandbc.track_in_wandb()
def objective(trial):
    ...

study.optimize(objective, n_trials=100, callbacks=[wandbc])
```

## Tabla resumen de integraciones disponibles

| Framework | Módulo | Qué automatiza |
|---|---|---|
| scikit-learn | `OptunaSearchCV` | Búsqueda con API idéntica a `GridSearchCV` |
| LightGBM | `optuna.integration.lightgbm` | Tuning secuencial especializado |
| PyTorch Lightning | `PyTorchLightningPruningCallback` | Pruning automático por época |
| TensorFlow/Keras | `TFKerasPruningCallback` | Pruning automático por época |
| XGBoost | `XGBoostPruningCallback` | Pruning automático por ronda de boosting |
| MLflow | `MLflowCallback` | Logging automático de cada trial como run |
| Weights & Biases | `WeightsAndBiasesCallback` | Logging automático a W&B |

## Ver también

- [[04 - Pruners en Profundidad]]
- `MLflow/10 - Hyperparameter Tuning con MLflow.md`
- `Machine Learning/45-Optimizacion-de-Hiperparametros.md`
