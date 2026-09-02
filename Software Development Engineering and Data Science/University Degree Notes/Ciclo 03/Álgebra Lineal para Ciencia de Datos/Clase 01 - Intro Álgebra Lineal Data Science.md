---
Fecha de creación: 2026-01-24T17:45:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-01-24
---
[[Clase 02 - Vectores en Python y MATLAB|Clase siguiente →]]

# Introducción al Álgebra Lineal para Ciencia de Datos

La **álgebra lineal** es una de las bases matemáticas más importantes para la **ciencia de datos**, la **inteligencia artificial** y el **machine learning**, ya que permite representar, transformar y analizar grandes volúmenes de datos de forma eficiente.

En términos simples:

> La mayoría de los datos que usan los algoritmos modernos se representan como **vectores, matrices y tensores**, y las operaciones entre ellos son exactamente las que estudia el álgebra lineal.

## ¿Por qué es tan importante el Álgebra Lineal en Ciencia de Datos?

El álgebra lineal permite:

- Representar datos de forma estructurada (vectores y matrices)
- Realizar transformaciones sobre datos (rotaciones, escalados, proyecciones)
- Modelar relaciones entre variables
- Optimizar modelos de machine learning
- Reducir dimensiones de datos (PCA, SVD)
- Resolver sistemas de ecuaciones

## Aplicaciones Directa en Ciencia de Datos y Machine Learning

| Área                                    | Uso del Álgebra Lineal                               |
| --------------------------------------- | ---------------------------------------------------- |
| Regresión lineal                        | Resolver sistemas de ecuaciones y mínimos cuadrados  |
| Redes neuronales                        | Propagación de activaciones y actualización de pesos |
| NLP (procesamiento de lenguaje natural) | Representación vectorial de palabras (embeddings)    |
| Visión por computadora                  | Transformaciones de imágenes como matrices           |
| Reducción de dimensionalidad            | PCA, SVD                                             |
| Sistemas de recomendación               | Factorización de matrices                            |

## Software de Apoyo

### Python
(ver [[Introduction to Python]])

Es el lenguaje principal en ciencia de datos por su ecosistema:

- **[[Python/NumPy/01 - Introducción y Arquitectura Interna|Numpy]]** → Vectores y matrices
- **SciPy** → Álgebra lineal avanzada
- **[[Python/Pandas/01 - Introducción y Arquitectura Interna|Pandas]]** → Manipulación de datos tabulares
- **Scikit-learn** → Machine learning

**Ejemplo básico en Python:**

```python
import numpy as np

v = np.array([2, 4, 6])
print(v * 2)   # Escalamiento vectorial
```

### MATLAB

Muy usado en ingeniería y matemática aplicada:

- Maneja matrices de forma nativa
- Ideal para prototipos matemáticos rápidos
- Usado en señales, control, optimización y simulación

**Ejemplo básico en Matlab:**

```matlab
v = [2 4 6];
v * 2
```

## ¿Qué es un Vector?
(ver [[Clase 01 - Sistemas Coordenados Cartesianos]])

Un **vector** es una estructura matemática que representa:

- Una magnitud
- Una dirección
- Un conjunto ordenado de números

En ciencia de datos:

> Un vector representa una **observación**, un **registro**, una **imagen**, un **usuario**, un **documento**, etc.

### Representación de un Vector

Vector en $ℝ^3$:

$$
\vec{v} = \begin{bmatrix}2\\4\\6\end{bmatrix}
$$

También puede escribirse como:

$$\vec{v}=(2,4,6)$$

### Tipos de Vectores

| Tipo            | Descripción                           |
| --------------- | ------------------------------------- |
| Vector fila     | $[1 2 3]$                             |
| Vector columna  | $\begin{bmatrix}1\\2\\3\end{bmatrix}$ |
| Vector nulo     | $(0,0,...,0)$                         |
| Vector unitario | Norma igual a 1                       |
| Vector posición | Representa un punto desde el origen   |

### Operaciones Básicas con Vectores
(ver [[Clase 02 - Vectores en Python y MATLAB]])
(ver [[Python/NumPy/10 - Operaciones Matemáticas y Vectorización|Operaciones Matemáticas]])

#### Suma de Vectores

$$
\vec{a} = (1,2), \vec{b} = (3,4) \newline
\vec{a} + \vec{b} = (4,6)
$$

#### Resta de Vectores

$$
\vec{a} - \vec{b} = (-2,-2)
$$

**En Python:**

```python
a = np.array([1,2])
b = np.array([3,4])

suma = a + b
resta = a - b
```

**En Matlab:**

```matlab
a = [1, 2]
b = [3, 4]

suma = a + b
resta = a - b
```

#### Multiplicación por Escalar

$$
2\vec{v} = 2(1, 3) = (2,6)
$$

**En Python:**

```python
escalar = 2 * v
```

**En Matlab:**

```matlab
escalar = 2 * v
```

#### Norma (Longitud) de un Vector

$$
∥\vec{v}∥=\sqrt{v_1^2​+v_2^2​+...+v_n^2}​
$$

**Ejemplo:**

$$
\vec{v}=(3,4) \newline
​∥\vec{v}∥ = 5
$$
**En Python:**

```python
norma = np.linalg.norm(v)
```

**En Matlab:**

```matlab
norma = norm(v)
```

#### Producto Punto (Producto Escalar)

$$
\vec{a} \cdot \vec{b} = a_1b_2+a_2b_2+...
$$

**Usos:**

- Medir similitud
- Calcular proyecciones
- Encontrar ángulos

**Ejemplo:**

$$
(1,2) \cdot (3,4) = 11
$$

**En Python:**

```python
prod_punto = np.dot(a, b)
```

**En Matlab:**

```matlab
prod_punto = dot(a, b)
```

## Conexión Directa con Ciencia de Datos

|Concepto Matemático|En Ciencia de Datos|
|---|---|
|Vector|Registro de datos|
|Matriz|Dataset completo|
|Producto punto|Similaridad|
|Norma|Magnitud de una observación|
|Proyección|Reducción de dimensiones|
|Transformación lineal|Cambio de espacio de características|
