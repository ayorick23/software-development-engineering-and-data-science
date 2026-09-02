---
tags: [gradient-boosting, lightgbm, scikit-learn, cheat-sheet]
---

# 05 — LightGBM: API Scikit-Learn Compatible y Categóricas Nativas

> Continúa de [[04 - LightGBM - Arquitectura Leaf-wise y API Nativa]].

## `LGBMRegressor` / `LGBMClassifier`

```python
from lightgbm import LGBMRegressor

modelo = LGBMRegressor(
    n_estimators=300,
    num_leaves=31,
    learning_rate=0.05,
    feature_fraction=0.8,
    bagging_fraction=0.8,
    bagging_freq=5,
    random_state=42,
    n_jobs=-1,
    verbosity=-1,
)
modelo.fit(X_train, y_train)
predicciones = modelo.predict(X_test)
```

Compatible con `Pipeline`, `GridSearchCV`, `VotingRegressor`, `StackingRegressor` — mismo contrato que cualquier estimador de scikit-learn.

## Hiperparámetros clave — guía de referencia

| Hiperparámetro | Qué controla | Equivalente aproximado en XGBoost |
|---|---|---|
| `num_leaves` | Complejidad del árbol (el más importante en LightGBM) | `max_depth` (indirectamente) |
| `max_depth` | Límite adicional de profundidad (a menudo `-1`) | `max_depth` |
| `learning_rate` | Peso de cada corrección | `learning_rate` |
| `n_estimators` | Número de árboles | `n_estimators` |
| `feature_fraction` | Fracción de columnas por árbol | `colsample_bytree` |
| `bagging_fraction` | Fracción de filas por iteración | `subsample` |
| `min_child_samples` | Mínimo de muestras por hoja | `min_child_weight` (concepto similar) |
| `lambda_l1` / `lambda_l2` | Regularización L1/L2 | `reg_alpha` / `reg_lambda` |
| `min_split_gain` | Ganancia mínima para aceptar un split | `gamma` |

Ver tabla comparativa completa en [[08 - Comparativa Técnica y Tuning Cruzado]].

## `early_stopping` — vía API sklearn-compatible

```python
from lightgbm import early_stopping, log_evaluation

modelo = LGBMRegressor(n_estimators=1000, learning_rate=0.03, num_leaves=31)
modelo.fit(
    X_train, y_train,
    eval_set=[(X_val, y_val)],
    eval_metric="mae",
    callbacks=[early_stopping(stopping_rounds=20), log_evaluation(period=50)],
)

print(modelo.best_iteration_)
```

## Manejo de categóricas — vía API sklearn-compatible

```python
modelo = LGBMRegressor(n_estimators=300)

X_train["region"] = X_train["region"].astype("category")   # dtype "category" de pandas — detección automática

modelo.fit(X_train, y_train, categorical_feature=["region", "tipo_producto"])
```

Con columnas `dtype="category"` en un DataFrame de pandas, LightGBM las detecta automáticamente sin necesitar especificar `categorical_feature` explícitamente en muchos casos — aunque especificarlo explícitamente evita ambigüedad, especialmente cuando las categorías están codificadas como enteros y podrían confundirse con features numéricas continuas.

> **Diferencia clave con CatBoost**: LightGBM necesita que el *tipo de dato* de la columna esté marcado como categórico (o se indique explícitamente vía `categorical_feature`) — no infiere automáticamente qué columnas son categóricas a partir de datos crudos tipo string sin ese paso previo. CatBoost, en cambio, puede trabajar directamente con columnas de texto/categorías sin ninguna preparación (ver [[06 - CatBoost - Ordered Boosting y Manejo de Categóricas]]).

## Controlar el overfitting — el ajuste más específico de LightGBM

Dado que el crecimiento leaf-wise es más agresivo que level-wise, LightGBM suele necesitar regularización más explícita en datasets pequeños-medianos:

```python
modelo = LGBMRegressor(
    n_estimators=1000,
    learning_rate=0.03,        # más bajo = más conservador
    num_leaves=15,               # más bajo que el default (31) en datasets pequeños
    min_child_samples=30,        # más alto = evita hojas basadas en pocas muestras
    lambda_l1=0.1,
    lambda_l2=1.0,
    feature_fraction=0.7,
    bagging_fraction=0.7,
    bagging_freq=5,
)
```

Si a pesar de esto el modelo sigue sobreajustando, la recomendación estándar de la documentación oficial es reducir `num_leaves` antes que cualquier otro hiperparámetro — es la palanca de mayor efecto directo sobre la complejidad del modelo en esta librería.

## Integración en Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer

pipeline = Pipeline([
    ("preprocesamiento", preprocesador),   # ColumnTransformer con imputación/encoding donde aplique
    ("modelo", LGBMRegressor(n_estimators=300, num_leaves=31, learning_rate=0.05)),
])
pipeline.fit(X_train, y_train)
```

## `LGBMClassifier` — clasificación

```python
from lightgbm import LGBMClassifier

modelo = LGBMClassifier(n_estimators=300, num_leaves=31, class_weight="balanced")
modelo.fit(X_train, y_train)

modelo.predict_proba(X_test)[:, 1]
```

`class_weight="balanced"` funciona igual que en scikit-learn (ver `Scikit-learn/06 - Modelos Lineales - Sintaxis y API.md`) — útil para clases desbalanceadas sin recurrir a resampling.

## Feature importance

```python
import pandas as pd

importancias = pd.Series(modelo.feature_importances_, index=X_train.columns).sort_values(ascending=False)
```

## Nota de instalación en Windows

LightGBM en Windows requiere, en algunos casos, el runtime de Visual C++ (Microsoft Visual C++ Redistributable) para funcionar correctamente, especialmente con soporte GPU — si `pip install lightgbm` falla o el import da error de DLL, verificar que esté instalado antes de asumir un problema con la librería en sí.

## Ver también

- [[04 - LightGBM - Arquitectura Leaf-wise y API Nativa]]
- [[06 - CatBoost - Ordered Boosting y Manejo de Categóricas]]
- [[08 - Comparativa Técnica y Tuning Cruzado]]
