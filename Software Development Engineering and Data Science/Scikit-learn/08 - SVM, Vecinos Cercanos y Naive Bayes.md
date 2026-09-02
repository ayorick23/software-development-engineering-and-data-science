---
tags: [scikit-learn, machine-learning, svm, knn, naive-bayes, cheat-sheet]
---

# 08 — SVM, Vecinos Cercanos y Naive Bayes

> Continúa de [[07 - Árboles y Ensambles - Sintaxis y API]]. Estas tres familias no tienen una nota de teoría dedicada en `Machine Learning/` — este archivo incluye contexto conceptual además de la sintaxis.

## Support Vector Machines (`sklearn.svm`)

### La idea central

SVM busca el hiperplano que separa las clases con el **margen más amplio posible** — no solo cualquier frontera que clasifique bien, sino la que deja más distancia entre las muestras más cercanas de cada clase. Con datos no linealmente separables, el **kernel trick** proyecta los datos a un espacio de mayor dimensión donde sí lo son, sin calcular explícitamente esa proyección (se computa solo el producto punto en el espacio transformado).

### `SVC` — clasificación

```python
from sklearn.svm import SVC

modelo = SVC(
    C=1.0,                # regularización — igual que en LogisticRegression, menor C = más regularización
    kernel="rbf",           # "linear", "rbf", "poly", "sigmoid"
    gamma="scale",           # solo aplica a "rbf"/"poly"/"sigmoid" — controla el alcance de cada muestra
    probability=True,        # habilita predict_proba (más lento, usa validación cruzada interna)
    class_weight="balanced",
)
modelo.fit(X_train, y_train)   # SIEMPRE escalar features antes — SVM es muy sensible a la escala

modelo.support_vectors_   # las muestras que efectivamente definen el hiperplano
modelo.n_support_          # cuántos vectores de soporte por clase
```

- `kernel="linear"`: frontera de decisión lineal — rápido, interpretable, bueno cuando las clases ya son casi separables linealmente.
- `kernel="rbf"` (Radial Basis Function, el default): frontera no lineal flexible, el más usado en la práctica cuando no hay razón para asumir separabilidad lineal.
- `gamma`: valores altos = cada muestra individual tiene influencia muy local (riesgo de overfitting); valores bajos = influencia más global (riesgo de underfitting).

### `SVR` — regresión

```python
from sklearn.svm import SVR

modelo = SVR(kernel="rbf", C=1.0, epsilon=0.1)
# epsilon: ancho del "tubo" dentro del cual los errores NO se penalizan — a mayor epsilon, modelo más simple
modelo.fit(X_train, y_train)
```

### `LinearSVC` — versión optimizada para el caso lineal

```python
from sklearn.svm import LinearSVC

modelo = LinearSVC(C=1.0, max_iter=10000)   # mucho más rápido que SVC(kernel="linear") en datasets grandes
```

Usa una implementación distinta (`liblinear`) optimizada específicamente para el caso lineal — preferible a `SVC(kernel="linear")` cuando se sabe de antemano que no se necesita ningún otro kernel, especialmente en datasets con muchas muestras.

> **Advertencia de escalabilidad**: SVM con kernel no lineal escala mal — el entrenamiento es aproximadamente O(n² a n³) con el número de muestras. Para datasets de más de ~50,000-100,000 filas, SVM con `rbf` puede volverse impracticablemente lento; en ese régimen, Gradient Boosting o redes neuronales suelen ser mejores opciones prácticas.

## K-Nearest Neighbors (`sklearn.neighbors`)

### La idea central

KNN no "aprende" un modelo en el sentido tradicional — memoriza todo el conjunto de entrenamiento, y para predecir una muestra nueva, busca las `k` muestras más cercanas (según alguna métrica de distancia) y agrega sus valores (voto mayoritario en clasificación, promedio en regresión). Es un algoritmo *lazy* (todo el cómputo ocurre en `predict()`, no en `fit()`).

### `KNeighborsClassifier` / `KNeighborsRegressor`

```python
from sklearn.neighbors import KNeighborsClassifier

modelo = KNeighborsClassifier(
    n_neighbors=5,            # k — el hiperparámetro más importante
    weights="distance",        # "uniform" (todos los vecinos pesan igual) o "distance" (más cerca, más peso)
    metric="minkowski", p=2,   # p=2 es distancia euclidiana, p=1 es Manhattan
    algorithm="auto",           # "ball_tree", "kd_tree", "brute" — auto elige según los datos
    n_jobs=-1,
)
modelo.fit(X_train, y_train)   # SIEMPRE escalar features primero — KNN depende directamente de distancias
```

- `n_neighbors` pequeño (ej. 1-3): frontera de decisión muy irregular, alta varianza, riesgo de overfitting.
- `n_neighbors` grande: frontera más suave, alto sesgo, riesgo de underfitting.
- **Maldición de la dimensionalidad**: KNN se degrada con muchas features — en espacios de alta dimensión, la noción de "cercanía" pierde significado porque todas las distancias tienden a volverse similares. Considerar reducción de dimensionalidad (ver [[09 - Clustering y Reducción de Dimensionalidad]]) antes de aplicar KNN en datasets con muchas columnas.

### `NearestNeighbors` — búsqueda de vecinos sin clasificación/regresión

```python
from sklearn.neighbors import NearestNeighbors

nn = NearestNeighbors(n_neighbors=5)
nn.fit(X_train)

distancias, indices = nn.kneighbors(X_nuevo)   # los k vecinos más cercanos a cada muestra de X_nuevo
```

Base de sistemas de recomendación simples y de detección de anomalías (una muestra cuyos vecinos más cercanos están todos muy lejos es sospechosa de ser un outlier).

## Naive Bayes (`sklearn.naive_bayes`)

### La idea central

Aplica el teorema de Bayes asumiendo (ingenuamente, de ahí el nombre) que todas las features son **condicionalmente independientes** dado la clase. Esta suposición rara vez es literalmente cierta, pero el algoritmo funciona sorprendentemente bien en la práctica, es extremadamente rápido de entrenar (una sola pasada por los datos, sin iteración), y es el estándar de facto para clasificación de texto.

### `GaussianNB` — features numéricas continuas

```python
from sklearn.naive_bayes import GaussianNB

modelo = GaussianNB()
modelo.fit(X_train, y_train)   # asume que cada feature sigue una distribución normal dentro de cada clase
```

### `MultinomialNB` — conteos discretos (el estándar para texto)

```python
from sklearn.naive_bayes import MultinomialNB
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import make_pipeline

modelo = make_pipeline(
    TfidfVectorizer(max_features=5000),
    MultinomialNB(alpha=1.0),   # alpha: suavizado de Laplace, evita probabilidad cero para palabras no vistas
)
modelo.fit(textos_train, y_train)
```

`alpha` (suavizado de Laplace/Lidstone) es crítico: sin suavizado, cualquier palabra en el texto de prueba que nunca apareció en una clase durante el entrenamiento haría que la probabilidad de esa clase colapse a exactamente cero, sin importar el resto de la evidencia.

### `BernoulliNB` — features binarias (presencia/ausencia)

```python
from sklearn.naive_bayes import BernoulliNB

modelo = BernoulliNB(alpha=1.0)
# útil cuando solo importa si una palabra/feature APARECE o no, no cuántas veces (a diferencia de MultinomialNB)
```

### `ComplementNB` — variante robusta a clases desbalanceadas en texto

```python
from sklearn.naive_bayes import ComplementNB

modelo = ComplementNB()
# diseñado específicamente para corpus de texto con distribución de clases desigual — suele superar a MultinomialNB en ese escenario
```

## Tabla de decisión rápida

| Situación | Algoritmo |
|---|---|
| Frontera de decisión compleja, dataset mediano (< 50k filas) | `SVC` con kernel `rbf` |
| Frontera lineal, dataset grande | `LinearSVC` |
| Predicción basada en similitud local, pocas features | `KNeighborsClassifier`/`Regressor` |
| Clasificación de texto, rapidez de entrenamiento prioritaria | `MultinomialNB` / `ComplementNB` |
| Features numéricas continuas, baseline rápido | `GaussianNB` |
| Features binarias (presencia/ausencia) | `BernoulliNB` |

## Ver también

- [[02 - Preprocessing y Escalado]] (escalar es obligatorio para SVM y KNN)
- [[09 - Clustering y Reducción de Dimensionalidad]]
- [[15 - Integraciones con el Ecosistema]]
