---
tags: [scikit-learn, machine-learning, integraciones, xgboost, mlflow, optuna, shap, cheat-sheet]
---

# 15 — Integraciones con el Ecosistema

> Consolida referencias de todo el cheat-sheet. Ver también `Machine Learning/07-Librerias-de-Data-Science-y-ML.md` para el panorama general de librerías.

scikit-learn rara vez se usa aislado — su valor real está en cuánto se integra con el resto del ecosistema de Python para ciencia de datos.

## pandas — el input natural

```python
import pandas as pd
from sklearn import set_config

set_config(transform_output="pandas")   # transformers devuelven DataFrame, no numpy array

pipeline.fit(X_train_df, y_train)   # X_train_df: pandas.DataFrame, con nombres de columna preservados

X_transformado = pipeline[:-1].transform(X_test_df)   # slice del pipeline SIN el paso final (el modelo)
print(X_transformado.columns)   # con transform_output="pandas", los nombres de columna se conservan
```

Desde scikit-learn 1.2+, `set_config(transform_output="pandas")` es la forma recomendada de mantener nombres de columna a través de todo un `Pipeline`/`ColumnTransformer` — antes de esto, encadenar transformadores perdía los nombres y devolvía arrays de NumPy sin etiquetas, complicando la interpretación posterior (`feature_importances_`, `coef_`).

## NumPy — la base matemática subyacente

Todos los estimadores operan internamente sobre arrays de NumPy (o matrices dispersas de SciPy) — entender esto importa para depurar errores de forma/tipo:

```python
import numpy as np

X_train.shape   # (n_muestras, n_features) — SIEMPRE 2D, incluso con una sola feature
X_train.values.reshape(-1, 1)   # error común: una sola feature necesita explícitamente forma 2D
```

## Matplotlib — visualización nativa de resultados

```python
from sklearn.tree import plot_tree
from sklearn.metrics import RocCurveDisplay, ConfusionMatrixDisplay, PrecisionRecallDisplay
from sklearn.inspection import PartialDependenceDisplay, DecisionBoundaryDisplay

# Curvas de dependencia parcial — cómo afecta UNA feature a la predicción, manteniendo las demás fijas
PartialDependenceDisplay.from_estimator(modelo, X_train, features=["dias_atras", "temperatura"])

# Frontera de decisión en 2D (solo para 2 features a la vez, útil pedagógicamente)
DecisionBoundaryDisplay.from_estimator(modelo, X_train[["feature1", "feature2"]], response_method="predict")
```

Ver también [[05 - Métricas y Evaluación]] para `RocCurveDisplay`/`ConfusionMatrixDisplay`/`PrecisionRecallDisplay`.

## XGBoost y LightGBM — API compatible con scikit-learn

```python
from xgboost import XGBClassifier, XGBRegressor
from lightgbm import LGBMClassifier, LGBMRegressor

modelo_xgb = XGBRegressor(n_estimators=300, learning_rate=0.05, max_depth=6)
modelo_xgb.fit(X_train, y_train)   # misma API: fit/predict/score

# Funcionan DENTRO de un Pipeline y de GridSearchCV/RandomizedSearchCV exactamente igual que un modelo nativo:
pipeline = Pipeline([
    ("preprocesamiento", column_transformer),
    ("modelo", XGBRegressor(n_estimators=300)),
])
```

Tanto XGBoost como LightGBM implementan un "wrapper" (`XGBClassifier`/`XGBRegressor`, `LGBMClassifier`/`LGBMRegressor`) que respeta el contrato de scikit-learn (`BaseEstimator`, `fit`/`predict`/`get_params`) — esto es lo que permite usarlos dentro de `Pipeline`, `cross_val_score`, `GridSearchCV`, `VotingClassifier`, `StackingClassifier` sin ningún adaptador adicional, a pesar de ser librerías completamente independientes con su propia implementación en C++.

## Optuna — búsqueda de hiperparámetros avanzada

```python
from optuna.integration import OptunaSearchCV
from optuna.distributions import IntDistribution, FloatDistribution

search = OptunaSearchCV(
    estimator=pipeline,
    param_distributions={
        "modelo__n_estimators": IntDistribution(100, 600),
        "modelo__max_depth": IntDistribution(3, 15),
    },
    n_trials=100, cv=5,
)
search.fit(X_train, y_train)
```

Ver `Optuna/08 - Integraciones con Frameworks de ML.md` para el detalle completo — `OptunaSearchCV` es un reemplazo directo de `GridSearchCV`/`RandomizedSearchCV` con optimización bayesiana (TPE) en vez de búsqueda exhaustiva o aleatoria.

## MLflow — tracking y registro de modelos

```python
import mlflow
mlflow.sklearn.autolog()   # captura automáticamente params, metrics y el modelo mismo

with mlflow.start_run():
    pipeline.fit(X_train, y_train)   # todo el logging ocurre automáticamente
```

`mlflow.sklearn.autolog()` soporta `Pipeline` completos de forma nativa — el modelo logueado incluye todos los pasos de preprocesamiento, no solo el estimador final. Ver `MLflow/05 - Autologging en Profundidad.md` y `MLflow/06 - Model Format y Flavors.md`.

## SHAP — interpretabilidad más allá de `feature_importances_`

```bash
pip install shap
```

```python
import shap

explainer = shap.TreeExplainer(pipeline.named_steps["modelo"])   # para modelos basados en árboles
X_transformado = pipeline[:-1].transform(X_test)   # aplicar el preprocesamiento, SIN el modelo final
shap_values = explainer.shap_values(X_transformado)

shap.summary_plot(shap_values, X_transformado)
```

A diferencia de `feature_importances_` (importancia global, agregada) o `permutation_importance` (también global), SHAP explica **predicciones individuales**: cuánto contribuyó cada feature a alejar la predicción de una muestra específica del valor promedio — ver también `Machine Learning/39-Interpretabilidad-de-Modelos.md`.

## `imbalanced-learn` — resampling compatible con Pipeline

Ya cubierto en detalle en [[11 - Datos Faltantes y Clases Desbalanceadas]] — vale recordar que requiere `imblearn.pipeline.Pipeline`, no el `Pipeline` estándar de scikit-learn, para que el resampling se aplique correctamente solo dentro de cada fold de entrenamiento.

## `category_encoders` — encoders adicionales más allá de `sklearn.preprocessing`

```bash
pip install category_encoders
```

```python
from category_encoders import TargetEncoder, CatBoostEncoder, WOEEncoder

encoder = CatBoostEncoder()   # target encoding con un esquema de ordenamiento que reduce leakage aún más
X_encoded = encoder.fit_transform(X_train[["oficina_id"]], y_train)
```

Complementa el `TargetEncoder` nativo de scikit-learn (ver [[02 - Preprocessing y Escalado]]) con variantes adicionales (`CatBoostEncoder`, `WOEEncoder`, `JamesSteinEncoder`) que usan esquemas distintos de suavizado/ordenamiento para reducir aún más el riesgo de leakage en encoding de alta cardinalidad — todos compatibles con la API `fit`/`transform` de scikit-learn.

## `feature-engine` — transformadores adicionales orientados a feature engineering

```bash
pip install feature-engine
```

```python
from feature_engine.outliers import Winsorizer
from feature_engine.creation import CyclicalFeatures

winsorizer = Winsorizer(capping_method="quantiles", tail="both", fold=0.05)
X_capped = winsorizer.fit_transform(X_train)
```

Librería complementaria con transformadores listos para casos que scikit-learn no cubre directamente de forma nativa (winsorización, features cíclicas de calendario, discretización supervisada) — todos compatibles con `Pipeline`, siguiendo el mismo contrato `BaseEstimator`/`TransformerMixin`.

## Tabla resumen de integraciones

| Librería | Qué aporta a scikit-learn |
|---|---|
| pandas | Input/output con nombres de columna preservados (`set_config`) |
| XGBoost / LightGBM | Modelos de boosting de alto rendimiento, API 100% compatible |
| Optuna | Búsqueda de hiperparámetros bayesiana (`OptunaSearchCV`) |
| MLflow | Tracking, registro y despliegue de modelos/pipelines |
| SHAP | Interpretabilidad a nivel de predicción individual |
| imbalanced-learn | Resampling seguro contra leakage (`imblearn.pipeline.Pipeline`) |
| category_encoders | Encoders adicionales para alta cardinalidad |
| feature-engine | Transformadores adicionales de feature engineering |

## Ver también

- [[07 - Árboles y Ensambles - Sintaxis y API]]
- `MLflow/05 - Autologging en Profundidad.md`
- `Optuna/08 - Integraciones con Frameworks de ML.md`
- `Machine Learning/07-Librerias-de-Data-Science-y-ML.md`
