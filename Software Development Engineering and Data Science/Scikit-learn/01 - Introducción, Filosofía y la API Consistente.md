---
tags: [scikit-learn, machine-learning, cheat-sheet]
---

# 01 — Introducción, Filosofía y la API Consistente

> Este cheat-sheet profundiza en la **sintaxis y API** de scikit-learn. Para la teoría matemática detrás de cada familia de algoritmos, ver las notas de `Machine Learning/`: [[34-Modelos-Lineales-en-Profundidad]], [[35-Arboles-de-Decision-en-Profundidad]], [[36-Ensambles-en-Profundidad]], [[37-Validacion-Rigurosa-en-ML]].

## ¿Qué es scikit-learn y por qué su diseño importa?

**scikit-learn** es la librería de referencia de Machine Learning clásico en Python. Su valor no es solo la colección de algoritmos — es que **todos comparten la misma interfaz**. Una vez que entiendes cómo se usa un `RandomForestClassifier`, sabes cómo usar un `SVC`, un `KMeans` o un `PCA`, porque todos obedecen el mismo contrato. Esa consistencia es lo que hace que Pipelines, `GridSearchCV` y la composición de transformadores funcionen de forma genérica sin importar el algoritmo interno.

## Instalación

```bash
pip install scikit-learn

# Para verificar versión y configuración de build (BLAS, OpenMP, etc.):
python -c "import sklearn; sklearn.show_versions()"
```

## Los tres roles: Estimator, Transformer, Predictor

Todo objeto de scikit-learn es un **Estimator** — implementa como mínimo `.fit()`. A partir de ahí, se especializa en uno o más roles adicionales:

| Rol | Método característico | Ejemplos |
|---|---|---|
| **Estimator** | `.fit(X, y=None)` | Base de todo — modelos, transformadores, todos lo implementan |
| **Transformer** | `.transform(X)` / `.fit_transform(X)` | `StandardScaler`, `PCA`, `OneHotEncoder` |
| **Predictor** | `.predict(X)` | `LinearRegression`, `RandomForestClassifier`, `KMeans` |

Algunos objetos son ambas cosas — por ejemplo, `KMeans` es Predictor (`.predict()` asigna cluster) y también Transformer (`.transform()` devuelve distancias a los centroides).

## El contrato `fit` / `predict` / `transform`

```python
from sklearn.linear_model import LinearRegression

modelo = LinearRegression()      # 1. Instanciar — NO entrena nada todavía, solo configura hiperparámetros
modelo.fit(X_train, y_train)     # 2. Entrenar — aprende del conjunto de entrenamiento, muta el objeto
predicciones = modelo.predict(X_test)   # 3. Predecir — usa lo aprendido, NO vuelve a entrenar

score = modelo.score(X_test, y_test)    # 4. (opcional) Evaluar — métrica por defecto del estimador
```

Para transformadores, el ciclo es análogo pero con `transform` en vez de `predict`:

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
scaler.fit(X_train)                    # aprende media y desviación estándar de X_train
X_train_scaled = scaler.transform(X_train)   # aplica la transformación aprendida
X_test_scaled = scaler.transform(X_test)     # usa los MISMOS parámetros aprendidos de train, no re-fitea

# Atajo cuando fit y transform ocurren sobre el mismo dato (SOLO en train):
X_train_scaled = scaler.fit_transform(X_train)
```

> **Regla de oro que causa la mayoría de los bugs de leakage**: `fit()` (o `fit_transform()`) se llama **solo sobre datos de entrenamiento**. Sobre validación/test, siempre `transform()` únicamente — nunca volver a ajustar. Se profundiza en [[16 - Buenas Prácticas, Errores Comunes y Rendimiento]].

## Convención de nombres: parámetros vs. atributos aprendidos

```python
modelo = RandomForestClassifier(n_estimators=200, max_depth=8)
# n_estimators, max_depth: HIPERPARÁMETROS — se pasan al constructor, existen ANTES de fit()

modelo.fit(X_train, y_train)

modelo.feature_importances_   # ATRIBUTO APRENDIDO — termina en "_", solo existe DESPUÉS de fit()
modelo.classes_                # idem
modelo.n_features_in_          # idem
```

La convención `nombre_` (guión bajo al final) es universal en scikit-learn: cualquier atributo que termina en `_` fue **aprendido de los datos**, no configurado por el usuario. Intentar acceder a uno de estos atributos antes de llamar `.fit()` lanza `NotFittedError`.

## `get_params()` / `set_params()` — introspección y modificación de hiperparámetros

```python
modelo = RandomForestClassifier(n_estimators=100)

print(modelo.get_params())
# {'n_estimators': 100, 'max_depth': None, 'criterion': 'gini', ...}

modelo.set_params(n_estimators=300, max_depth=10)   # modifica hiperparámetros sin recrear el objeto
```

Esto es lo que permite que `GridSearchCV` funcione de forma genérica con cualquier estimador — internamente usa `set_params()` para probar cada combinación, sin necesitar conocer los hiperparámetros específicos del algoritmo de antemano.

## Mapa de módulos — dónde vive cada cosa

| Módulo | Contenido |
|---|---|
| `sklearn.linear_model` | Regresión lineal, Ridge, Lasso, LogisticRegression, SGD |
| `sklearn.tree` | Árboles de decisión |
| `sklearn.ensemble` | Random Forest, Gradient Boosting, Voting, Stacking, Bagging, AdaBoost |
| `sklearn.svm` | Support Vector Machines |
| `sklearn.neighbors` | KNN |
| `sklearn.naive_bayes` | Naive Bayes |
| `sklearn.cluster` | KMeans, DBSCAN, clustering jerárquico |
| `sklearn.decomposition` | PCA, SVD, NMF |
| `sklearn.preprocessing` | Escalado, encoding, transformación de features |
| `sklearn.impute` | Imputación de valores faltantes |
| `sklearn.feature_selection` | Selección de variables |
| `sklearn.pipeline` | `Pipeline`, `FeatureUnion` |
| `sklearn.compose` | `ColumnTransformer` |
| `sklearn.model_selection` | Splits, cross-validation, búsqueda de hiperparámetros |
| `sklearn.metrics` | Todas las métricas de evaluación |
| `sklearn.base` | Clases base para crear estimadores custom |

## `random_state` — reproducibilidad explícita

La mayoría de los estimadores con componentes aleatorios (`RandomForestClassifier`, `train_test_split`, `KMeans` con inicialización aleatoria) aceptan `random_state`:

```python
modelo = RandomForestClassifier(random_state=42)   # mismos resultados en cada ejecución
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)
```

Sin fijarlo, cada ejecución puede dar resultados ligeramente distintos — crítico para reproducibilidad en experimentos y en tests automatizados del pipeline (ver también `MLflow/02 - Tracking - Fundamentos y API de Logging.md` para registrar `random_state` como parámetro trazable).

## `n_jobs` — paralelización nativa

```python
modelo = RandomForestClassifier(n_jobs=-1)   # usa TODOS los cores disponibles
modelo = RandomForestClassifier(n_jobs=4)    # usa 4 cores específicamente
```

Disponible en estimadores cuyo entrenamiento es paralelizable de forma natural (Random Forest entre árboles, `GridSearchCV` entre combinaciones, KNN en la búsqueda de vecinos). No todos los algoritmos lo soportan — Gradient Boosting clásico, por su naturaleza secuencial, no se beneficia igual (para eso existe `HistGradientBoostingClassifier`, ver [[07 - Árboles y Ensambles - Sintaxis y API]]).

## `set_config` — configuración global de comportamiento

```python
from sklearn import set_config

set_config(transform_output="pandas")   # transformers devuelven DataFrame en vez de numpy array
set_config(display="diagram")            # Pipelines se renderizan visualmente en Jupyter
set_config(print_changed_only=True)      # repr() de estimadores solo muestra params NO default
```

## Ver también

- [[03 - Pipelines y ColumnTransformer]]
- [[04 - Model Selection - Validación y Búsqueda]]
- [[14 - Extensión de Scikit-learn - Custom Estimators]]
- `Machine Learning/06-Familia-de-Algoritmos-ML.md`
