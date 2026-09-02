---
tags: [scikit-learn, machine-learning, feature-selection, cheat-sheet]
---

# 10 — Selección de Features

> Continúa de [[09 - Clustering y Reducción de Dimensionalidad]]. Complementa `Machine Learning/40-Feature-Engineering-Avanzado.md`.

El módulo `sklearn.feature_selection` reduce el número de features **conservando las originales** (a diferencia de PCA, que las combina en componentes nuevos) — importante cuando la interpretabilidad de qué feature específica importa es un requisito.

## `VarianceThreshold` — eliminar features casi constantes

```python
from sklearn.feature_selection import VarianceThreshold

selector = VarianceThreshold(threshold=0.01)   # elimina features cuya varianza es menor al umbral
X_filtrado = selector.fit_transform(X_train)

print(selector.get_support())   # array booleano de qué columnas sobrevivieron
```

El filtro más simple posible — una feature con varianza casi cero (por ejemplo, una columna que vale `0` en el 99.9% de las filas) casi no aporta información discriminativa, sin importar su relación con `y`. Útil como primer paso barato antes de métodos más costosos.

## `SelectKBest` / `SelectPercentile` — filtrado univariado

```python
from sklearn.feature_selection import SelectKBest, f_regression, f_classif, mutual_info_regression, chi2

selector = SelectKBest(score_func=f_regression, k=10)   # las 10 features más correlacionadas con y
X_selected = selector.fit_transform(X_train, y_train)

print(selector.get_feature_names_out())
print(selector.scores_)   # score de cada feature individual
```

| `score_func` | Cuándo usarla |
|---|---|
| `f_regression` | Target continuo, relación lineal con cada feature |
| `f_classif` | Target categórico, features numéricas |
| `chi2` | Target categórico, features no negativas (ej. conteos, frecuencias) |
| `mutual_info_regression` / `mutual_info_classif` | Captura relaciones NO lineales, más costoso de calcular |

Estos métodos evalúan cada feature **de forma independiente** contra `y` — no consideran interacciones entre features ni redundancia entre ellas (dos features muy correlacionadas entre sí pueden ambas "pasar" el filtro aunque aporten información casi idéntica).

## `RFE` (Recursive Feature Elimination) — eliminación recursiva basada en un modelo

```python
from sklearn.feature_selection import RFE
from sklearn.ensemble import RandomForestRegressor

selector = RFE(
    estimator=RandomForestRegressor(n_estimators=100, random_state=42),
    n_features_to_select=10,
    step=1,   # cuántas features elimina en cada iteración
)
selector.fit(X_train, y_train)

print(selector.support_)     # qué features sobrevivieron
print(selector.ranking_)     # orden de eliminación (1 = seleccionada, valores mayores = eliminada antes)
```

A diferencia de los filtros univariados, `RFE` entrena el modelo repetidamente, elimina en cada ronda la(s) feature(s) menos importante(s) según `feature_importances_`/`coef_` del estimador, y repite — captura mejor las interacciones entre features porque evalúa el conjunto completo en cada paso, a costa de ser mucho más lento computacionalmente.

## `RFECV` — igual que RFE, pero eligiendo el número óptimo automáticamente

```python
from sklearn.feature_selection import RFECV

selector = RFECV(
    estimator=RandomForestRegressor(n_estimators=100, random_state=42),
    step=1,
    cv=5,
    scoring="neg_mean_absolute_error",
    min_features_to_select=3,
)
selector.fit(X_train, y_train)

print(selector.n_features_)   # el número ÓPTIMO encontrado vía cross-validation, no fijado a mano
```

Evita tener que adivinar `n_features_to_select` de antemano — prueba distintos tamaños de subconjunto vía cross-validation y elige el que da mejor score de validación.

## `SelectFromModel` — selección basada en importancia/coeficientes, sin recursión

```python
from sklearn.feature_selection import SelectFromModel
from sklearn.linear_model import Lasso

# Con un modelo que ya penaliza features irrelevantes (Lasso, árboles):
selector = SelectFromModel(
    estimator=Lasso(alpha=0.1),
    threshold="median",   # o un valor numérico explícito, o "mean"
)
selector.fit(X_train, y_train)
X_selected = selector.transform(X_train)
```

Más rápido que `RFE` (entrena el modelo **una sola vez**, no repetidamente) — simplemente toma el modelo ya ajustado y conserva las features cuya importancia/coeficiente supera el `threshold`. La elección natural cuando ya se está usando `Lasso` (que produce coeficientes exactamente en cero) o un modelo basado en árboles con `feature_importances_`.

## Integración con Pipeline — selección como un paso más

```python
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ("seleccion", SelectFromModel(RandomForestRegressor(n_estimators=100), threshold="median")),
    ("modelo", Ridge(alpha=1.0)),
])
pipeline.fit(X_train, y_train)
```

Como cualquier transformador, la selección de features se integra directamente en un `Pipeline` — esto es importante para evitar leakage: el criterio de qué features "importan" debe decidirse **solo con datos de entrenamiento**, igual que cualquier otro paso de preprocesamiento (ver [[16 - Buenas Prácticas, Errores Comunes y Rendimiento]]).

## Selección manual basada en correlación (diagnóstico, no automatizado)

```python
import pandas as pd

matriz_corr = X_train.corr().abs()
upper = matriz_corr.where(np.triu(np.ones(matriz_corr.shape), k=1).astype(bool))
columnas_redundantes = [col for col in upper.columns if any(upper[col] > 0.95)]
X_train_reducido = X_train.drop(columns=columnas_redundantes)
```

Paso de diagnóstico manual habitual antes de aplicar cualquier método automatizado — eliminar features casi perfectamente correlacionadas entre sí (redundancia pura) reduce el ruido para los métodos de selección posteriores y mejora directamente la interpretabilidad de modelos lineales.

## Tabla de decisión rápida

| Situación | Método |
|---|---|
| Eliminar features casi constantes, paso rápido inicial | `VarianceThreshold` |
| Filtro rápido, sin considerar interacciones | `SelectKBest` |
| Considerar interacciones entre features, presupuesto de cómputo disponible | `RFE` / `RFECV` |
| Ya usas Lasso/árboles, quieres aprovechar su importancia nativa | `SelectFromModel` |
| Diagnóstico de redundancia antes de cualquier método automático | Matriz de correlación manual |

## Ver también

- [[06 - Modelos Lineales - Sintaxis y API]] (Lasso como selector implícito)
- [[07 - Árboles y Ensambles - Sintaxis y API]] (feature_importances_)
- `Machine Learning/40-Feature-Engineering-Avanzado.md`
- `Machine Learning/39-Interpretabilidad-de-Modelos.md`
