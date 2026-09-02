---
tags: [scikit-learn, machine-learning, buenas-practicas, rendimiento, cheat-sheet]
---

# 16 — Buenas Prácticas, Errores Comunes y Rendimiento

> Cierre del cheat-sheet. Se apoya en todos los archivos anteriores, especialmente [[03 - Pipelines y ColumnTransformer]] y [[04 - Model Selection - Validación y Búsqueda]].

## El error más costoso: data leakage por transformar antes de dividir

```python
# INCORRECTO — el scaler "ve" estadísticas de TODO el dataset, incluyendo lo que será test
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)             # fit sobre TODO X, antes del split
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y)

# CORRECTO — dividir primero, ajustar el scaler SOLO con train
X_train, X_test, y_train, y_test = train_test_split(X, y)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)        # NO fit_transform aquí

# MEJOR AÚN — usar Pipeline, estructuralmente imposible de hacer mal
pipeline = Pipeline([("scaler", StandardScaler()), ("modelo", Ridge())])
pipeline.fit(X_train, y_train)   # el scaler solo ve X_train, garantizado por diseño
```

Este es el error #1 en proyectos de scikit-learn — cualquier estadística calculada sobre el dataset completo antes de dividir (media, desviación estándar, categorías vistas, features seleccionadas) filtra información de test hacia el entrenamiento, produciendo métricas de validación optimistamente sesgadas que no se sostienen en producción.

## `cross_val_score` sin Pipeline — el mismo error, más sutil

```python
# INCORRECTO — el scaler se ajusta UNA VEZ sobre todo X_train, fuera del loop de cross-validation
X_train_scaled = StandardScaler().fit_transform(X_train)
scores = cross_val_score(Ridge(), X_train_scaled, y_train, cv=5)
# cada fold de "validación" dentro del cross-validation YA fue visto por el scaler

# CORRECTO — el Pipeline re-ajusta el scaler en cada fold, usando SOLO ese fold de entrenamiento
pipeline = Pipeline([("scaler", StandardScaler()), ("modelo", Ridge())])
scores = cross_val_score(pipeline, X_train, y_train, cv=5)
```

`cross_val_score`/`GridSearchCV` llaman `.fit()` del objeto que reciben en cada fold — si ese objeto es solo el modelo (con datos ya pre-escalados fuera), el escalado no se repite por fold y hay leakage; si es un `Pipeline` completo, el escalado se re-ajusta correctamente en cada fold.

## No usar `TimeSeriesSplit` con datos secuenciales

```python
# INCORRECTO en series de tiempo — mezcla pasado y futuro entre folds
scores = cross_val_score(pipeline, X_train, y_train, cv=5)   # KFold por defecto, con shuffle implícito según versión

# CORRECTO
from sklearn.model_selection import TimeSeriesSplit
scores = cross_val_score(pipeline, X_train, y_train, cv=TimeSeriesSplit(n_splits=5))
```

Ver [[04 - Model Selection - Validación y Búsqueda]] y `Machine Learning/37-Validacion-Rigurosa-en-ML.md`.

## Comparar coeficientes/importancias sin escalar features primero

```python
# ENGAÑOSO — una feature en escala [0, 1000000] puede tener coeficiente pequeño solo por su magnitud
modelo = LinearRegression().fit(X_train, y_train)   # X_train SIN escalar
coeficientes = pd.Series(modelo.coef_, index=X_train.columns)

# CORRECTO — escalar antes de interpretar magnitudes relativas entre coeficientes
pipeline = make_pipeline(StandardScaler(), LinearRegression())
pipeline.fit(X_train, y_train)
coeficientes = pd.Series(pipeline.named_steps["linearregression"].coef_, index=X_train.columns)
```

## Olvidar `random_state` en experimentos que deben ser reproducibles

```python
# Cada ejecución da resultados ligeramente distintos, complica debugging y comparación de experimentos
modelo = RandomForestRegressor()
X_train, X_test, y_train, y_test = train_test_split(X, y)

# Reproducible entre ejecuciones y entre sesiones de trabajo
modelo = RandomForestRegressor(random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)
```

## Ignorar `n_jobs` en operaciones paralelizables

```python
# Lento — usa un solo core, sin necesidad
modelo = RandomForestRegressor(n_estimators=500)
grid = GridSearchCV(modelo, param_grid, cv=5)

# Rápido — paraleliza tanto el entrenamiento del ensamble como la búsqueda de hiperparámetros
modelo = RandomForestRegressor(n_estimators=500, n_jobs=-1)
grid = GridSearchCV(modelo, param_grid, cv=5, n_jobs=-1)
```

> **Cuidado con doble paralelización**: poner `n_jobs=-1` tanto en el modelo como en `GridSearchCV` simultáneamente puede sobre-suscribir los cores disponibles (cada combinación de `GridSearchCV` corriendo en paralelo, y cada una intentando usar todos los cores para el modelo) — en la práctica, suele ser más eficiente paralelizar en un solo nivel (típicamente en `GridSearchCV`/`cross_val_score`, dejando `n_jobs=1` en el modelo individual) cuando hay muchas combinaciones a probar.

## Rendimiento con datasets grandes

### Matrices dispersas (sparse) — no convertir a denso innecesariamente

```python
from scipy.sparse import issparse

# OneHotEncoder con muchas categorías produce matrices dispersas por defecto — mantenerlas así
encoder = OneHotEncoder(sparse_output=True)   # default en versiones recientes
X_encoded = encoder.fit_transform(X_train)     # matriz dispersa — ocupa MUCHO menos memoria

print(issparse(X_encoded))   # True
# Convertir a denso (.toarray()) solo si un paso posterior específicamente lo requiere
```

Convertir prematuramente una matriz dispersa a densa con `.toarray()`/`sparse_output=False` puede multiplicar el uso de memoria por órdenes de magnitud cuando hay muchas columnas categóricas de alta cardinalidad — mantener la representación dispersa el mayor tiempo posible es clave para escalar a datasets grandes.

### `HistGradientBoosting*` en vez de `GradientBoosting*` para datasets grandes

Ya cubierto en [[07 - Árboles y Ensambles - Sintaxis y API]] — la versión basada en histogramas escala considerablemente mejor en filas que la implementación clásica.

### `partial_fit` para datos que no caben en memoria

```python
from sklearn.linear_model import SGDRegressor

modelo = SGDRegressor()
for X_batch, y_batch in generador_de_lotes(dataset_grande):
    modelo.partial_fit(X_batch, y_batch)   # entrena incrementalmente, sin cargar todo en memoria a la vez
```

Disponible en `SGDClassifier`/`SGDRegressor`, `MiniBatchKMeans`, `MultinomialNB`, entre otros — la solución nativa de scikit-learn para datasets que exceden la RAM disponible, sin necesitar migrar a otra librería.

## Checklist final antes de considerar un pipeline "listo"

1. ¿Todo el preprocesamiento (escalado, encoding, imputación, selección de features) vive dentro de un `Pipeline`, no aplicado manualmente antes del split?
2. ¿La validación usa la estrategia de `cv` correcta para la naturaleza de los datos (`StratifiedKFold` para clases desbalanceadas, `TimeSeriesSplit` para series temporales, `GroupKFold` si hay grupos no independientes)?
3. ¿`random_state` está fijado en todos los componentes con aleatoriedad?
4. ¿Los coeficientes/importancias se interpretan sobre features ya escaladas, si el modelo lo requiere?
5. ¿El `Pipeline` completo (no solo el modelo final) es lo que se serializa con `joblib`?
6. ¿La versión exacta de `scikit-learn`/`numpy`/`scipy` está fijada en el entorno de despliegue?
7. Si hay resampling (SMOTE), ¿se usa `imblearn.pipeline.Pipeline`, no el estándar de scikit-learn?

## Ver también

- [[03 - Pipelines y ColumnTransformer]]
- [[04 - Model Selection - Validación y Búsqueda]]
- [[13 - Persistencia de Modelos]]
- `Machine Learning/37-Validacion-Rigurosa-en-ML.md`
