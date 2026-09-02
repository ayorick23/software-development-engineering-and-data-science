---
tags: [scikit-learn, machine-learning, pipeline, columntransformer, cheat-sheet]
---

# 03 — Pipelines y ColumnTransformer

> Continúa de [[02 - Preprocessing y Escalado]]. Es la pieza que conecta preprocesamiento con modelado de forma segura contra leakage.

Un **Pipeline** encadena varios pasos (transformadores + un estimador final) en un solo objeto que se comporta como cualquier otro estimador de scikit-learn — tiene `.fit()`, `.predict()`, se puede pasar a `cross_val_score`, a `GridSearchCV`, y se serializa como una sola unidad.

## Por qué usar Pipeline en vez de transformar "a mano"

```python
# INCORRECTO — riesgo real de leakage
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
modelo = LogisticRegression()
modelo.fit(X_train_scaled, y_train)
# fácil de olvidar aplicar el MISMO scaler (ya ajustado) a X_test en vez de uno nuevo

# CORRECTO — todo el flujo empaquetado, imposible de aplicar mal
pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("modelo", LogisticRegression()),
])
pipeline.fit(X_train, y_train)          # fit_transform en scaler, luego fit en modelo — todo con TRAIN
pipeline.predict(X_test)                 # transform en scaler (NO fit), luego predict — todo con TEST
```

El beneficio no es solo comodidad de código: un Pipeline hace **estructuralmente imposible** filtrar información de test hacia el ajuste de los transformadores, porque `cross_val_score`/`GridSearchCV` llaman `.fit()` del pipeline completo únicamente sobre cada fold de entrenamiento.

## `Pipeline` — sintaxis explícita

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Ridge

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("modelo", Ridge(alpha=1.0)),
])

pipeline.fit(X_train, y_train)
predicciones = pipeline.predict(X_test)
score = pipeline.score(X_test, y_test)
```

Cada paso es una tupla `(nombre, transformador)`. Todos los pasos excepto el último deben ser transformadores (implementar `.transform()`); el último paso puede ser un transformador o un estimador final (predictor).

## `make_pipeline` — atajo sin nombrar los pasos manualmente

```python
from sklearn.pipeline import make_pipeline

pipeline = make_pipeline(StandardScaler(), Ridge(alpha=1.0))
# los nombres se generan automáticamente en minúsculas: "standardscaler", "ridge"
```

## Acceder a pasos individuales del Pipeline

```python
pipeline.named_steps["scaler"]           # acceso por nombre
pipeline["scaler"]                        # sintaxis de indexado, equivalente
pipeline.steps[0]                         # acceso por posición, devuelve tupla (nombre, objeto)

pipeline.named_steps["modelo"].coef_      # atributos aprendidos del último paso
```

## `ColumnTransformer` — aplicar transformaciones distintas a columnas distintas

El caso real casi siempre involucra columnas numéricas y categóricas que necesitan tratamiento diferente — `ColumnTransformer` resuelve exactamente esto.

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

columnas_numericas = ["dias_atras", "temperatura", "precio"]
columnas_categoricas = ["region", "tipo_producto"]

preprocesador = ColumnTransformer([
    ("num", StandardScaler(), columnas_numericas),
    ("cat", OneHotEncoder(handle_unknown="ignore"), columnas_categoricas),
])

X_transformado = preprocesador.fit_transform(X_train)
```

`ColumnTransformer` aplica cada transformador **solo** a las columnas indicadas, y concatena los resultados en una sola matriz de salida — las columnas numéricas terminan escaladas, las categóricas terminan one-hot-encoded, todo en un solo objeto.

### `remainder` — qué hacer con las columnas no mencionadas

```python
preprocesador = ColumnTransformer(
    transformers=[
        ("num", StandardScaler(), columnas_numericas),
        ("cat", OneHotEncoder(), columnas_categoricas),
    ],
    remainder="passthrough",   # columnas no listadas pasan sin modificar (default: "drop", las elimina)
)
```

### `make_column_selector` — seleccionar columnas por tipo de dato, no por nombre

```python
from sklearn.compose import make_column_selector, make_column_transformer

preprocesador = make_column_transformer(
    (StandardScaler(), make_column_selector(dtype_include="number")),
    (OneHotEncoder(handle_unknown="ignore"), make_column_selector(dtype_include="object")),
)
```

Útil cuando el dataset tiene muchas columnas y nombrarlas todas manualmente sería tedioso — selecciona automáticamente por tipo de dato del DataFrame.

## El patrón completo: ColumnTransformer + Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import RandomForestRegressor

preprocesador = ColumnTransformer([
    ("num", Pipeline([
        ("imputer", SimpleImputer(strategy="median")),
        ("scaler", StandardScaler()),
    ]), columnas_numericas),
    ("cat", Pipeline([
        ("imputer", SimpleImputer(strategy="most_frequent")),
        ("encoder", OneHotEncoder(handle_unknown="ignore")),
    ]), columnas_categoricas),
])

pipeline_completo = Pipeline([
    ("preprocesamiento", preprocesador),
    ("modelo", RandomForestRegressor(n_estimators=300, random_state=42)),
])

pipeline_completo.fit(X_train, y_train)
predicciones = pipeline_completo.predict(X_test)
```

Este es el patrón estándar de producción: imputación + escalado/encoding por tipo de columna + modelo final, todo como un único objeto serializable (ver [[13 - Persistencia de Modelos]]).

## `FeatureUnion` — combinar transformaciones en paralelo sobre las MISMAS columnas

Distinto a `ColumnTransformer` (que separa columnas): `FeatureUnion` aplica varios transformadores **al mismo input** y concatena horizontalmente los resultados.

```python
from sklearn.pipeline import FeatureUnion
from sklearn.decomposition import PCA
from sklearn.feature_selection import SelectKBest

combinacion = FeatureUnion([
    ("pca", PCA(n_components=5)),
    ("seleccion", SelectKBest(k=10)),
])
X_combinado = combinacion.fit_transform(X_train, y_train)   # concatena 5 componentes PCA + 10 features seleccionadas
```

En la práctica, `ColumnTransformer` cubre la mayoría de los casos reales (columnas distintas, tratamiento distinto); `FeatureUnion` es para cuando se quieren **múltiples vistas transformadas del mismo conjunto de columnas**.

## `FunctionTransformer` — envolver funciones propias como transformador

```python
from sklearn.preprocessing import FunctionTransformer
import numpy as np

log_transformer = FunctionTransformer(np.log1p, validate=True)
X_log = log_transformer.fit_transform(X_train[["precio"]])

# Con función custom y su inversa (útil para invertir predicciones transformadas):
transformer = FunctionTransformer(
    func=lambda x: np.log1p(x),
    inverse_func=lambda x: np.expm1(x),
)
```

Forma rápida de meter lógica de transformación simple (sin estado que aprender de los datos) dentro de un Pipeline, sin tener que escribir una clase completa — para transformadores con estado aprendido, ver [[14 - Extensión de Scikit-learn - Custom Estimators]].

## `TransformedTargetRegressor` — transformar también la variable objetivo

```python
from sklearn.compose import TransformedTargetRegressor
import numpy as np

modelo = TransformedTargetRegressor(
    regressor=Ridge(alpha=1.0),
    func=np.log1p,          # transforma y ANTES de entrenar
    inverse_func=np.expm1,  # revierte la transformación en las predicciones automáticamente
)
modelo.fit(X_train, y_train)
predicciones = modelo.predict(X_test)   # ya vienen en la escala ORIGINAL, no en log
```

Resuelve un problema común: cuando `y` tiene distribución muy asimétrica (precios, demanda) y conviene entrenar en escala logarítmica, pero las predicciones deben reportarse en la escala original — sin esto, habría que aplicar `np.expm1` manualmente después de cada `.predict()`, con riesgo de olvidarlo.

## Visualizar el Pipeline

```python
from sklearn import set_config
set_config(display="diagram")

pipeline_completo   # en un notebook/Jupyter, muestra un diagrama interactivo del pipeline completo
```

## Ver también

- [[02 - Preprocessing y Escalado]]
- [[04 - Model Selection - Validación y Búsqueda]]
- [[14 - Extensión de Scikit-learn - Custom Estimators]]
- [[16 - Buenas Prácticas, Errores Comunes y Rendimiento]]
