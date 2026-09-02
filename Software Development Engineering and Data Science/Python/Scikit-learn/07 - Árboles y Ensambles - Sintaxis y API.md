---
tags: [scikit-learn, machine-learning, arboles, ensambles, cheat-sheet]
---

# 07 — Árboles y Ensambles: Sintaxis y API

> Complementa la teoría de [[35-Arboles-de-Decision-en-Profundidad]] y [[36-Ensambles-en-Profundidad]] (en `Machine Learning/`) con la API concreta de `sklearn.tree` y `sklearn.ensemble`.

## `DecisionTreeClassifier` / `DecisionTreeRegressor`

```python
from sklearn.tree import DecisionTreeClassifier, plot_tree

modelo = DecisionTreeClassifier(
    max_depth=5,               # profundidad máxima — el hiperparámetro de regularización más directo
    min_samples_split=20,       # mínimo de muestras para considerar dividir un nodo
    min_samples_leaf=10,        # mínimo de muestras que debe tener cada hoja resultante
    criterion="gini",           # "gini" o "entropy" para clasificación; "squared_error" para regresión
    class_weight="balanced",
    random_state=42,
)
modelo.fit(X_train, y_train)

modelo.feature_importances_   # importancia de cada feature (basada en reducción de impureza)
modelo.tree_.max_depth         # profundidad real alcanzada (puede ser menor al max_depth configurado)
```

### Visualizar el árbol directamente

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(20, 10))
plot_tree(modelo, feature_names=X_train.columns, class_names=["no", "si"], filled=True, max_depth=3)
plt.show()

# Alternativa en texto plano, útil fuera de notebooks:
from sklearn.tree import export_text
print(export_text(modelo, feature_names=list(X_train.columns)))
```

## `RandomForestClassifier` / `RandomForestRegressor`

```python
from sklearn.ensemble import RandomForestRegressor

modelo = RandomForestRegressor(
    n_estimators=300,           # número de árboles — más árboles rara vez perjudica, solo cuesta tiempo
    max_depth=None,               # None = árboles crecen hasta pureza total (regularizado por otros params)
    max_features="sqrt",          # features consideradas por split — "sqrt" (clasificación), 1.0 (regresión)
    min_samples_leaf=5,
    bootstrap=True,               # muestreo con reemplazo — el "bagging" del Random Forest
    oob_score=True,               # evalúa con las muestras NO usadas en cada árbol (out-of-bag)
    n_jobs=-1,
    random_state=42,
)
modelo.fit(X_train, y_train)

modelo.oob_score_               # score sobre muestras out-of-bag — validación "gratis" sin split adicional
modelo.feature_importances_
```

`oob_score=True` es una particularidad valiosa de los métodos de bagging: cada árbol se entrena con una muestra bootstrap (con reemplazo) de los datos, dejando ~37% de las muestras fuera de cada árbol — esas muestras "out-of-bag" sirven como validación interna sin necesidad de reservar un conjunto de validación separado.

## `GradientBoostingClassifier` / `GradientBoostingRegressor`

```python
from sklearn.ensemble import GradientBoostingRegressor

modelo = GradientBoostingRegressor(
    n_estimators=300,
    learning_rate=0.05,          # tasa de aprendizaje — más bajo requiere más n_estimators, pero generaliza mejor
    max_depth=3,                   # árboles DÉBILES por diseño — profundidad baja a propósito
    subsample=0.8,                 # fracción de muestras usadas por árbol (Stochastic Gradient Boosting)
    validation_fraction=0.1,       # fracción reservada para early stopping
    n_iter_no_change=10,            # detiene si no mejora en 10 iteraciones consecutivas
    random_state=42,
)
modelo.fit(X_train, y_train)
```

Implementación "clásica" de Gradient Boosting en scikit-learn — más lenta que XGBoost/LightGBM en datasets grandes porque no tiene las mismas optimizaciones de histograma. Para datasets grandes, preferir `HistGradientBoosting*` (siguiente sección) o las librerías externas XGBoost/LightGBM (ver [[15 - Integraciones con el Ecosistema]]).

## `HistGradientBoostingClassifier` / `Regressor` — la versión rápida, nativa

```python
from sklearn.ensemble import HistGradientBoostingRegressor

modelo = HistGradientBoostingRegressor(
    max_iter=300,                  # equivalente a n_estimators
    learning_rate=0.05,
    max_depth=None,
    max_leaf_nodes=31,
    l2_regularization=0.0,
    early_stopping="auto",          # detiene automáticamente si el dataset es grande
    categorical_features="from_dtype",   # soporta columnas categóricas NATIVAMENTE, sin one-hot previo
)
modelo.fit(X_train, y_train)
```

Inspirado directamente en LightGBM: usa *histogram binning* para acelerar drásticamente el entrenamiento en datasets grandes, y soporta valores faltantes y columnas categóricas de forma nativa (sin necesidad de imputar o one-hot-encodear antes) — la opción recomendada dentro de scikit-learn puro cuando XGBoost/LightGBM no están disponibles como dependencia.

## `AdaBoostClassifier` / `AdaBoostRegressor`

```python
from sklearn.ensemble import AdaBoostClassifier
from sklearn.tree import DecisionTreeClassifier

modelo = AdaBoostClassifier(
    estimator=DecisionTreeClassifier(max_depth=1),   # "stumps" — árboles de un solo split
    n_estimators=200,
    learning_rate=1.0,
)
modelo.fit(X_train, y_train)
```

Boosting clásico anterior a Gradient Boosting — en cada iteración, aumenta el peso de las muestras mal clasificadas por el modelo anterior, en vez de ajustar sobre los residuos como hace Gradient Boosting. Menos usado en producción moderna, pero sigue apareciendo como baseline o en contextos educativos.

## `BaggingClassifier` / `BaggingRegressor` — bagging genérico con cualquier estimador base

```python
from sklearn.ensemble import BaggingClassifier
from sklearn.neighbors import KNeighborsClassifier

modelo = BaggingClassifier(
    estimator=KNeighborsClassifier(),   # Random Forest es, conceptualmente, Bagging + árboles específicamente
    n_estimators=50,
    max_samples=0.8,
    max_features=0.8,
    bootstrap=True,
    n_jobs=-1,
)
```

Generaliza la idea de Random Forest a **cualquier** estimador base, no solo árboles — útil cuando se quiere reducir varianza de un modelo inestable distinto a un árbol (por ejemplo, KNN, que también es sensible a variaciones en los datos de entrenamiento).

## `VotingClassifier` / `VotingRegressor` — combinar modelos heterogéneos

```python
from sklearn.ensemble import VotingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC

modelo = VotingClassifier(
    estimators=[
        ("lr", LogisticRegression()),
        ("rf", RandomForestClassifier(n_estimators=200)),
        ("svc", SVC(probability=True)),
    ],
    voting="soft",   # "soft"=promedia probabilidades; "hard"=voto mayoritario de clases predichas
    weights=[1, 2, 1],   # peso relativo de cada modelo en el promedio (opcional)
)
modelo.fit(X_train, y_train)
```

`voting="soft"` requiere que todos los estimadores soporten `predict_proba` (por eso `SVC(probability=True)` explícito, ya que por defecto SVC no calcula probabilidades). Combina modelos de **familias distintas** — la diversidad entre ellos es lo que reduce el error de forma más efectiva que combinar variantes del mismo algoritmo.

## `StackingClassifier` / `StackingRegressor` — un meta-modelo aprende a combinar

```python
from sklearn.ensemble import StackingRegressor
from sklearn.linear_model import Ridge

modelo = StackingRegressor(
    estimators=[
        ("rf", RandomForestRegressor(n_estimators=200)),
        ("gb", GradientBoostingRegressor(n_estimators=200)),
    ],
    final_estimator=Ridge(alpha=1.0),   # el "meta-modelo" que aprende a combinar las predicciones base
    cv=5,   # las predicciones de los modelos base para entrenar el meta-modelo se generan OUT-OF-FOLD
)
modelo.fit(X_train, y_train)
```

Más sofisticado que `VotingClassifier`: en vez de un promedio fijo (simple o ponderado manualmente), un `final_estimator` **aprende** la mejor forma de combinar las predicciones de los modelos base. El `cv` interno es crítico — genera las predicciones base vía out-of-fold (similar a `cross_val_predict`, ver [[04 - Model Selection - Validación y Búsqueda]]) para que el meta-modelo no vea predicciones "contaminadas" por haber sido generadas sobre datos que esos mismos modelos ya vieron en entrenamiento.

## `feature_importances_` vs. Permutation Importance

```python
# Importancia basada en impureza (rápida, pero sesgada hacia features de alta cardinalidad)
modelo.feature_importances_

# Permutation Importance — más confiable, mide el efecto real de "romper" cada feature
from sklearn.inspection import permutation_importance

resultado = permutation_importance(modelo, X_test, y_test, n_repeats=10, random_state=42, n_jobs=-1)
importancias = pd.Series(resultado.importances_mean, index=X_test.columns).sort_values(ascending=False)
```

`feature_importances_` nativo de árboles/ensambles tiene un sesgo conocido: sobreestima la importancia de features numéricas continuas o categóricas de alta cardinalidad, simplemente porque tienen más puntos de corte posibles para el árbol. `permutation_importance` es más robusto porque mide directamente cuánto empeora el modelo al permutar aleatoriamente una feature — ver también `Machine Learning/39-Interpretabilidad-de-Modelos.md`.

## Ver también

- [[35-Arboles-de-Decision-en-Profundidad]] y [[36-Ensambles-en-Profundidad]] (en `Machine Learning/`)
- [[04 - Model Selection - Validación y Búsqueda]]
- [[15 - Integraciones con el Ecosistema]] (XGBoost/LightGBM)
- `Machine Learning/39-Interpretabilidad-de-Modelos.md`
