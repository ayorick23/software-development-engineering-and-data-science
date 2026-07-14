---
Fecha de creación: 2026-02-07T18:04:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-02-07
---
# Matrices en Python y MATLAB

## Matrices

Las **matrices** son estructuras matemáticas que organizan datos en forma de **filas y columnas**. En ciencia de datos, prácticamente **todo dataset es una matriz**, donde:

- Cada fila = una observación
- Cada columna = una variable o característica

## Importancia de las Matrices en Ciencia de Datos

Las matrices permiten:

- Representar datasets completos
- Aplicar transformaciones a datos
- Resolver sistemas de ecuaciones
- Entrenar modelos de machine learning
- Realizar operaciones masivas de forma eficiente

Ejemplos reales:

- Regresión lineal → $X\beta = y$
- Redes neuronales → multiplicaciones matriciales
- PCA → descomposición matricial

## Concepto de Matriz

Una matriz AAA de tamaño $m \times n$ se escribe:
$$
A = \begin{bmatrix} a_{11} & a_{12} & \dots & a_{1n} \\ a_{21} & a_{22} & \dots & a_{2n} \\ \vdots & \vdots & \ddots & \vdots \\ a_{m1} & a_{m2} & \dots & a_{mn} \end{bmatrix}
$$

Donde:

- $m$ = número de filas
- $n$ = número de columnas

## Tipos de Matrices

- **Matriz Fila:** Una sola fila.

$$
\begin{bmatrix}1 & 2 & 3\end{bmatrix}
$$

- **Matriz Columna:** Una sola columna.

$$
\begin{bmatrix}1\\2\\3\end{bmatrix}
$$

- **Matriz Cuadrada:** Mismo número de filas y columnas ($n \times n$).
- **Matriz Nula:** Todos sus elementos son cero.
- **Matriz Diagonal:** Solo la diagonal principal es diferente de cero.

- **Matriz Identidad $I$:** Diagonal con unos en la diagonal principal.

$$
I = \begin{bmatrix}1 & 0\\0 & 1\end{bmatrix}
$$

- **Matriz Triangular Superior / Inferior:** Ceros debajo o encima de la diagonal.
- **Matriz Simétrica:**

$$
A = A^T
$$

- **Matriz Ortogonal:**

$$
A^T A = I
$$

- **Matriz Regular (Invertible):** Tiene inversa.
- **Matriz Singular:** No tiene inversa (determinante = 0).
- **Matriz Rectangular:** Número de filas distinto al de columnas.

## Operaciones con Matrices

### Igualdad de Matrices

Dos matrices son iguales si:

- Tienen el mismo tamaño
- Todos sus elementos correspondientes son iguales

$$
A = B <==> a_{ij} = b_{ij}
$$

### Suma y Resta de Matrices

Solo si tienen el mismo tamaño.

$$
C = A + B => c_{ij} = a_{ij} + b_{ij}
$$

$$
A - B = a_{ij} - b_{ij}
$$

```python
A + B
A - B
```

```matlab
A + B
A - B
```

### Multiplicación por Escalar

$$
kA = (ka_{ij})
$$

```python
2 * A
```

```matlab
2 * A
```

### Multiplicación de Matrices

Solo es posible si:

$$
A_{m\times n}\cdot B_{n\times p} = C_{m\times p}
$$

```python
A @ B
```

```matlab
A * B
```

### Potencia de una Matriz

Solo para matrices cuadradas:

$$
A^k = A\cdot A \cdot\cdot\cdot A
$$

```python
np.linalg.matrix_power(A, 2)
```

```matlab
A^2
```

### Transpuesta

Intercambia filas por columnas.

$$
A^T
$$

```python
A.T
```

```matlab
A'
```

### Matriz Inversa

La inversa de una matriz cuadrada $A$ es otra matriz $A^{-1}$ tal que:

$$
AA^{-1} = A^{-1}A = I
$$

Solo existe si:

$$
det(A) \neq 0
$$

```python
np.linalg.inv(A)
```

```matlab
inv(A)
```

### Matriz Singular

Una matriz es singular si:

$$
det(A) = 0
$$

Esto implica:

- No tiene inversa
- Sus filas o columnas son linealmente dependientes

**Ejemplo:**

$$
\begin{bmatrix}1 & 2\\2 & 4\end{bmatrix}
$$

### Matriz Regular (Invertible)

Una matriz es regular si:

- Es cuadrada
- Su determinante es distinto de cero
- Tiene inversa

## Relación con Sistemas de Ecuaciones

Sistema:

$$
A\vec{x} = \vec{b} => \vec{x} = A^{-1}\vec{b}
$$

```python
x = np.linalg.solve(A, b)
```

```matlab
x = A\b
```
### Funciones Clave en Python con ``numpy``

```python
import numpy as np

A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

A + B
A - B
A @ B
A.T
np.linalg.det(A)
np.linalg.inv(A)
np.linalg.matrix_power(A, 2)
np.eye(2) # Identidad
np.zeros((2, 2))
np.ones((2, 2))
```

### Funciones Clave en MATLAB

```matlab
A = [1 2; 3 4];
B = [5 6; 7 8];

A + B
A - B
A * B
A'
det(A)
inv(A)
A^2
eye(2)
zeros(2)
ones(2)
disp(A)
```

## Función ``disp`` en MATLAB

Se usa para mostrar resultados en consola sin imprimir el nombre de la variable.

```matlab
A = [1 2; 3 4]
disp(A)
```

Salida:

```shell
1     2
3     4
```

Muy útil en scripts largos para mostrar resultados parciales.
