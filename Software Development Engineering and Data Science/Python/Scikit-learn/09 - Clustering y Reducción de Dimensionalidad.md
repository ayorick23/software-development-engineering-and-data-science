---
tags: [scikit-learn, machine-learning, clustering, pca, no-supervisado, cheat-sheet]
---

# 09 — Clustering y Reducción de Dimensionalidad

> Continúa de [[08 - SVM, Vecinos Cercanos y Naive Bayes]]. Cubre `sklearn.cluster` y `sklearn.decomposition`.

## Clustering (`sklearn.cluster`)

### `KMeans` — el algoritmo de partición más usado

```python
from sklearn.cluster import KMeans

modelo = KMeans(
    n_clusters=4,             # el hiperparámetro central — número de clusters a encontrar
    init="k-means++",          # inicialización inteligente de centroides (evita mínimos locales pobres)
    n_init="auto",              # número de inicializaciones distintas; se queda con la de menor inercia
    max_iter=300,
    random_state=42,
)
modelo.fit(X_scaled)   # SIEMPRE escalar antes — KMeans usa distancia euclidiana

modelo.labels_          # cluster asignado a cada muestra de entrenamiento
modelo.cluster_centers_  # coordenadas de los centroides finales
modelo.inertia_          # suma de distancias al cuadrado dentro de cada cluster (menor = clusters más compactos)

modelo.predict(X_nuevo)   # asigna cluster a muestras nuevas, según el centroide más cercano
```

### Elegir `n_clusters` — método del codo y silhouette

```python
import matplotlib.pyplot as plt
from sklearn.metrics import silhouette_score

inercias, siluetas = [], []
for k in range(2, 11):
    modelo = KMeans(n_clusters=k, random_state=42, n_init="auto").fit(X_scaled)
    inercias.append(modelo.inertia_)
    siluetas.append(silhouette_score(X_scaled, modelo.labels_))

plt.plot(range(2, 11), inercias)   # buscar el "codo" — donde agregar más clusters deja de reducir mucho la inercia
```

`KMeans` requiere **especificar `n_clusters` de antemano** — no lo descubre solo. El método del codo (inercia) y el `silhouette_score` (ver [[05 - Métricas y Evaluación]]) son las dos formas estándar de elegir un valor razonable cuando no hay conocimiento de dominio que lo determine directamente.

### `DBSCAN` — clustering basado en densidad, sin especificar número de clusters

```python
from sklearn.cluster import DBSCAN

modelo = DBSCAN(
    eps=0.5,             # radio de vecindad — el hiperparámetro más sensible
    min_samples=5,         # mínimo de puntos para considerar una región "densa"
)
labels = modelo.fit_predict(X_scaled)

# labels == -1 identifica outliers/ruido — DBSCAN los detecta de forma NATIVA, a diferencia de KMeans
```

Ventaja clave sobre `KMeans`: no requiere especificar el número de clusters, encuentra clusters de forma arbitraria (no solo esféricos/convexos), y clasifica explícitamente puntos como ruido en vez de forzarlos dentro de algún cluster. Desventaja: sensible a la elección de `eps`, y no funciona bien cuando los clusters tienen densidades muy distintas entre sí.

### `AgglomerativeClustering` — clustering jerárquico

```python
from sklearn.cluster import AgglomerativeClustering

modelo = AgglomerativeClustering(
    n_clusters=4,
    linkage="ward",   # "ward" (minimiza varianza intra-cluster), "complete", "average", "single"
)
labels = modelo.fit_predict(X_scaled)
```

Construye una jerarquía de clusters (de abajo hacia arriba, fusionando los más cercanos iterativamente) — el resultado se puede visualizar como un **dendrograma**, útil para explorar visualmente cuántos clusters tienen sentido antes de fijar `n_clusters`:

```python
from scipy.cluster.hierarchy import dendrogram, linkage
import matplotlib.pyplot as plt

Z = linkage(X_scaled, method="ward")
dendrogram(Z)
plt.show()
```

### `GaussianMixture` — clustering probabilístico (soft clustering)

```python
from sklearn.mixture import GaussianMixture   # nota: vive en sklearn.mixture, no en sklearn.cluster

modelo = GaussianMixture(n_components=4, covariance_type="full", random_state=42)
modelo.fit(X_scaled)

labels = modelo.predict(X_scaled)              # asignación "dura" (cluster más probable)
probabilidades = modelo.predict_proba(X_scaled)  # probabilidad de pertenecer a CADA cluster — soft clustering
```

A diferencia de `KMeans` (que asigna cada punto a exactamente un cluster, "hard clustering"), `GaussianMixture` modela cada cluster como una distribución gaussiana y da una **probabilidad** de pertenencia a cada uno — útil cuando los límites entre grupos son ambiguos por naturaleza, no discretos.

## Reducción de Dimensionalidad (`sklearn.decomposition`)

### `PCA` — Análisis de Componentes Principales

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=0.95)   # conserva componentes hasta explicar el 95% de la varianza (o un entero fijo)
X_reducido = pca.fit_transform(X_scaled)   # SIEMPRE escalar antes — PCA es sensible a la escala

print(pca.n_components_)                    # cuántos componentes resultaron necesarios
print(pca.explained_variance_ratio_)        # varianza explicada por cada componente individual
print(pca.explained_variance_ratio_.cumsum())  # varianza acumulada
```

PCA encuentra las direcciones (combinaciones lineales de las features originales) que capturan la mayor varianza posible en los datos, ordenadas de mayor a menor importancia. Usos principales: reducir dimensionalidad antes de un modelo sensible a la maldición de la dimensionalidad (KNN, clustering), visualización en 2D/3D, y eliminar colinealidad entre features.

```python
# Graficar varianza explicada acumulada — para decidir cuántos componentes conservar:
import matplotlib.pyplot as plt
pca_completo = PCA().fit(X_scaled)
plt.plot(pca_completo.explained_variance_ratio_.cumsum())
plt.xlabel("Número de componentes"); plt.ylabel("Varianza explicada acumulada")
```

> **Costo de interpretabilidad**: los componentes de PCA son combinaciones lineales de TODAS las features originales — se pierde la interpretación directa de "esta feature específica importa". Para casos donde la interpretabilidad de features individuales es prioritaria, considerar selección de features (ver [[10 - Selección de Features]]) en vez de PCA.

### `TruncatedSVD` — PCA para matrices dispersas (sparse)

```python
from sklearn.decomposition import TruncatedSVD

svd = TruncatedSVD(n_components=100, random_state=42)
X_reducido = svd.fit_transform(X_tfidf_sparse)   # funciona directamente sobre matrices dispersas
```

`PCA` estándar requiere centrar los datos (restar la media), lo que destruye la dispersión (sparsity) de una matriz — `TruncatedSVD` evita ese problema, siendo la opción estándar para reducir dimensionalidad de vectores TF-IDF en NLP (esto es, de hecho, la base de LSA — Latent Semantic Analysis).

### `NMF` — Factorización No-negativa de Matrices

```python
from sklearn.decomposition import NMF

modelo = NMF(n_components=10, init="nndsvda", random_state=42)
W = modelo.fit_transform(X)   # requiere que X tenga SOLO valores no negativos
H = modelo.components_
```

Similar en espíritu a PCA/SVD, pero restringe los componentes a valores no negativos — produce factores más interpretables en contextos donde los valores negativos no tienen sentido físico (conteos de palabras, píxeles de imagen, intensidades).

### `t-SNE` — visualización no lineal (no para preprocesamiento de modelos)

```python
from sklearn.manifold import TSNE

tsne = TSNE(n_components=2, perplexity=30, random_state=42)
X_2d = tsne.fit_transform(X_scaled)   # SOLO para visualización, nunca como input de un modelo posterior
```

> **Advertencia importante**: a diferencia de PCA, `t-SNE` **no tiene un método `.transform()` separado** ni preserva distancias globales de forma confiable — está diseñado exclusivamente para visualización exploratoria en 2D/3D, no como paso de un pipeline de modelado. No usar sus salidas como features de entrada a otro modelo.

### `LinearDiscriminantAnalysis` — reducción supervisada

```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis

lda = LinearDiscriminantAnalysis(n_components=2)   # máximo: n_clases - 1
X_reducido = lda.fit_transform(X_train, y_train)    # requiere y — a diferencia de PCA, es SUPERVISADO
```

A diferencia de PCA (que maximiza varianza sin considerar las clases), LDA busca las direcciones que **mejor separan las clases conocidas** — útil cuando el objetivo final es clasificación y se quiere reducir dimensionalidad preservando esa separabilidad específicamente.

## Ver también

- [[02 - Preprocessing y Escalado]]
- [[05 - Métricas y Evaluación]] (métricas de clustering)
- [[10 - Selección de Features]]
