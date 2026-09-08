---
tags: [scipy, matrices-dispersas, cheat-sheet]
---

# 09 — Matrices Dispersas (`scipy.sparse`)

## El problema que resuelve

Una matriz **dispersa** (sparse) es una matriz donde la gran mayoría de los elementos son cero. Un array denso de NumPy almacena **todos** los elementos, ceros incluidos — para una matriz de un millón de columnas donde cada fila tiene solo 10 valores no-cero (típico en texto vectorizado con TF-IDF, grafos, sistemas de recomendación), almacenarla como densa desperdiciaría más del 99.99% de la memoria en ceros.

`scipy.sparse` almacena solo los valores no-cero y sus posiciones, y reimplementa las operaciones de álgebra lineal para aprovechar esa estructura.

## Formatos de almacenamiento

```python
import numpy as np
from scipy import sparse

denso = np.array([[0, 0, 3], [4, 0, 0], [0, 5, 0]])

# CSR (Compressed Sparse Row) — eficiente para operaciones fila por fila y multiplicación matriz-vector
csr = sparse.csr_matrix(denso)

# CSC (Compressed Sparse Column) — eficiente para operaciones columna por columna
csc = sparse.csc_matrix(denso)

# COO (Coordinate) — más simple de construir, ideal para ensamblar la matriz elemento por elemento
coo = sparse.coo_matrix(denso)
```

| Formato | Mejor para | Peor para |
|---|---|---|
| CSR | Multiplicación matriz-vector, slicing de filas | Construcción incremental, slicing de columnas |
| CSC | Slicing de columnas, algunos solvers | Slicing de filas |
| COO | Construir la matriz desde cero (tripletas fila/columna/valor) | Casi cualquier operación aritmética — convertir a CSR/CSC primero |

Patrón habitual: construir en COO, convertir a CSR para operar.

```python
coo.tocsr()   # conversión entre formatos
```

## Construir una matriz dispersa directamente

```python
filas = [0, 1, 2]
columnas = [2, 0, 1]
valores = [3, 4, 5]

sparse.coo_matrix((valores, (filas, columnas)), shape=(3, 3)).toarray()
```

## Operaciones básicas

```python
csr @ csr.T          # multiplicación matricial — sintaxis idéntica a NumPy
csr.multiply(csr)    # multiplicación elemento a elemento
csr.toarray()        # convertir a denso — solo si el tamaño lo permite
csr.nnz              # número de elementos no-cero almacenados
csr.data             # array plano de los valores no-cero
```

## Álgebra lineal dispersa (`scipy.sparse.linalg`)

Versión de `scipy.linalg` que opera directamente sobre matrices dispersas sin densificarlas:

```python
from scipy.sparse.linalg import spsolve, eigs

x = spsolve(csr, b)              # resolver Ax = b sin convertir a denso
valores, vectores = eigs(csr, k=3)   # los k autovalores/autovectores más relevantes (no todos)
```

## Cuándo usar disperso vs denso

Regla práctica: si más del ~50-70% de los elementos son cero y la matriz es grande, el formato disperso ahorra memoria y tiempo de cómputo de forma significativa. Para matrices pequeñas o mayoritariamente densas, el overhead del formato disperso no compensa — un array de NumPy normal es más simple y suficientemente rápido.

## Ver también

- [[02 - Álgebra Lineal (scipy.linalg)]]
- [[Python/Scikit-learn/01 - Introducción, Filosofía y la API Consistente|Scikit-learn]] — `TfidfVectorizer` y similares devuelven matrices dispersas de este mismo tipo
