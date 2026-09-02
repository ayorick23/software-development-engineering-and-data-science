---
tags: [gradient-boosting, xgboost, scikit-learn, cheat-sheet]
---

# 03 — XGBoost: API Scikit-Learn Compatible e Hiperparámetros

> Continúa de [[02 - XGBoost - API Nativa y DMatrix]]. Ver también `Scikit-learn/15 - Integraciones con el Ecosistema.md`.

## `XGBRegressor` / `XGBClassifier` — el wrapper compatible con sklearn

```python
from xgboost import XGBRegressor

modelo = XGBRegressor(
    n_estimators=300,
    max_depth=6,
    learning_rate=0.05,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_alpha=0.1,
    reg_lambda=1.0,
    random_state=42,
    n_jobs=-1,
)
modelo.fit(X_train, y_train)   # misma API que cualquier estimador de scikit-learn
predicciones = modelo.predict(X_test)
```

Funciona directamente dentro de `Pipeline`, `GridSearchCV`, `cross_val_score`, `VotingRegressor`, `StackingRegressor` — sin ningún adaptador adicional, ver `Scikit-learn/03 - Pipelines y ColumnTransformer.md`.

## Hiperparámetros clave — guía de referencia

| Hiperparámetro | Qué controla | Efecto de aumentarlo |
|---|---|---|
| `n_estimators` | Número de árboles secuenciales | Más capacidad, más riesgo de overfitting sin early stopping |
| `max_depth` | Profundidad máxima de cada árbol | Árboles más complejos, más riesgo de overfitting |
| `learning_rate` (alias `eta`) | Peso de la corrección de cada árbol | Menor = más conservador, requiere más `n_estimators` |
| `subsample` | Fracción de filas usadas por árbol | Menor = más regularización (como bagging parcial) |
| `colsample_bytree` | Fracción de columnas usadas por árbol | Menor = más regularización, árboles más diversos |
| `reg_alpha` | Regularización L1 sobre pesos de hojas | Mayor = más hojas exactamente en cero |
| `reg_lambda` | Regularización L2 sobre pesos de hojas | Mayor = pesos de hojas más pequeños/suaves |
| `gamma` | Ganancia mínima para aceptar un split | Mayor = árboles más simples, poda más agresiva |
| `min_child_weight` | Suma mínima de pesos (Hessiano) en una hoja | Mayor = evita hojas basadas en muy pocas muestras |

Ver [[36-Ensambles-en-Profundidad]] para la explicación matemática de por qué `reg_alpha`/`reg_lambda`/`gamma` funcionan de esta forma (Newton boosting con regularización explícita en la función objetivo).

## `early_stopping_rounds` — vía API sklearn-compatible

```python
modelo = XGBRegressor(
    n_estimators=1000,   # límite superior alto — early stopping decide el número REAL de árboles
    learning_rate=0.03,
    early_stopping_rounds=20,
    eval_metric="mae",
)
modelo.fit(
    X_train, y_train,
    eval_set=[(X_val, y_val)],
    verbose=50,
)

print(modelo.best_iteration)
print(modelo.best_score)
```

Detiene el entrenamiento cuando la métrica de validación deja de mejorar durante `early_stopping_rounds` rondas consecutivas — la forma estándar de evitar overfitting sin tener que adivinar `n_estimators` manualmente, y de aprovechar todo el presupuesto de tiempo posible sin desperdiciarlo en árboles que ya no aportan.

## `predict_proba` — clasificación

```python
from xgboost import XGBClassifier

modelo = XGBClassifier(n_estimators=300, max_depth=5, learning_rate=0.05)
modelo.fit(X_train, y_train)

modelo.predict(X_test)              # clases predichas
modelo.predict_proba(X_test)[:, 1]  # probabilidad de la clase positiva
```

## `enable_categorical` — soporte experimental de categóricas nativas

```python
modelo = XGBRegressor(
    n_estimators=300,
    tree_method="hist",         # requerido para enable_categorical
    enable_categorical=True,
)
X_train["region"] = X_train["region"].astype("category")   # requiere dtype "category" explícito de pandas
modelo.fit(X_train, y_train)
```

A diferencia de CatBoost (donde el manejo de categóricas es el diseño central de la librería, ver [[06 - CatBoost - Ordered Boosting y Manejo de Categóricas]]), en XGBoost esta funcionalidad es más reciente y menos sofisticada — para datasets con muchas columnas categóricas de alta cardinalidad, CatBoost o `TargetEncoder` (ver `Scikit-learn/02 - Preprocessing y Escalado.md`) previo suelen dar mejores resultados que depender de `enable_categorical` en XGBoost.

## Integración completa en un Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import OneHotEncoder

preprocesador = ColumnTransformer([
    ("cat", OneHotEncoder(handle_unknown="ignore"), columnas_categoricas),
], remainder="passthrough")

pipeline = Pipeline([
    ("preprocesamiento", preprocesador),
    ("modelo", XGBRegressor(n_estimators=300, max_depth=6, learning_rate=0.05)),
])
pipeline.fit(X_train, y_train)
```

## `GridSearchCV`/`RandomizedSearchCV` sobre XGBoost

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

param_distributions = {
    "modelo__n_estimators": randint(100, 800),
    "modelo__max_depth": randint(3, 10),
    "modelo__learning_rate": uniform(0.01, 0.3),
    "modelo__subsample": uniform(0.6, 0.4),
}

search = RandomizedSearchCV(pipeline, param_distributions, n_iter=50, cv=5, n_jobs=-1)
search.fit(X_train, y_train)
```

Para búsquedas más eficientes que Grid/Random Search, ver `Optuna/08 - Integraciones con Frameworks de ML.md` (incluye `XGBoostPruningCallback` para abandonar trials poco prometedores por ronda de boosting).

## Feature importance y visualización

```python
import pandas as pd

importancias = pd.Series(
    modelo.feature_importances_,
    index=X_train.columns,
).sort_values(ascending=False)

# Con más detalle que feature_importances_ (que por defecto usa "gain"):
booster = modelo.get_booster()
print(booster.get_score(importance_type="gain"))
```

## Advertencias de migración de versión

XGBoost ha cambiado defaults importantes entre versiones mayores — por ejemplo, `tree_method="hist"` pasó a ser el método recomendado (y eventualmente default) sobre `"exact"`, y el manejo de `early_stopping_rounds` se movió del método `.fit()` al constructor en versiones recientes (2.x). Al actualizar la librería en un proyecto existente, revisar el changelog oficial antes de asumir que el código anterior sigue funcionando igual — a diferencia de scikit-learn (que mantiene compatibilidad hacia atrás de forma más conservadora), XGBoost ha sido más agresivo modernizando su API entre versiones mayores.

## Ver también

- [[02 - XGBoost - API Nativa y DMatrix]]
- [[08 - Comparativa Técnica y Tuning Cruzado]]
- `Scikit-learn/04 - Model Selection - Validación y Búsqueda.md`
- `Optuna/08 - Integraciones con Frameworks de ML.md`
