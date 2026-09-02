---
tags: [scikit-learn, machine-learning, modelos-lineales, cheat-sheet]
---

# 06 — Modelos Lineales: Sintaxis y API

> Complementa la teoría matemática de [[34-Modelos-Lineales-en-Profundidad]] (en `Machine Learning/`) con la API concreta de `sklearn.linear_model`.

## `LinearRegression` — regresión sin regularización

```python
from sklearn.linear_model import LinearRegression

modelo = LinearRegression(
    fit_intercept=True,   # si False, fuerza a que la recta pase por el origen
    n_jobs=-1,
)
modelo.fit(X_train, y_train)

modelo.coef_        # coeficientes aprendidos, uno por feature
modelo.intercept_   # término independiente
```

## `Ridge` — regularización L2

```python
from sklearn.linear_model import Ridge, RidgeCV

modelo = Ridge(alpha=1.0, solver="auto")   # alpha: fuerza de regularización, mayor = más penalización
modelo.fit(X_train, y_train)

# RidgeCV busca el mejor alpha automáticamente vía cross-validation, sin GridSearchCV manual:
modelo_cv = RidgeCV(alphas=[0.01, 0.1, 1.0, 10.0, 100.0], cv=5)
modelo_cv.fit(X_train, y_train)
print(modelo_cv.alpha_)   # el alpha elegido
```

## `Lasso` — regularización L1, selección de features implícita

```python
from sklearn.linear_model import Lasso, LassoCV

modelo = Lasso(alpha=0.1, max_iter=10000)   # max_iter: Lasso usa optimización iterativa, puede no converger
modelo.fit(X_train, y_train)

print((modelo.coef_ == 0).sum())   # cuántos coeficientes quedaron exactamente en cero

modelo_cv = LassoCV(alphas=None, cv=5, max_iter=10000)   # alphas=None: LassoCV genera su propia grilla
modelo_cv.fit(X_train, y_train)
```

## `ElasticNet` — combinación de L1 y L2

```python
from sklearn.linear_model import ElasticNet, ElasticNetCV

modelo = ElasticNet(alpha=0.1, l1_ratio=0.5)   # l1_ratio: 0=Ridge puro, 1=Lasso puro, 0.5=mitad y mitad
modelo.fit(X_train, y_train)

modelo_cv = ElasticNetCV(l1_ratio=[0.1, 0.5, 0.7, 0.9, 1.0], cv=5)
modelo_cv.fit(X_train, y_train)
print(modelo_cv.alpha_, modelo_cv.l1_ratio_)
```

## `LogisticRegression` — clasificación lineal (a pesar del nombre)

```python
from sklearn.linear_model import LogisticRegression

modelo = LogisticRegression(
    penalty="l2",        # "l1", "l2", "elasticnet", "none"
    C=1.0,                 # INVERSO de la fuerza de regularización — menor C = MÁS regularización
    solver="lbfgs",        # "liblinear" para datasets pequeños/L1, "saga" soporta elasticnet
    max_iter=1000,
    class_weight="balanced",   # ajusta automáticamente pesos inversamente proporcionales a frecuencia de clase
    multi_class="auto",         # maneja multiclase automáticamente (one-vs-rest o multinomial según el solver)
)
modelo.fit(X_train, y_train)

modelo.predict(X_test)             # clases predichas
modelo.predict_proba(X_test)       # probabilidad de cada clase — [:, 1] para la clase positiva en binario
modelo.decision_function(X_test)   # distancia al hiperplano de decisión (antes de aplicar sigmoide/softmax)
```

> **Cuidado con `C`**: es lo opuesto a `alpha` en `Ridge`/`Lasso` — en `LogisticRegression`, `C` es el inverso de la regularización. `C` pequeño = mucha regularización; `C` grande = poca regularización. Es una de las inconsistencias históricas de la API que vale memorizar explícitamente.

### `class_weight` — manejar clases desbalanceadas sin resamplear datos

```python
modelo = LogisticRegression(class_weight="balanced")
# equivale a pesar cada clase inversamente proporcional a su frecuencia:
# peso_clase = n_muestras / (n_clases * n_muestras_de_esa_clase)

modelo = LogisticRegression(class_weight={0: 1, 1: 5})   # peso manual explícito por clase
```

Alternativa a técnicas de resampling como SMOTE (ver [[11 - Datos Faltantes y Clases Desbalanceadas]]) — en vez de generar/eliminar muestras, penaliza más los errores sobre la clase minoritaria directamente en la función de pérdida.

## `SGDClassifier` / `SGDRegressor` — modelos lineales vía descenso de gradiente estocástico

```python
from sklearn.linear_model import SGDClassifier, SGDRegressor

modelo = SGDClassifier(
    loss="log_loss",        # "log_loss"=regresión logística, "hinge"=SVM lineal, "modified_huber", etc.
    penalty="l2",
    alpha=0.0001,
    learning_rate="optimal",
    max_iter=1000,
)
modelo.fit(X_train, y_train)

# partial_fit: entrenamiento incremental, útil para datasets que no caben en memoria
modelo.partial_fit(X_batch, y_batch, classes=[0, 1])
```

La ventaja central de `SGD*` sobre `LogisticRegression`/`Ridge` estándar: escala a datasets muy grandes (millones de filas) porque procesa los datos en lotes/muestra por muestra en vez de requerir toda la matriz en memoria para una solución cerrada — y soporta **aprendizaje incremental** vía `partial_fit()`, útil para streams de datos o reentrenamiento sin recargar todo el histórico.

## `PolynomialFeatures` — capturar no linealidad con un modelo lineal

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import make_pipeline

modelo = make_pipeline(
    PolynomialFeatures(degree=2, include_bias=False),   # genera x1², x2², x1*x2, además de x1, x2
    StandardScaler(),
    Ridge(alpha=1.0),
)
modelo.fit(X_train, y_train)
```

Genera features polinómicas e interacciones antes de aplicar el modelo lineal — permite que `Ridge`/`Lasso` capturen relaciones no lineales simples sin cambiar de familia de algoritmo. El número de features crece rápidamente con `degree` y el número de columnas originales — casi siempre requiere regularización (`Ridge`/`Lasso`, no `LinearRegression` sin penalización) para evitar overfitting severo.

## Interpretación de coeficientes — con y sin escalado

```python
import pandas as pd

pipeline = make_pipeline(StandardScaler(), Ridge(alpha=1.0))
pipeline.fit(X_train, y_train)

coeficientes = pd.Series(
    pipeline.named_steps["ridge"].coef_,
    index=X_train.columns,
).sort_values(key=abs, ascending=False)
print(coeficientes)
```

Los coeficientes solo son directamente comparables en magnitud entre sí **si las features fueron escaladas primero** — sin escalar, un coeficiente pequeño puede corresponder a una feature con escala grande (y por tanto igual de importante), lo que hace la comparación cruda entre coeficientes engañosa.

## Tabla resumen — cuándo cada uno

| Situación | Modelo |
|---|---|
| Regresión, sin necesidad de regularización, pocas features | `LinearRegression` |
| Regresión, features correlacionadas, quieres encoger sin eliminar | `Ridge` |
| Regresión, quieres selección automática de features | `Lasso` |
| Regresión, features correlacionadas Y quieres selección | `ElasticNet` |
| Clasificación lineal estándar | `LogisticRegression` |
| Dataset muy grande (no cabe en memoria) o streaming | `SGDClassifier`/`SGDRegressor` |

## Ver también

- [[34-Modelos-Lineales-en-Profundidad]] (en `Machine Learning/`)
- [[03 - Pipelines y ColumnTransformer]]
- [[10 - Selección de Features]]
- [[11 - Datos Faltantes y Clases Desbalanceadas]]
