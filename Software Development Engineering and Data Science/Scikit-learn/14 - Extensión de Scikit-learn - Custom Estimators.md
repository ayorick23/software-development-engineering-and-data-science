---
tags: [scikit-learn, machine-learning, custom-estimators, cheat-sheet]
---

# 14 — Extensión de Scikit-learn: Custom Estimators

> Se apoya en [[01 - Introducción, Filosofía y la API Consistente]] y [[03 - Pipelines y ColumnTransformer]].

Escribir un transformador o modelo propio que sea 100% compatible con `Pipeline`, `GridSearchCV` y `cross_val_score` requiere seguir un contrato específico — este archivo cubre ese contrato.

## Las clases base en `sklearn.base`

```python
from sklearn.base import BaseEstimator, TransformerMixin, ClassifierMixin, RegressorMixin
```

| Clase base | Qué aporta |
|---|---|
| `BaseEstimator` | `get_params()`/`set_params()` automáticos, `__repr__` legible — base de TODO estimador custom |
| `TransformerMixin` | Genera `fit_transform()` automáticamente a partir de `fit()` + `transform()` |
| `ClassifierMixin` | Genera `.score()` automáticamente (accuracy por defecto), marca el estimador como clasificador |
| `RegressorMixin` | Genera `.score()` automáticamente (R² por defecto), marca el estimador como regresor |

## Un Transformer custom — el caso más común

```python
from sklearn.base import BaseEstimator, TransformerMixin
import numpy as np

class ClipOutliers(BaseEstimator, TransformerMixin):
    """Recorta valores fuera de un rango basado en percentiles aprendidos del train set."""

    def __init__(self, percentil_inferior=1, percentil_superior=99):
        # REGLA DE ORO: __init__ solo asigna parámetros, NO valida ni transforma nada aquí
        self.percentil_inferior = percentil_inferior
        self.percentil_superior = percentil_superior

    def fit(self, X, y=None):
        X = np.asarray(X)
        self.limite_inferior_ = np.percentile(X, self.percentil_inferior, axis=0)
        self.limite_superior_ = np.percentile(X, self.percentil_superior, axis=0)
        self.n_features_in_ = X.shape[1]   # atributo esperado por scikit-learn internamente
        return self   # fit SIEMPRE retorna self

    def transform(self, X):
        X = np.asarray(X)
        return np.clip(X, self.limite_inferior_, self.limite_superior_)

# Uso idéntico a cualquier transformer nativo:
clipper = ClipOutliers(percentil_inferior=5, percentil_superior=95)
X_train_clipped = clipper.fit_transform(X_train)
X_test_clipped = clipper.transform(X_test)   # usa los límites aprendidos de TRAIN
```

### Las reglas que hacen que un estimador sea "bien portado"

1. **`__init__` no hace nada más que guardar parámetros** — cada argumento de `__init__` se guarda como un atributo del **mismo nombre**, sin transformarlo ni validarlo ahí. La validación va dentro de `fit()`. Esto es lo que permite que `get_params()`/`set_params()`/`clone()` funcionen correctamente.
2. **Atributos aprendidos terminan en `_`** (`limite_inferior_`, no `limite_inferior`) — convención que distingue "lo que el usuario configuró" de "lo que el modelo aprendió" (ver [[01 - Introducción, Filosofía y la API Consistente]]).
3. **`fit()` siempre retorna `self`** — necesario para poder encadenar `.fit(X).transform(X)` y para que `Pipeline` funcione.
4. **`transform()`/`predict()` no modifican el estado del objeto** — solo leen los atributos ya aprendidos en `fit()`.

## Un Predictor (Classifier) custom

```python
from sklearn.base import BaseEstimator, ClassifierMixin
from sklearn.utils.validation import check_X_y, check_array, check_is_fitted
import numpy as np

class ClasificadorConReglas(BaseEstimator, ClassifierMixin):
    def __init__(self, modelo_base=None, umbral_regla_negocio=0.5):
        self.modelo_base = modelo_base
        self.umbral_regla_negocio = umbral_regla_negocio

    def fit(self, X, y):
        X, y = check_X_y(X, y)   # valida forma y tipos, lanza error claro si algo está mal
        self.classes_ = np.unique(y)
        self.modelo_base_ = self.modelo_base.fit(X, y)
        return self

    def predict_proba(self, X):
        check_is_fitted(self)   # lanza NotFittedError si predict se llama antes que fit
        X = check_array(X)
        return self.modelo_base_.predict_proba(X)

    def predict(self, X):
        probs = self.predict_proba(X)[:, 1]
        return (probs > self.umbral_regla_negocio).astype(int)
```

`check_X_y`, `check_array` y `check_is_fitted` (de `sklearn.utils.validation`) son las utilidades estándar para validar inputs de forma consistente con el resto de la librería — producen los mismos mensajes de error que verías al usar un estimador nativo mal usado.

## `check_estimator` — validar que el contrato se cumple correctamente

```python
from sklearn.utils.estimator_checks import check_estimator

check_estimator(ClipOutliers())   # corre una batería extensa de tests de compatibilidad de la API
```

Ejecuta automáticamente decenas de tests estándar (¿`fit` retorna `self`? ¿`get_params`/`set_params` son consistentes? ¿maneja bien arrays vacíos o con NaN según corresponda?) — la forma más rigurosa de confirmar que un estimador custom es "ciudadano de primera clase" dentro del ecosistema scikit-learn antes de usarlo dentro de `Pipeline`/`GridSearchCV` en producción.

## Por qué esto importa: compatibilidad con `Pipeline` y `GridSearchCV`

```python
pipeline = Pipeline([
    ("clip", ClipOutliers(percentil_inferior=1, percentil_superior=99)),
    ("scaler", StandardScaler()),
    ("modelo", Ridge()),
])

# GridSearchCV puede buscar hiperparámetros del transformer CUSTOM exactamente igual que uno nativo:
param_grid = {"clip__percentil_superior": [95, 97, 99]}
grid = GridSearchCV(pipeline, param_grid, cv=5)
grid.fit(X_train, y_train)
```

Esto solo funciona porque `ClipOutliers` respeta el contrato: `GridSearchCV` internamente llama `set_params(percentil_superior=...)`, que requiere que el nombre del parámetro en `__init__` coincida exactamente con el atributo guardado — cualquier desviación (por ejemplo, transformar el valor dentro de `__init__` en vez de guardarlo tal cual) rompe silenciosamente esta compatibilidad.

## `clone()` — por qué importa la regla de `__init__`

```python
from sklearn.base import clone

modelo_original = RandomForestRegressor(n_estimators=100, random_state=42)
modelo_clonado = clone(modelo_original)   # mismos hiperparámetros, SIN entrenar (no copia atributos con _)
```

`cross_val_score`/`GridSearchCV` usan `clone()` internamente para crear una copia "limpia" (sin entrenar) del estimador antes de cada fold — si `__init__` no guarda los parámetros exactamente como los recibió, `clone()` produce un objeto con hiperparámetros distintos al original, causando bugs muy difíciles de rastrear (el modelo entrenado en cada fold silenciosamente no es el que el usuario configuró).

## Estimador con hiperparámetros que dependen de otro objeto

```python
class ModeloConPreprocesamiento(BaseEstimator, RegressorMixin):
    def __init__(self, preprocesador=None, modelo=None):
        self.preprocesador = preprocesador   # se guarda TAL CUAL, sin clonar aquí
        self.modelo = modelo

    def fit(self, X, y):
        self.preprocesador_ = clone(self.preprocesador)   # clonar DENTRO de fit, no en __init__
        self.modelo_ = clone(self.modelo)
        X_transformado = self.preprocesador_.fit_transform(X)
        self.modelo_.fit(X_transformado, y)
        return self

    def predict(self, X):
        X_transformado = self.preprocesador_.transform(X)
        return self.modelo_.predict(X_transformado)
```

Cuando un hiperparámetro es en sí mismo otro estimador (patrón usado internamente por `Pipeline`, `VotingClassifier`, etc.), la convención es clonarlo **dentro de `fit()`**, nunca en `__init__` — de nuevo, para preservar la compatibilidad con `clone()`/`get_params()`.

## Ver también

- [[01 - Introducción, Filosofía y la API Consistente]]
- [[03 - Pipelines y ColumnTransformer]]
- [[04 - Model Selection - Validación y Búsqueda]]
