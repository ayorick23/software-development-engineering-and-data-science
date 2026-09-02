---
tags: [gradient-boosting, xgboost, lightgbm, catboost, tuning, cheat-sheet]
---

# 08 — Comparativa Técnica y Tuning Cruzado

> Consolida [[03 - XGBoost - API Scikit-Learn Compatible e Hiperparámetros]], [[05 - LightGBM - API Scikit-Learn Compatible y Categóricas Nativas]] y [[07 - CatBoost - API, Pool y Funcionalidades Distintivas]].

## Tabla "Rosetta Stone" — hiperparámetros equivalentes

| Concepto | XGBoost | LightGBM | CatBoost |
|---|---|---|---|
| Número de árboles | `n_estimators` | `n_estimators` | `iterations` |
| Tasa de aprendizaje | `learning_rate` | `learning_rate` | `learning_rate` |
| Profundidad del árbol | `max_depth` | `max_depth` (secundario) | `depth` |
| Complejidad principal del árbol | `max_depth` / `gamma` | **`num_leaves`** | `depth` |
| Fracción de columnas por árbol | `colsample_bytree` | `feature_fraction` | `rsm` (colsample) |
| Fracción de filas por árbol | `subsample` | `bagging_fraction` | `subsample` (bagging_temperature en modo Bayesiano) |
| Regularización L1 | `reg_alpha` | `lambda_l1` | Sin equivalente L1 directo |
| Regularización L2 | `reg_lambda` | `lambda_l2` | `l2_leaf_reg` |
| Mínimo de muestras por hoja | `min_child_weight` | `min_child_samples` | `min_data_in_leaf` |
| Ganancia mínima para split | `gamma` | `min_split_gain` | Regularizado implícitamente vía `l2_leaf_reg` |
| Rondas de early stopping | `early_stopping_rounds` | `early_stopping_rounds` (vía callback) | `early_stopping_rounds` |
| Método de árbol | `tree_method="hist"` | (histogram por diseño) | `grow_policy` |
| Núcleos de CPU | `n_jobs` | `n_jobs` | `thread_count` |
| Semilla aleatoria | `random_state` | `random_state` | `random_seed` |

## Estrategia de crecimiento — resumen comparativo

Ya cubierto en detalle en cada archivo individual — resumen rápido:

| | XGBoost | LightGBM | CatBoost |
|---|---|---|---|
| Estrategia | Level-wise (default) | Leaf-wise | Symmetric (oblivious) |
| Riesgo de overfitting | Moderado | Alto sin regular `num_leaves` | Bajo |
| Velocidad de inferencia | Buena | Buena | **Muy alta** (árboles simétricos, vectorizable) |

## Manejo de categóricas — resumen comparativo

| | XGBoost | LightGBM | CatBoost |
|---|---|---|---|
| Requiere encoding previo | Sí (o `enable_categorical`, experimental) | Parcial (`dtype="category"` o `categorical_feature`) | **No — nativo y automático** |
| Método interno | N/A (depende del encoding externo) | Partición óptima de categorías por gradiente | Ordered Target Encoding |
| Riesgo de leakage en el encoding | Depende de cómo se haga el encoding externo | Bajo | **Mínimo, por diseño (Ordered)** |

## Tuning con Optuna — patrón unificado para las tres

El patrón de `nested=True`/`objective` cubierto en `Optuna/01 - Introducción y Conceptos Fundamentales.md` aplica igual a las tres, cambiando solo el espacio de hiperparámetros:

```python
import optuna
from sklearn.model_selection import cross_val_score

def objective_xgboost(trial):
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 800),
        "max_depth": trial.suggest_int("max_depth", 3, 10),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
        "subsample": trial.suggest_float("subsample", 0.6, 1.0),
        "colsample_bytree": trial.suggest_float("colsample_bytree", 0.6, 1.0),
        "reg_lambda": trial.suggest_float("reg_lambda", 1e-3, 10.0, log=True),
    }
    modelo = XGBRegressor(**params, random_state=42)
    scores = cross_val_score(modelo, X_train, y_train, cv=5, scoring="neg_mean_absolute_error")
    return -scores.mean()

def objective_lightgbm(trial):
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 800),
        "num_leaves": trial.suggest_int("num_leaves", 15, 255, log=True),   # CENTRAL en LightGBM
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
        "feature_fraction": trial.suggest_float("feature_fraction", 0.6, 1.0),
        "bagging_fraction": trial.suggest_float("bagging_fraction", 0.6, 1.0),
        "lambda_l2": trial.suggest_float("lambda_l2", 1e-3, 10.0, log=True),
    }
    modelo = LGBMRegressor(**params, random_state=42, verbosity=-1)
    scores = cross_val_score(modelo, X_train, y_train, cv=5, scoring="neg_mean_absolute_error")
    return -scores.mean()

def objective_catboost(trial):
    params = {
        "iterations": trial.suggest_int("iterations", 200, 800),
        "depth": trial.suggest_int("depth", 4, 10),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
        "l2_leaf_reg": trial.suggest_float("l2_leaf_reg", 1.0, 10.0, log=True),
    }
    modelo = CatBoostRegressor(**params, cat_features=columnas_categoricas, random_seed=42, verbose=0)
    scores = cross_val_score(modelo, X_train, y_train, cv=5, scoring="neg_mean_absolute_error")
    return -scores.mean()
```

Ver `Optuna/08 - Integraciones con Frameworks de ML.md` para los callbacks de pruning específicos de cada librería (`XGBoostPruningCallback`, y equivalentes disponibles para LightGBM/CatBoost).

## Torneo entre las tres — patrón de comparación honesta

```python
from sklearn.model_selection import cross_val_score

modelos = {
    "XGBoost": XGBRegressor(n_estimators=300, max_depth=6, learning_rate=0.05, random_state=42),
    "LightGBM": LGBMRegressor(n_estimators=300, num_leaves=31, learning_rate=0.05, random_state=42, verbosity=-1),
    "CatBoost": CatBoostRegressor(iterations=300, depth=6, learning_rate=0.05, cat_features=columnas_categoricas, random_seed=42, verbose=0),
}

resultados = {}
for nombre, modelo in modelos.items():
    scores = cross_val_score(modelo, X_train, y_train, cv=5, scoring="neg_mean_absolute_error")
    resultados[nombre] = -scores.mean()
    print(f"{nombre}: MAE = {resultados[nombre]:.3f} (+/- {scores.std():.3f})")
```

> **Advertencia de comparación justa**: cada librería tiene defaults distintos y sensibilidad distinta al tuning — comparar con hiperparámetros "de fábrica" sin ajustar puede favorecer artificialmente a CatBoost (más robusto sin tuning) sobre LightGBM (que casi siempre necesita algo de ajuste para no sobreajustar). Para una comparación rigurosa, cada modelo debería pasar por su propia búsqueda de hiperparámetros (vía Optuna) antes de comparar el mejor resultado de cada una.

## Cuándo el ganador "depende del dataset" — no hay una respuesta universal

En benchmarks públicos y en la práctica, las diferencias de precisión entre las tres, **con tuning adecuado**, suelen ser pequeñas (a menudo dentro del margen de error de la validación cruzada) — la elección final en un proyecto real termina dependiendo más de:

1. Cuántas columnas categóricas de alta cardinalidad tiene el dataset (favorece CatBoost).
2. El tamaño del dataset y las restricciones de tiempo/memoria de entrenamiento (favorece LightGBM en datasets muy grandes).
3. La madurez del ecosistema de herramientas alrededor (XGBoost tiene la integración más probada con la mayoría de plataformas MLOps).
4. La familiaridad del equipo con el tuning de cada una (CatBoost requiere menos experiencia previa para obtener buenos resultados).

## Ver también

- [[03 - XGBoost - API Scikit-Learn Compatible e Hiperparámetros]]
- [[05 - LightGBM - API Scikit-Learn Compatible y Categóricas Nativas]]
- [[07 - CatBoost - API, Pool y Funcionalidades Distintivas]]
- `Optuna/08 - Integraciones con Frameworks de ML.md`
