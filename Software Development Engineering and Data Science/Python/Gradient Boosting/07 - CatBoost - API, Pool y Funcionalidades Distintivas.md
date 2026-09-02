---
tags: [gradient-boosting, catboost, cheat-sheet]
---

# 07 — CatBoost: API, Pool y Funcionalidades Distintivas

> Continúa de [[06 - CatBoost - Ordered Boosting y Manejo de Categóricas]].

## `Pool` — la estructura de datos interna de CatBoost

```python
from catboost import Pool

train_pool = Pool(
    data=X_train,
    label=y_train,
    cat_features=["region", "tipo_producto"],
    weight=pesos_muestra,   # opcional
)
val_pool = Pool(data=X_val, label=y_val, cat_features=["region", "tipo_producto"])
```

Análogo a `DMatrix` en XGBoost o `Dataset` en LightGBM — encapsula datos, etiquetas, pesos y la declaración de columnas categóricas en un solo objeto optimizado para el entrenamiento.

## `CatBoostRegressor` / `CatBoostClassifier` — API principal (ya compatible con sklearn)

```python
from catboost import CatBoostRegressor

modelo = CatBoostRegressor(
    iterations=500,          # equivalente a n_estimators
    depth=6,                   # profundidad del árbol simétrico
    learning_rate=0.05,
    l2_leaf_reg=3.0,            # regularización L2 sobre los valores de las hojas
    cat_features=["region", "tipo_producto"],
    loss_function="RMSE",       # "MAE", "Quantile", "Logloss", "MultiClass", etc.
    random_seed=42,
    verbose=100,                # imprime progreso cada 100 iteraciones (0 para silenciar)
)
modelo.fit(X_train, y_train, eval_set=(X_val, y_val))
```

A diferencia de XGBoost/LightGBM (que tienen una API nativa separada del wrapper sklearn-compatible), en CatBoost **`CatBoostRegressor`/`CatBoostClassifier` ya es la API principal recomendada** — es simultáneamente compatible con el contrato de scikit-learn (`fit`/`predict`/`get_params`) y la forma estándar de usar la librería, sin necesitar una API "nativa" separada como `xgb.train`/`lgb.train`.

## Hiperparámetros clave

| Hiperparámetro | Qué controla | Equivalente en XGBoost |
|---|---|---|
| `iterations` | Número de árboles | `n_estimators` |
| `depth` | Profundidad del árbol simétrico | `max_depth` |
| `learning_rate` | Peso de cada corrección | `learning_rate` |
| `l2_leaf_reg` | Regularización L2 | `reg_lambda` |
| `random_strength` | Aleatoriedad en la selección de splits (regularización adicional) | Sin equivalente directo |
| `bagging_temperature` | Intensidad del bagging Bayesiano (por defecto) | `subsample` (concepto similar) |
| `border_count` | Número de bins para discretizar features numéricas | Similar a `max_bin` en LightGBM |

## `early_stopping_rounds`

```python
modelo = CatBoostRegressor(iterations=2000, learning_rate=0.03, early_stopping_rounds=20)
modelo.fit(X_train, y_train, eval_set=(X_val, y_val), verbose=100)

print(modelo.get_best_iteration())
```

## Text Features — manejo nativo de columnas de texto libre

```python
modelo = CatBoostRegressor(
    iterations=500,
    cat_features=["region"],
    text_features=["descripcion_producto"],   # columnas de texto LIBRE, no categorías discretas
)
modelo.fit(X_train, y_train)
```

Funcionalidad distintiva sin equivalente directo en XGBoost/LightGBM: CatBoost puede procesar columnas de texto libre (no solo categorías discretas) generando automáticamente features basadas en n-gramas y estadísticas del texto — útil para datasets de negocio con campos como descripciones de producto o comentarios de clientes, sin necesitar un pipeline de NLP separado para esas columnas específicas.

## Embeddings — features vectoriales precalculadas

```python
modelo = CatBoostRegressor(
    iterations=500,
    embedding_features=["embedding_producto"],   # columna cuyo valor es un vector (ej. de un modelo de NLP externo)
)
```

Permite pasar directamente vectores de embeddings (por ejemplo, generados por un modelo de lenguaje) como una columna más del dataset, sin tener que expandirlos manualmente en columnas individuales.

## `select_features` — selección de features integrada

```python
resultado = modelo.select_features(
    train_pool,
    eval_set=val_pool,
    features_for_select=list(X_train.columns),
    num_features_to_select=15,
    algorithm="RecursiveByShapValues",   # usa importancia SHAP internamente para decidir qué eliminar
)
print(resultado["selected_features_names"])
```

Funcionalidad de selección de features integrada directamente en la librería, usando SHAP internamente — evita tener que orquestar manualmente un `RFECV` de scikit-learn (ver `Scikit-learn/10 - Selección de Features.md`) cuando ya se está trabajando con CatBoost.

## Herramientas de análisis de modelo integradas

```python
# Importancia de features (varias formas, incluyendo SHAP nativo):
modelo.get_feature_importance(train_pool, type="FeatureImportance")
modelo.get_feature_importance(train_pool, type="ShapValues")

# Comparar dos modelos entrenados en el mismo dataset:
from catboost import CatBoostRegressor
modelo.compare(otro_modelo, val_pool, metrics=["RMSE", "MAE"])

# Detección de overfitting integrada — grafica train vs. validation por iteración:
modelo.get_evals_result()
```

CatBoost incluye más herramientas de diagnóstico "listas para usar" sin dependencias externas que XGBoost/LightGBM (que típicamente requieren `matplotlib`/`shap` por separado para lo mismo) — parte de su filosofía de minimizar fricción operativa.

## Guardar y cargar modelos

```python
modelo.save_model("modelo_catboost.cbm")

modelo_cargado = CatBoostRegressor()
modelo_cargado.load_model("modelo_catboost.cbm")
```

## Integración en Pipeline de scikit-learn

```python
from sklearn.pipeline import Pipeline

# CatBoost maneja las categóricas internamente — el ColumnTransformer solo necesita imputación, no encoding
pipeline = Pipeline([
    ("imputer_numerico", SimpleImputer(strategy="median")),   # aplicado solo a columnas numéricas vía ColumnTransformer
    ("modelo", CatBoostRegressor(iterations=500, cat_features=columnas_categoricas, verbose=0)),
])
```

> **Precaución con `cat_features` e índices dentro de un Pipeline**: `cat_features` en CatBoost puede referenciar columnas por **nombre** (si se usa un DataFrame de pandas) o por **índice posicional** (si se usa un array de NumPy) — dentro de un `ColumnTransformer` que reordena columnas, es fácil que los índices posicionales dejen de corresponder a las columnas categóricas esperadas. Verificar explícitamente el orden final de columnas que llega al paso de CatBoost antes de asumir que `cat_features` sigue siendo correcto tras el preprocesamiento.

## Ver también

- [[06 - CatBoost - Ordered Boosting y Manejo de Categóricas]]
- [[10 - Interpretabilidad e Importancia de Features]]
- `Scikit-learn/10 - Selección de Features.md`
