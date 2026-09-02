---
tags: [numpy, python, data-science, linear-algebra, cheat-sheet]
---

# 11 — Álgebra Lineal (np.linalg)

> Continúa de [[10 - Operaciones Matemáticas y Vectorización]]. Base matemática detrás de casi todo el Machine Learning clásico y las redes neuronales.

## Producto matricial

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

A @ B                    # producto matricial — operador preferido
np.matmul(A, B)            # equivalente explícito
np.dot(A, B)                 # también funciona para matrices 2D, pero @ es más legible e idiomático
```

**`@` vs `*`:** `A * B` multiplica **elemento por elemento** (requiere broadcasting compatible); `A @ B` es el producto matricial real de álgebra lineal — confundir ambos es un error común y silencioso (ambos "funcionan" sin error si las formas son cuadradas del mismo tamaño, pero dan resultados matemáticamente distintos).

## Transpuesta, inversa y determinante

```python
A.T                          # transpuesta (ver 07)
np.linalg.inv(A)               # matriz inversa — requiere A cuadrada y no singular
np.linalg.det(A)                 # determinante
np.linalg.matrix_rank(A)           # rango de la matriz
```

## Resolver sistemas de ecuaciones lineales

```python
# Resolver Ax = b
A = np.array([[3, 1], [1, 2]])
b = np.array([9, 8])
x = np.linalg.solve(A, b)         # forma preferida — más estable numéricamente que calcular inv(A) @ b
```

`np.linalg.solve()` es preferible a `np.linalg.inv(A) @ b` tanto por rendimiento como por estabilidad numérica — calcular la inversa explícita acumula más error de punto flotante que resolver el sistema directamente.

## Valores y vectores propios (eigenvalues/eigenvectors)

```python
valores_propios, vectores_propios = np.linalg.eig(A)
valores_propios_sim, vectores_propios_sim = np.linalg.eigh(A)   # versión optimizada para matrices SIMÉTRICAS
```

`eigh()` (la "h" es de *Hermitian*) asume que la matriz es simétrica (o Hermitiana en el caso complejo) y aprovecha esa propiedad para ser más rápida y numéricamente más estable que `eig()` — usar `eigh()` siempre que se sepa que la matriz de entrada es simétrica (ej. una matriz de covarianza).

## Descomposición en Valores Singulares (SVD)

```python
U, S, Vt = np.linalg.svd(A)
```

Base matemática de PCA y de la reducción de dimensionalidad — la explicación conceptual completa, con la conexión a covarianza y componentes principales, está desarrollada paso a paso en [[Clase 13 - Descomposición en Valores Singulares (SVD)|Ciclo 03 — SVD]] y [[Clase 14 - Análisis de Componentes Principales (PCA)|Ciclo 03 — PCA]].

## Normas de vectores y matrices

```python
np.linalg.norm(a)                     # norma euclidiana (L2) por default
np.linalg.norm(a, ord=1)                # norma L1 (suma de valores absolutos)
np.linalg.norm(matriz, axis=1)            # norma de cada FILA por separado
```

Las normas L1/L2 son la base matemática de la regularización Lasso/Ridge en modelos lineales — ver [[06 - Modelos Lineales - Sintaxis y API|Scikit-learn]].

## Producto exterior, traza y otras operaciones

```python
np.outer(a, b)          # producto exterior — matriz de todos los productos posibles ai*bj
np.trace(A)                # suma de la diagonal principal
np.diag(A)                   # extrae la diagonal como vector (ver también 02)
```

## Ver también

- [[10 - Operaciones Matemáticas y Vectorización]]
- [[02 - Creación de Arrays]]
- [[06 - Modelos Lineales - Sintaxis y API|Scikit-learn]]
- [[Clase 01 - Intro Álgebra Lineal Data Science|Ciclo 03 — Álgebra Lineal para Ciencia de Datos]] — el desarrollo matemático completo detrás de estas operaciones
