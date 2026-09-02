---
tags: [gradient-boosting, xgboost, lightgbm, catboost, mlflow, optuna, cheat-sheet]
---

# 11 — Integración con el Ecosistema

> Consolida referencias de todo el cheat-sheet.

## scikit-learn — Pipeline, GridSearchCV, ensambles

Ya cubierto extensamente en cada archivo individual — el punto central es que las tres implementan el contrato de scikit-learn (`Scikit-learn/01 - Introducción, Filosofía y la API Consistente.md`), por lo que participan de:

```python
from sklearn.ensemble import VotingRegressor, StackingRegressor

# Combinar las tres en un ensamble de ensambles:
modelo_combinado = VotingRegressor([
    ("xgb", XGBRegressor(n_estimators=300)),
    ("lgb", LGBMRegressor(n_estimators=300)),
    ("cat", CatBoostRegressor(iterations=300, cat_features=columnas_categoricas, verbose=0)),
])
modelo_combinado.fit(X_train, y_train)
```

Combinar las tres librerías en un `VotingRegressor`/`StackingRegressor` es una técnica legítima y frecuente en competencias de datos — cada una captura patrones ligeramente distintos por sus diferencias arquitectónicas (level-wise vs. leaf-wise vs. symmetric), y promediar sus predicciones suele reducir el error por encima de cualquiera individual. Ver `Scikit-learn/07 - Árboles y Ensambles - Sintaxis y API.md`.

## MLflow — autologging nativo para las tres

```python
import mlflow

mlflow.xgboost.autolog()
mlflow.lightgbm.autolog()
mlflow.catboost.autolog()

with mlflow.start_run():
    modelo_xgb.fit(X_train, y_train, eval_set=[(X_val, y_val)])
    # params, metrics de entrenamiento, y el modelo mismo se loguean automáticamente
```

Las tres tienen módulos de autolog dedicados en MLflow (`mlflow.xgboost`, `mlflow.lightgbm`, `mlflow.catboost`), independientes del `mlflow.sklearn.autolog()` genérico — capturan específicamente las curvas de entrenamiento/validación por iteración (relevante dado que las tres soportan early stopping) además de hiperparámetros y el modelo final. Ver `MLflow/05 - Autologging en Profundidad.md`.

```python
# Logging manual con el flavor específico, si se necesita más control:
mlflow.xgboost.log_model(modelo_xgb, "model")
mlflow.lightgbm.log_model(modelo_lgb, "model")
mlflow.catboost.log_model(modelo_cat, "model")
```

Ver `MLflow/06 - Model Format y Flavors.md` — cada flavor sabe cómo serializar y cargar el formato nativo específico de cada librería (`.json`/`.ubj` para XGBoost, `.txt` para LightGBM, `.cbm` para CatBoost), en vez de depender únicamente de `pickle`/`joblib` genérico.

## Optuna — pruning específico por librería

```python
from optuna.integration import XGBoostPruningCallback

def objective(trial):
    params = {...}
    pruning_callback = XGBoostPruningCallback(trial, "validation_0-mae")
    modelo = XGBRegressor(**params)
    modelo.fit(X_train, y_train, eval_set=[(X_val, y_val)], callbacks=[pruning_callback])
    return mean_absolute_error(y_val, modelo.predict(X_val))
```

Ver `Optuna/08 - Integraciones con Frameworks de ML.md` y `Optuna/04 - Pruners en Profundidad.md` — el pruning por ronda de boosting es especialmente valioso en estas librerías porque cada trial puede tener cientos de iteraciones, y abandonar temprano un trial poco prometedor ahorra tiempo real significativo.

## Persistencia — comparación de formatos

| Librería | Formato nativo recomendado | Estabilidad entre versiones |
|---|---|---|
| XGBoost | `.json` / `.ubj` (`save_model`) | Alta — formato diseñado para portabilidad |
| LightGBM | `.txt` (`save_model`) | Alta |
| CatBoost | `.cbm` (`save_model`) | Alta, incluye también exportación a `.onnx`, `.pmml`, C++ |

Los tres formatos nativos son preferibles a `joblib`/`pickle` genérico para persistencia de largo plazo, por la misma razón cubierta en `Scikit-learn/13 - Persistencia de Modelos.md`: son estables entre versiones de la librería, mientras que `pickle` puede romperse al actualizar. CatBoost destaca por exportar directamente a formatos de despliegue fuera de Python (`.onnx`, `.pmml`, código C++ standalone) sin pasos intermedios.

```python
modelo_cat.save_model("modelo.onnx", format="onnx")   # exportación directa a ONNX, sin skl2onnx externo
```

## Docker — imágenes con las tres librerías

```dockerfile
FROM python:3.11-slim

RUN pip install xgboost lightgbm catboost scikit-learn pandas

COPY modelo_xgboost.json /app/modelo.json
COPY inferencia.py /app/inferencia.py

CMD ["python", "/app/inferencia.py"]
```

Ver `Docker/Introduction to Docker.md` y `MLflow/09 - Model Serving y Despliegue.md` — al construir una imagen de despliegue, conviene fijar versiones exactas de la librería de boosting usada (`xgboost==2.1.0`, no un rango), dado el riesgo de incompatibilidad entre versiones mencionado en la sección de persistencia.

## FastAPI — servir un modelo de boosting directamente

```python
from fastapi import FastAPI
import xgboost as xgb

app = FastAPI()
modelo = xgb.XGBRegressor()
modelo.load_model("modelo.json")

@app.post("/predecir")
def predecir(payload: dict):
    import pandas as pd
    df = pd.DataFrame([payload])
    return {"prediccion": float(modelo.predict(df)[0])}
```

Ver `Machine Learning/49-APIs-con-FastAPI-para-Servir-Modelos.md`.

## Ver también

- `MLflow/05 - Autologging en Profundidad.md`
- `MLflow/06 - Model Format y Flavors.md`
- `Optuna/08 - Integraciones con Frameworks de ML.md`
- `Scikit-learn/15 - Integraciones con el Ecosistema.md`
