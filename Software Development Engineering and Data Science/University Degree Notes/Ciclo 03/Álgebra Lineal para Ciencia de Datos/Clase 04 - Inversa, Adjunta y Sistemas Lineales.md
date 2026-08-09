---
Fecha de creación: 2026-02-22T14:59:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-02-14
---
[[Clase 03 - Matrices en Python y MATLAB|← Clase anterior]] | [[Clase 07 - Geometría de Transformaciones Lineales|Clase siguiente →]]

# Matriz Inversa, Adjunta, Gauss-Jordan y Sistemas Lineales

Esta clase fue un repaso integral de los métodos algebraicos clásicos para resolver sistemas lineales y calcular la inversa de una matriz. Aunque muchos de estos procedimientos hoy se automatizan con software, entenderlos es fundamental para comprender cómo funcionan los algoritmos numéricos utilizados en Ciencia de Datos y Machine Learning.

## Matriz Inversa

Sea AAA una matriz cuadrada $n \times n$. Se dice que $A^{-1}$ es la **matriz inversa** de $A$ si:

$$
AA^{-1} = A^{-1}A = I
$$

donde $I$ es la matriz identidad.

### Condición para que exista inversa

Una matriz tiene inversa si y solo si:

$$
\det(A) \neq 0
$$

Cuando:

- $\det(A)\neq 0$ - matriz regular o invertible
- $\det(A) = 0$ - matriz singular (no tiene inversa)

## Determinante y Ley de Sarrus (Solo 3×3)

Para matrices 2×2:

$$
A = \begin{pmatrix} a & b \\ c & d \end{pmatrix}
$$

$$
\det(A) = ad - bc
$$

### Ley de Sarrus (3×3)

Sea:

$$
A = \begin{pmatrix} a&b&c\\ d&e&f \\ g&h&i \end{pmatrix}
$$

Se repiten las dos primeras columnas y se calcula:

$$
\det(A) = aei + bfg + cdh - ceg - bdi - afh
$$

 >[!IMPORTANT] Solo funciona para matrices 3×3
 
## Matriz Adjunta y Cofactores
 
Para encontrar la inversa usando determinantes:

$$
A^{-1} = \frac{1}{\det(A)}Adj(A)
$$

**Paso 1:** Menor $M_{ij}$

Eliminar fila $i$ y columna $j$.

**Paso 2:** Cofactor

$$
C_{ij} = (-1)^{i+j}M_{ij}
$$

**Paso 3:** Matriz Adjunta

Es la transpuesta de la matriz de cofactores:

$$
Adj(A) = C^T
$$

## Método de Gauss-Jordan para Inversa

En lugar de usar determinantes, usamos operaciones elementales por filas.

### Procedimiento

1. Construir matriz aumentada:

$$
[A|I]
$$

2. Aplicar operaciones elementales hasta convertir $A$ en identidad.    
3. El lado derecho se convierte en $A^{-1}$.

## Sistemas de Ecuaciones Lineales

Un sistema lineal puede escribir se como:

$$
AX = B
$$

Donde:

- $A$ - matriz de coeficientes
- $X$ - vector incógnitas
- $B$ - vector términos independientes

Si $A$ es invertible:

$$
X = A^{-1}B
$$

## Regla de Cramer

Se usa cuando:

- Matriz cuadrada
- Determinante distinto de cero

Para sistema 2×2:

$$
AX = B
$$
$$
x_i = \frac{\det(A_i)}{\det(A)}
$$

Donde $A_i$​ se obtiene reemplazando la columna $i$ por el vector $B$.

## Implementación en Python

```python
import numpy as np

A = np.array([[4,7],
              [2,6]])

# Determinante
det_A = np.linalg.det(A)
print("Determinante:", det_A)

# Inversa
A_inv = np.linalg.inv(A)
print("Inversa:\n", A_inv)

# Verificación
print("A * A_inv:\n", np.dot(A, A_inv))
```

### Sistema Lineal

```python
A = np.array([[2,1],
              [5,3]])

B = np.array([1,2])

# Resolver sistema
X = np.linalg.solve(A,B)
print("Solución:", X)
```

## Implementación en MATLAB

```matlab
A = [4 7; 2 6];

detA = det(A)

A_inv = inv(A)

A * A_inv
```

### Sistema Lineal

```matlab
A = [2 1; 5 3];
B = [1; 2];

X = A\B
```

>[!IMPORTANT] En MATLAB es mejor usar `A\B` que `inv(A)*B` por estabilidad numérica.
