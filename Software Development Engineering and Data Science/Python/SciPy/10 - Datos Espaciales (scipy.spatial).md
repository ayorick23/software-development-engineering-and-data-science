---
tags: [scipy, datos-espaciales, cheat-sheet]
---

# 10 — Datos Espaciales (`scipy.spatial`)

## Distancias entre puntos

```python
import numpy as np
from scipy.spatial import distance

p1 = np.array([0, 0])
p2 = np.array([3, 4])

distance.euclidean(p1, p2)     # 5.0
distance.cityblock(p1, p2)     # distancia Manhattan: 7
distance.cosine(p1, p2)        # distancia coseno (1 - similitud coseno) — común en embeddings de texto
```

### Matriz de distancias entre todos los pares de puntos

```python
puntos = np.random.rand(5, 2)

distance.pdist(puntos)                    # distancias entre todos los pares, formato condensado (vector)
distance.squareform(distance.pdist(puntos))   # convertida a matriz cuadrada simétrica
distance.cdist(puntos, otros_puntos)      # distancias entre dos conjuntos distintos de puntos
```

## KDTree — búsqueda eficiente del vecino más cercano

Para encontrar puntos cercanos sin comparar contra todos (fuerza bruta $O(n^2)$), `KDTree` indexa los puntos en una estructura de árbol y responde consultas de vecindad en $O(\log n)$:

```python
from scipy.spatial import KDTree

puntos = np.random.rand(10000, 2)
arbol = KDTree(puntos)

distancia, indice = arbol.query([0.5, 0.5])           # el punto más cercano a (0.5, 0.5)
distancias, indices = arbol.query([0.5, 0.5], k=5)    # los 5 más cercanos

# Todos los puntos dentro de un radio dado
indices_en_radio = arbol.query_ball_point([0.5, 0.5], r=0.1)
```

Este es, conceptualmente, el mecanismo interno detrás de `KNeighborsClassifier` en [[Python/Scikit-learn/01 - Introducción, Filosofía y la API Consistente|scikit-learn]].

## Diagrama de Voronoi y triangulación de Delaunay

```python
from scipy.spatial import Voronoi, Delaunay, ConvexHull

vor = Voronoi(puntos)        # regiones donde cada punto es el más cercano
tri = Delaunay(puntos)       # triangulación — útil para interpolación 2D irregular
hull = ConvexHull(puntos)    # el polígono convexo mínimo que contiene todos los puntos
```

## Clustering jerárquico

`scipy.spatial.distance` se combina con `scipy.cluster.hierarchy` para clustering aglomerativo (alternativa a K-Means de scikit-learn cuando se quiere ver el dendrograma completo):

```python
from scipy.cluster.hierarchy import linkage, dendrogram, fcluster

Z = linkage(puntos, method='ward')          # matriz de enlace jerárquico
clusters = fcluster(Z, t=3, criterion='maxclust')   # cortar el dendrograma en 3 clusters
```

## Ver también

- [[02 - Álgebra Lineal (scipy.linalg)]]
- [[Python/Scikit-learn/09 - Clustering y Reducción de Dimensionalidad]]
