---
tags: [scipy, algebra-lineal, cheat-sheet]
---

# 02 — Álgebra Lineal (`scipy.linalg`)

> Para las operaciones básicas de álgebra lineal sobre arrays, ver primero [[Python/NumPy/11 - Álgebra Lineal (np.linalg)|NumPy/np.linalg]]. Este archivo cubre solo lo que `scipy.linalg` añade o hace distinto.

## `scipy.linalg` vs `numpy.linalg` — por qué existen los dos

NumPy incluye `np.linalg` para las operaciones más comunes (inversa, determinante, autovalores, sistemas lineales). `scipy.linalg` **contiene todo lo de `np.linalg` y más**: siempre está compilado contra BLAS/LAPACK (NumPy a veces usa una versión reducida), soporta matrices más generales (no cuadradas, complejas) y añade descomposiciones adicionales.

Regla práctica: si el proyecto ya depende de SciPy, usar `scipy.linalg` en vez de `numpy.linalg` — es un superconjunto estrictamente más completo y no más lento.

```python
import numpy as np
from scipy import linalg

A = np.array([[3, 1], [1, 2]])
b = np.array([9, 8])

linalg.solve(A, b)        # resuelve Ax = b
linalg.inv(A)             # inversa
linalg.det(A)             # determinante
```

## Descomposiciones

```python
# LU: A = P @ L @ U
P, L, U = linalg.lu(A)

# QR: A = Q @ R
Q, R = linalg.qr(A)

# Cholesky (A debe ser simétrica definida positiva): A = L @ L.T
L = linalg.cholesky(A, lower=True)

# SVD: A = U @ diag(s) @ Vt
U, s, Vt = linalg.svd(A)

# Autovalores y autovectores
valores, vectores = linalg.eig(A)          # caso general (puede dar complejos)
valores, vectores = linalg.eigh(A)         # A simétrica/hermítica -> valores reales, más rápido
```

| Descomposición | Uso típico |
|---|---|
| LU | Resolver sistemas lineales repetidamente con la misma matriz |
| QR | Mínimos cuadrados, ortogonalización |
| Cholesky | Sistemas simétricos definidos positivos (más rápida que LU cuando aplica) |
| SVD | Reducción de dimensionalidad, pseudo-inversa, compresión |
| `eigh` | Matrices de covarianza, PCA manual |

## Matrices especiales y estructuradas

`scipy.linalg` incluye solvers optimizados para matrices con estructura conocida, mucho más rápidos que el caso general:

```python
# Matriz banda (band matrix) — común en discretizaciones de ecuaciones diferenciales
linalg.solve_banded((1, 1), ab, b)

# Matriz de Toeplitz
T = linalg.toeplitz([1, 2, 3])

# Norma matricial/vectorial
linalg.norm(A, ord='fro')   # norma de Frobenius
```

## Mínimos cuadrados y sistemas sobre/subdeterminados

`linalg.solve` exige una matriz cuadrada no singular. Para sistemas rectangulares (más ecuaciones que incógnitas, típico en regresión):

```python
# Ax = b, A de forma (m, n) con m != n
x, residuales, rank, sv = linalg.lstsq(A, b)
```

Esto es, conceptualmente, lo que hace por debajo una regresión lineal por mínimos cuadrados ordinarios.

## Matrices dispersas: cuándo saltar a `scipy.sparse`

`scipy.linalg` opera sobre matrices **densas** (todos los elementos almacenados, incluyendo ceros). Si la matriz es grande y mayoritariamente ceros (miles de columnas, pocas no-cero por fila — típico en NLP con one-hot/TF-IDF, grafos, sistemas de recomendación), usar [[09 - Matrices Dispersas (scipy.sparse)|scipy.sparse]] en su lugar: el equivalente denso simplemente no cabría en memoria.

## Ver también

- [[Python/NumPy/11 - Álgebra Lineal (np.linalg)|NumPy — np.linalg]]
- [[09 - Matrices Dispersas (scipy.sparse)]]
- [[12 - Integración con el Ecosistema]]
