---
tags: [scikit-learn, machine-learning, model-selection, cross-validation, cheat-sheet]
---

# 04 — Model Selection: Validación y Búsqueda

> Continúa de [[03 - Pipelines y ColumnTransformer]]. Para la disciplina de validación rigurosa en datos temporales, ver `Machine Learning/37-Validacion-Rigurosa-en-ML.md`.

El módulo `sklearn.model_selection` cubre tres necesidades: dividir datos, validar de forma robusta, y buscar hiperparámetros.

## `train_test_split` — la división más básica

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y,   # mantiene la proporción de clases igual en train y test (clasificación)
)
```

`stratify=y` es importante en clasificación con clases desbalanceadas — sin ella, una partición aleatoria puede dejar por azar muy pocos ejemplos de la clase minoritaria en test, haciendo la evaluación poco confiable.

## Estrategias de Cross-Validation — el iterador correcto según el problema

### `KFold` — el caso general

```python
from sklearn.model_selection import KFold

kf = KFold(n_splits=5, shuffle=True, random_state=42)
for train_idx, val_idx in kf.split(X):
    X_tr, X_val = X.iloc[train_idx], X.iloc[val_idx]
```

### `StratifiedKFold` — clasificación, preserva proporción de clases en cada fold

```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
for train_idx, val_idx in skf.split(X, y):
    ...
```

Usar siempre en clasificación en vez de `KFold` genérico — sin estratificar, un fold puede terminar por azar con una proporción de clases muy distinta al resto, sesgando la métrica de ese fold.

### `TimeSeriesSplit` — datos secuenciales, sin mezclar pasado y futuro

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5, gap=0)   # gap: días/pasos a saltar entre train y validation
for train_idx, val_idx in tscv.split(X):
    # train_idx SIEMPRE son índices anteriores a val_idx — nunca se mezclan
    ...
```

**Crítico para series de tiempo**: `KFold` estándar mezcla aleatoriamente pasado y futuro entre folds, lo que permite que el modelo "vea" información posterior al momento que está prediciendo — un leakage temporal severo. `TimeSeriesSplit` garantiza que cada fold de entrenamiento use solo datos anteriores al fold de validación correspondiente. Ver `Machine Learning/37-Validacion-Rigurosa-en-ML.md` para la discusión completa de walk-forward validation.

### `GroupKFold` — cuando las muestras no son independientes entre sí

```python
from sklearn.model_selection import GroupKFold

gkf = GroupKFold(n_splits=5)
for train_idx, val_idx in gkf.split(X, y, groups=oficina_id):
    # garantiza que la MISMA oficina nunca aparece en train Y en validation simultáneamente
    ...
```

Esencial cuando hay agrupaciones naturales en los datos (múltiples filas del mismo cliente, misma oficina, mismo paciente) — sin esto, el modelo puede "memorizar" patrones específicos de un grupo visto en train y parecer que generaliza bien en validation, cuando en realidad solo reconoce ese grupo específico.

### `LeaveOneOut` — validación exhaustiva para datasets muy pequeños

```python
from sklearn.model_selection import LeaveOneOut

loo = LeaveOneOut()   # cada fold deja UNA sola muestra fuera — n_splits = n_muestras
```

Computacionalmente costoso (entrena n modelos para n muestras), reservado para datasets pequeños donde cada muestra individual importa para la evaluación.

## `cross_val_score` — el atajo más común

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(
    pipeline, X_train, y_train,
    cv=StratifiedKFold(n_splits=5, shuffle=True, random_state=42),
    scoring="neg_mean_absolute_error",
)
print(f"MAE promedio: {-scores.mean():.3f} (+/- {scores.std():.3f})")
```

> Métricas de "error" (MAE, MSE, log loss) se pasan como **negativas** (`neg_mean_absolute_error`) porque la convención de scikit-learn es que un `scoring` siempre debe ser "mayor es mejor" — para errores, eso significa negarlos internamente.

## `cross_validate` — cuando se necesita más que un solo número

```python
from sklearn.model_selection import cross_validate

resultados = cross_validate(
    pipeline, X_train, y_train,
    cv=5,
    scoring=["neg_mean_absolute_error", "r2"],   # múltiples métricas a la vez
    return_train_score=True,    # compara desempeño en train vs. validation — detecta overfitting
    return_estimator=True,       # guarda cada modelo entrenado por fold, no solo el score
)

print(resultados["test_neg_mean_absolute_error"])
print(resultados["train_r2"])       # si train_score >> test_score, hay overfitting
print(resultados["fit_time"])       # tiempo de entrenamiento por fold
```

## `cross_val_predict` — predicciones out-of-fold

```python
from sklearn.model_selection import cross_val_predict

y_pred_oof = cross_val_predict(pipeline, X_train, y_train, cv=5)
# cada predicción viene de un modelo que NUNCA vio esa muestra durante su entrenamiento
```

Útil para generar predicciones "honestas" sobre todo el conjunto de entrenamiento (por ejemplo, para stacking de modelos, ver [[07 - Árboles y Ensambles - Sintaxis y API]]) sin el sesgo optimista de evaluar un modelo sobre datos que ya vio.

## `GridSearchCV` — búsqueda exhaustiva de hiperparámetros

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    "modelo__n_estimators": [100, 300, 500],
    "modelo__max_depth": [3, 6, 9],
}
# el prefijo "modelo__" apunta al paso "modelo" DENTRO del pipeline

grid = GridSearchCV(
    pipeline,
    param_grid=param_grid,
    cv=5,
    scoring="neg_mean_absolute_error",
    n_jobs=-1,
    refit=True,   # tras encontrar el mejor, re-entrena automáticamente con TODO el train set
)
grid.fit(X_train, y_train)

print(grid.best_params_)
print(grid.best_score_)
print(grid.best_estimator_)   # el pipeline completo ya re-entrenado con los mejores hiperparámetros
```

La sintaxis `paso__hiperparametro` (doble guión bajo) es la forma de referenciar hiperparámetros de un paso específico dentro de un `Pipeline` — indispensable cuando se busca hiperparámetros del modelo final sin tocar el preprocesamiento.

## `RandomizedSearchCV` — muestreo aleatorio del espacio

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

param_distributions = {
    "modelo__n_estimators": randint(100, 600),
    "modelo__max_depth": randint(3, 15),
    "modelo__learning_rate": uniform(0.01, 0.3),
}

random_search = RandomizedSearchCV(
    pipeline, param_distributions, n_iter=50, cv=5,
    scoring="neg_mean_absolute_error", random_state=42, n_jobs=-1,
)
random_search.fit(X_train, y_train)
```

Ver `Machine Learning/45-Optimizacion-de-Hiperparametros.md` y `Optuna/01 - Introducción y Conceptos Fundamentales.md` para alternativas de búsqueda bayesiana más eficientes que Grid/Random Search puro.

## `HalvingGridSearchCV` / `HalvingRandomSearchCV` — búsqueda con asignación adaptativa de recursos

```python
from sklearn.experimental import enable_halving_search_cv  # necesario, API aún experimental
from sklearn.model_selection import HalvingGridSearchCV

halving = HalvingGridSearchCV(
    pipeline, param_grid, cv=5,
    factor=3,           # en cada ronda, sobrevive 1/factor de las combinaciones
    resource="n_samples",  # el "recurso" que se incrementa entre rondas
)
halving.fit(X_train, y_train)
```

Implementa la misma idea de *Successive Halving* que Optuna (ver `Optuna/04 - Pruners en Profundidad.md`) pero integrada nativamente a la API de `GridSearchCV` — evalúa muchas combinaciones con pocos recursos primero, y da más recursos solo a las más prometedoras.

## `learning_curve` — diagnóstico de underfitting vs. overfitting

```python
from sklearn.model_selection import learning_curve
import numpy as np

train_sizes, train_scores, val_scores = learning_curve(
    pipeline, X_train, y_train,
    train_sizes=np.linspace(0.1, 1.0, 10),
    cv=5, scoring="neg_mean_absolute_error",
)
```

Si el score de entrenamiento y validación convergen a un valor bajo con poca separación entre ellos → underfitting (modelo demasiado simple). Si el score de entrenamiento es mucho mejor que el de validación y no convergen → overfitting (modelo demasiado complejo o falta de datos).

## `validation_curve` — efecto de un solo hiperparámetro

```python
from sklearn.model_selection import validation_curve

train_scores, val_scores = validation_curve(
    RandomForestRegressor(), X_train, y_train,
    param_name="max_depth", param_range=[2, 4, 6, 8, 10, 12],
    cv=5, scoring="neg_mean_absolute_error",
)
```

Similar a `learning_curve`, pero variando un hiperparámetro específico en vez de la cantidad de datos — permite visualizar directamente el punto donde un modelo pasa de underfitting a overfitting según ese parámetro.

## Ver también

- [[03 - Pipelines y ColumnTransformer]]
- [[05 - Métricas y Evaluación]]
- `Machine Learning/37-Validacion-Rigurosa-en-ML.md`
- `Optuna/01 - Introducción y Conceptos Fundamentales.md`
