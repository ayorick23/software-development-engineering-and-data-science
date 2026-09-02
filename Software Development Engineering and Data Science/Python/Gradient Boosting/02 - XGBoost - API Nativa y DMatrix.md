---
tags: [gradient-boosting, xgboost, cheat-sheet]
---

# 02 — XGBoost: API Nativa y DMatrix

> Continúa de [[01 - Introducción y Panorama]]. Para la teoría de Newton boosting y regularización interna, ver [[36-Ensambles-en-Profundidad]].

XGBoost tiene **dos APIs paralelas**: la API nativa (`xgb.train`, más antigua, más control de bajo nivel) y la API compatible con scikit-learn (`XGBClassifier`/`XGBRegressor`, ver [[03 - XGBoost - API Scikit-Learn Compatible e Hiperparámetros]]). Este archivo cubre la nativa — sigue siendo relevante porque expone funcionalidad (como `xgb.cv` con más control, o ciertas opciones de bajo nivel) que a veces no está igual de accesible en el wrapper de sklearn.

## `DMatrix` — la estructura de datos interna de XGBoost

```python
import xgboost as xgb

dtrain = xgb.DMatrix(X_train, label=y_train, feature_names=list(X_train.columns))
dval = xgb.DMatrix(X_val, label=y_val, feature_names=list(X_val.columns))
```

`DMatrix` es una estructura de datos optimizada internamente por XGBoost (maneja de forma eficiente valores faltantes, matrices dispersas, y pre-computa estadísticas usadas durante el entrenamiento) — convertir los datos a `DMatrix` antes de entrenar es lo que le da a la API nativa su ventaja de rendimiento sobre pasar arrays crudos directamente.

```python
# Con pesos por muestra (útil para dar más importancia a ciertas filas, ej. datos más recientes):
dtrain = xgb.DMatrix(X_train, label=y_train, weight=pesos_muestra)

# Con datos faltantes explícitos (por defecto XGBoost interpreta np.nan como faltante):
dtrain = xgb.DMatrix(X_train, label=y_train, missing=-999)
```

## El diccionario de parámetros — la forma nativa de configurar

```python
params = {
    "objective": "reg:squarederror",   # o "binary:logistic", "multi:softprob", etc.
    "max_depth": 6,
    "learning_rate": 0.05,
    "subsample": 0.8,
    "colsample_bytree": 0.8,
    "reg_alpha": 0.1,
    "reg_lambda": 1.0,
    "eval_metric": "mae",
    "tree_method": "hist",   # "hist" es el método moderno recomendado (histogram binning, inspirado en LightGBM)
}
```

### `objective` — funciones objetivo más comunes

| Objective | Uso |
|---|---|
| `reg:squarederror` | Regresión estándar (MSE) |
| `reg:absoluteerror` | Regresión robusta a outliers (MAE) |
| `reg:quantileerror` | Regresión de cuantiles |
| `binary:logistic` | Clasificación binaria, retorna probabilidad |
| `multi:softprob` | Clasificación multiclase, retorna probabilidades por clase |
| `rank:pairwise` | Ranking (aprendizaje a ordenar, ej. resultados de búsqueda) |

## `xgb.train` — entrenar con la API nativa

```python
modelo = xgb.train(
    params=params,
    dtrain=dtrain,
    num_boost_round=500,
    evals=[(dtrain, "train"), (dval, "validation")],
    early_stopping_rounds=20,
    verbose_eval=50,   # imprime progreso cada 50 rondas
)

print(modelo.best_iteration)   # ronda donde se detuvo (con early stopping activo)
print(modelo.best_score)
```

`evals` permite monitorear múltiples conjuntos simultáneamente durante el entrenamiento (típicamente train y validation) — la métrica de validation es la que se usa para decidir el early stopping.

## Predecir con un modelo entrenado vía API nativa

```python
dtest = xgb.DMatrix(X_test)
predicciones = modelo.predict(dtest)

# Predecir usando solo hasta la mejor ronda (si el entrenamiento continuó más allá del early stopping):
predicciones = modelo.predict(dtest, iteration_range=(0, modelo.best_iteration + 1))
```

## `xgb.cv` — cross-validation nativa, con más control que `cross_val_score`

```python
resultado_cv = xgb.cv(
    params=params,
    dtrain=dtrain,
    num_boost_round=500,
    nfold=5,
    early_stopping_rounds=20,
    metrics="mae",
    seed=42,
    as_pandas=True,
)

print(resultado_cv.tail())
mejor_num_rounds = resultado_cv["test-mae-mean"].idxmin() + 1
```

`xgb.cv` corre cross-validation con early stopping **integrado por fold** — cada fold detiene su entrenamiento independientemente cuando deja de mejorar, y el resultado devuelve la curva completa de la métrica en train y test por ronda, útil para diagnosticar overfitting directamente (comparando `train-mae-mean` vs. `test-mae-mean` a través de las rondas).

## Guardar y cargar modelos con la API nativa

```python
modelo.save_model("modelo_xgboost.json")   # formato JSON, recomendado — legible y estable entre versiones
modelo.save_model("modelo_xgboost.ubj")     # formato binario, más compacto

modelo_cargado = xgb.Booster()
modelo_cargado.load_model("modelo_xgboost.json")
```

> A diferencia de `joblib`/`pickle` (ver `Scikit-learn/13 - Persistencia de Modelos.md`), el formato nativo `.json`/`.ubj` de XGBoost es **estable entre versiones de la librería** — preferible para persistencia de modelos en producción de larga duración, donde `joblib` puede romper compatibilidad al actualizar XGBoost.

## Importancia de features — API nativa

```python
importancia = modelo.get_score(importance_type="gain")
# "weight": número de veces que la feature se usa en un split
# "gain": ganancia promedio de los splits que usan esa feature (la más informativa)
# "cover": número promedio de muestras afectadas por los splits de esa feature

xgb.plot_importance(modelo, importance_type="gain", max_num_features=15)
```

## Cuándo usar la API nativa en vez de la de sklearn

| Situación | API recomendada |
|---|---|
| Integración con `Pipeline`, `GridSearchCV`, `VotingClassifier` | sklearn-compatible (ver [[03 - XGBoost - API Scikit-Learn Compatible e Hiperparámetros]]) |
| Control fino de `xgb.cv`, callbacks de bajo nivel, múltiples `evals` simultáneos | API nativa |
| Consistencia de código con el resto de un proyecto basado en scikit-learn | sklearn-compatible |
| Rendimiento máximo evitando overhead de conversión repetida a `DMatrix` | API nativa |

En la práctica de proyectos que ya usan `Pipeline`/`ColumnTransformer` de scikit-learn, el wrapper sklearn-compatible es la opción por defecto — la API nativa se reserva para casos donde se necesita ese control adicional específico.

## Ver también

- [[03 - XGBoost - API Scikit-Learn Compatible e Hiperparámetros]]
- [[36-Ensambles-en-Profundidad]] (en `Machine Learning/`)
- [[08 - Comparativa Técnica y Tuning Cruzado]]
