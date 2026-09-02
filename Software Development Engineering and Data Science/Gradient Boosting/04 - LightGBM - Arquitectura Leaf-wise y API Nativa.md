---
tags: [gradient-boosting, lightgbm, cheat-sheet]
---

# 04 — LightGBM: Arquitectura Leaf-wise y API Nativa

> Continúa de [[01 - Introducción y Panorama]].

## La diferencia arquitectónica central: Leaf-wise vs. Level-wise

Esta es la decisión de diseño más importante que distingue a LightGBM del resto.

```
Level-wise (XGBoost por defecto, CatBoost, árboles clásicos):
       [raíz]
      /      \
   [A]        [B]         ← el árbol crece nivel completo por nivel completo
   / \        / \
 [C] [D]    [E] [F]

Leaf-wise (LightGBM):
       [raíz]
      /      \
   [A]        [B]
   / \
 [C] [D]                   ← solo se expande la hoja con MAYOR reducción de pérdida,
                              sin importar en qué "nivel" esté
```

- **Level-wise**: expande todos los nodos de un nivel antes de pasar al siguiente — más "balanceado", pero puede gastar cómputo expandiendo hojas que no aportan mucha reducción de error.
- **Leaf-wise**: en cada paso, identifica la **única hoja** cuya expansión reduciría más la pérdida, y solo expande esa — converge más rápido hacia un buen ajuste con el mismo número de hojas totales, pero puede producir árboles muy asimétricos y profundos en una sola rama, lo que aumenta el riesgo de overfitting en datasets pequeños.

```python
import lightgbm as lgb

params = {
    "num_leaves": 31,   # el hiperparámetro CENTRAL en LightGBM — controla directamente la complejidad del árbol
    "max_depth": -1,    # -1 = sin límite explícito de profundidad (num_leaves ya regula la complejidad)
}
```

> **La regla de tuning más importante de LightGBM**: `num_leaves` es el hiperparámetro de complejidad principal (no `max_depth`, que en LightGBM es secundario y a menudo se deja en `-1`). Como referencia, `num_leaves` debería ser menor a `2^max_depth` si se combina con un límite de profundidad — un valor de `num_leaves` demasiado alto relativo al tamaño del dataset es la causa más común de overfitting severo en LightGBM.

## Las dos optimizaciones que hacen a LightGBM más rápido

### GOSS (Gradient-based One-Side Sampling)

En vez de usar todas las muestras para calcular cada split (como XGBoost/CatBoost por defecto), GOSS conserva todas las muestras con gradiente grande (las que el modelo predice peor — más informativas) y **muestrea aleatoriamente** solo una fracción de las muestras con gradiente pequeño (las que ya está prediciendo bien) — reduce el cómputo sin perder mucha precisión, porque las muestras "fáciles" aportan poca información nueva a cada split.

```python
params = {"boosting": "goss", "top_rate": 0.2, "other_rate": 0.1}
```

### EFB (Exclusive Feature Bundling)

Agrupa automáticamente features dispersas y mutuamente excluyentes (típico de datos con muchas columnas one-hot-encoded) en una sola "feature combinada" — reduce la dimensionalidad efectiva sin perder información, porque esas features casi nunca son distintas de cero simultáneamente. Es una de las razones por las que LightGBM maneja datasets con muchas columnas dummy mejor que otras implementaciones.

## `Dataset` — la estructura de datos interna de LightGBM

```python
import lightgbm as lgb

dtrain = lgb.Dataset(X_train, label=y_train, feature_name=list(X_train.columns))
dval = lgb.Dataset(X_val, label=y_val, reference=dtrain)   # reference: comparte el binning de dtrain
```

`reference=dtrain` es importante: garantiza que la discretización de histogramas (binning) usada para validación sea **exactamente la misma** que la aprendida sobre entrenamiento, evitando inconsistencias sutiles entre ambos conjuntos.

## `lgb.train` — entrenamiento con la API nativa

```python
params = {
    "objective": "regression",       # "binary", "multiclass", "regression", "regression_l1" (MAE), etc.
    "metric": "mae",
    "num_leaves": 31,
    "learning_rate": 0.05,
    "feature_fraction": 0.8,          # equivalente a colsample_bytree de XGBoost
    "bagging_fraction": 0.8,           # equivalente a subsample
    "bagging_freq": 5,                  # cada cuántas iteraciones se re-samplea el bagging
    "verbosity": -1,
}

modelo = lgb.train(
    params,
    dtrain,
    num_boost_round=500,
    valid_sets=[dtrain, dval],
    valid_names=["train", "validation"],
    callbacks=[lgb.early_stopping(stopping_rounds=20), lgb.log_evaluation(period=50)],
)

print(modelo.best_iteration)
```

Nótese el patrón de `callbacks` — desde LightGBM 3.x/4.x, `early_stopping` y `log_evaluation` se configuran como callbacks explícitos en vez de argumentos directos de `train()`, a diferencia del patrón más simple de XGBoost.

## `lgb.cv` — cross-validation nativa

```python
resultado_cv = lgb.cv(
    params, dtrain,
    num_boost_round=500,
    nfold=5,
    callbacks=[lgb.early_stopping(stopping_rounds=20)],
    seed=42,
)
print(min(resultado_cv["valid mae-mean"]))
```

## Manejo nativo de columnas categóricas — API nativa

```python
dtrain = lgb.Dataset(
    X_train, label=y_train,
    categorical_feature=["region", "tipo_producto"],   # nombres o índices de columnas
)
```

LightGBM encuentra internamente la partición óptima de categorías (agrupa categorías similares respecto al gradiente) en vez de tratar cada categoría como un entero ordinal arbitrario — más sofisticado que un `OrdinalEncoder` simple, aunque menos automático que el enfoque de CatBoost (ver [[06 - CatBoost - Ordered Boosting y Manejo de Categóricas]]). Requiere que las columnas categóricas estén codificadas como enteros no negativos de antemano (no acepta strings directamente en la API nativa).

## Guardar y cargar modelos — API nativa

```python
modelo.save_model("modelo_lightgbm.txt")

modelo_cargado = lgb.Booster(model_file="modelo_lightgbm.txt")
```

## Importancia de features — API nativa

```python
importancia = modelo.feature_importance(importance_type="gain")   # o "split" (equivalente a "weight" en XGBoost)

lgb.plot_importance(modelo, importance_type="gain", max_num_features=15)
```

## Ver también

- [[05 - LightGBM - API Scikit-Learn Compatible y Categóricas Nativas]]
- [[02 - XGBoost - API Nativa y DMatrix]] (comparar enfoques)
- [[08 - Comparativa Técnica y Tuning Cruzado]]
