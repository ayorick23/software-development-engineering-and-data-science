---
tags: [mlflow, mlops, tracking, autolog, cheat-sheet]
---

# 05 — Autologging en Profundidad

> Relacionado con [[02 - Tracking - Fundamentos y API de Logging]].

**Autologging** activa el registro automático de parámetros, métricas y el modelo mismo, sin llamadas manuales a `log_param`/`log_metric`. Es la forma más rápida de empezar a usar Tracking sobre código ya existente.

## Activación básica — universal

```python
import mlflow

mlflow.autolog()   # detecta automáticamente la librería usada y activa su integración

with mlflow.start_run():
    model.fit(X_train, y_train)   # todo se loguea solo
```

`mlflow.autolog()` es un despachador: internamente detecta qué librerías están importadas (sklearn, XGBoost, PyTorch, etc.) y activa el autolog específico de cada una.

## Activación específica por librería

Cuando se quiere control fino o solo aplica a una librería:

```python
import mlflow.sklearn
mlflow.sklearn.autolog()

import mlflow.xgboost
mlflow.xgboost.autolog()

import mlflow.pytorch
mlflow.pytorch.autolog()

import mlflow.lightgbm
mlflow.lightgbm.autolog()

import mlflow.spark
mlflow.spark.autolog()

import mlflow.tensorflow
mlflow.tensorflow.autolog()
```

## Qué registra automáticamente (ejemplo: scikit-learn)

```python
import mlflow
mlflow.sklearn.autolog()

from sklearn.ensemble import RandomForestRegressor

with mlflow.start_run():
    model = RandomForestRegressor(n_estimators=200, max_depth=8)
    model.fit(X_train, y_train)
    model.score(X_val, y_val)   # dispara el logging de métricas de evaluación
```

Automáticamente se registran:

- **Params**: todos los hiperparámetros del estimador (`get_params()` de sklearn) — `n_estimators`, `max_depth`, etc.
- **Metrics**: las métricas que produce `.score()` o, en modelos de clasificación, métricas estándar (accuracy, F1, etc.)
- **Model**: el modelo serializado en formato MLflow (ver [[06 - Model Format y Flavors]])
- **Artifacts**: para algunos modelos, gráficas como matriz de confusión, curva ROC
- **Tags**: clase del estimador, versión de la librería

## Parámetros de configuración de `autolog()`

```python
mlflow.sklearn.autolog(
    log_input_examples=True,       # guarda un ejemplo de input para inferir el signature
    log_model_signatures=True,     # infiere y guarda el schema de entrada/salida
    log_models=True,               # si False, solo loguea params/metrics, NO el modelo
    log_datasets=True,             # loguea metadata del dataset de entrenamiento
    disable=False,                 # True desactiva completamente
    exclusive=False,               # True: el autolog gana sobre logging manual dentro del mismo run
    disable_for_unsupported_versions=False,
    silent=False,                  # True suprime warnings/logs de MLflow en consola
)
```

## Autologging para frameworks de deep learning — logging por época

En PyTorch Lightning o TensorFlow/Keras, autolog se integra con el ciclo de entrenamiento y loguea métricas **por época automáticamente**:

```python
import mlflow
mlflow.tensorflow.autolog()

with mlflow.start_run():
    model.fit(
        X_train, y_train,
        validation_data=(X_val, y_val),
        epochs=20,
        callbacks=[],   # no hace falta agregar un callback manual de MLflow
    )
    # cada época queda registrada como un punto en la serie de la métrica (step=epoch)
```

Para PyTorch "puro" (sin Lightning), no hay hooks automáticos de entrenamiento — ahí se necesita logging manual dentro del loop, o usar PyTorch Lightning que sí tiene integración nativa vía `Trainer`.

## Cuándo combinar autolog con logging manual

Autolog no es excluyente por defecto (`exclusive=False`): se pueden agregar métricas de negocio propias encima de lo que autolog ya captura.

```python
mlflow.sklearn.autolog()

with mlflow.start_run():
    model.fit(X_train, y_train)          # autolog captura params + metrics estándar
    preds = model.predict(X_val)

    # Métrica de negocio específica que autolog no conoce:
    mape_negocio = calcular_mape_ponderado(y_val, preds, pesos_por_region)
    mlflow.log_metric("mape_ponderado_negocio", mape_negocio)
```

## Limitaciones importantes

- **No captura todo automáticamente**: métricas de negocio custom, gráficas personalizadas o artefactos específicos del dominio siguen requiriendo logging manual.
- **Preprocesamiento fuera del estimador no se captura**: si haces `scaler.fit_transform(X)` antes de `model.fit()`, esos pasos no quedan en el modelo logueado a menos que uses un `Pipeline` de sklearn (autolog sí soporta `Pipeline` completos, incluyendo pasos de preprocesamiento).
- **Frameworks sin integración nativa** requieren logging manual completo.
- **Puede generar demasiado ruido**: en experimentación con muchas pruebas rápidas, autolog puede llenar el Tracking con runs de baja calidad si no se combina con una estrategia de limpieza (tags para marcar runs "descartables", o `mlflow gc`).

## Ejemplo con `Pipeline` de sklearn (preprocesamiento incluido)

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Ridge

mlflow.sklearn.autolog()

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", Ridge(alpha=1.0)),
])

with mlflow.start_run():
    pipeline.fit(X_train, y_train)
    # el modelo logueado incluye el scaler + el modelo, listo para servir con datos crudos
```

## Ver también

- [[02 - Tracking - Fundamentos y API de Logging]]
- [[06 - Model Format y Flavors]]
- [[10 - Hyperparameter Tuning con MLflow]]
