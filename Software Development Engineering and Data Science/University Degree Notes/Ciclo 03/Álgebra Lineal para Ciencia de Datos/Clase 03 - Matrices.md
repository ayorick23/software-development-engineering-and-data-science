---
Fecha de creación: 2026-02-07T18:04:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-02-07
---
# Matrices

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

### Matriz Fila

Una sola fila.

$$
\begin{bmatrix}1 2 3\end{bmatrix}
$$

### Matriz Columna

Una sola columna.

$$
\begin{bmatrix}1\\2\\3\end{bmatrix}
$$

### Matriz Cuadrada

Mismo número de filas y columnas ($n \times n$).

### Matriz Nula

Todos sus elementos son cero.

### Matriz Diagonal

Solo la diagonal principal es diferente de cero.

### Matriz Identidad $I$

Diagonal con unos en la diagonal principal.

$$
I = \begin{bmatrix}1 0\\0 1\end{bmatrix}
$$

### Matriz Triangular Superior / Inferior

Ceros debajo o encima de la diagonal.

### Matriz Simétrica



Concepto de matriz
Tipos de matrices
Operaciones con matrices
Sus respectivas funciones en python y matlab
Igualdades con las matrices ejemplos

Operaciones con matrices (numpy, matlab)
Sumas, restas, multiplicacion, potencia
``disp`` en matlab
Matriz inversa
matriz regular
matriz singular
