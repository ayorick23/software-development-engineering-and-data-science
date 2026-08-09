---
Fecha de creación: 2026-03-14T18:21:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-03-07
---
[[Clase 05 - Repaso para el Parcial|← Clase anterior]] | [[Clase 08 - Núcleo, Imagen de Transformación Lineal|Clase siguiente →]]

# Geometría de las Transformaciones Lineales

Desde una perspectiva geométrica, una transformación lineal $T: \mathbb{R}^n \rightarrow \mathbb{R}^m$ es una regla que "mueve" los puntos del espacio. Lo más importante es que las transformaciones lineales siempre:

1. Mantienen el **origen** ($0,0$) fijo.
2. Mapean **líneas rectas** a otras líneas rectas.
3. Mantienen las líneas **paralelas** y equidistantes.

## Tipos de Transformaciones Básicas ($\mathbb{R}^2$)

- **Escalamiento (Scaling):** Estira o comprime el espacio a lo largo de los ejes.
	
	$$A = \begin{bmatrix} k_x & 0 \\ 0 & k_y \end{bmatrix}$$
    
- **Rotación (Rotation):** Gira todo el plano alrededor del origen un ángulo $\theta$.
    
    $$A = \begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}$$
    
- **Reflexión (Reflection):** Crea una imagen especular respecto a un eje (ej. respecto al eje X).
    
    $$A = \begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}$$
    
- **Cizallamiento (Shear):** Desplaza los puntos en una dirección proporcional a su distancia desde un eje.
    
    $$A = \begin{bmatrix} 1 & k \\ 0 & 1 \end{bmatrix}$$

## El Determinante y el Cambio de Área

Geométricamente, el valor absoluto del **Determinante** de la matriz de transformación $|det(A)|$ representa el factor de cambio de área:

- Si $|det(A)| = 2$, el área de cualquier figura se duplica.
- Si $|det(A)| = 0$, el espacio se colapsa en una línea o un punto (perdemos información, la matriz no es invertible).
- Si $det(A) < 0$, la transformación incluye una reflexión (se invierte la orientación del espacio).

## Transformaciones Lineales

Las **Transformaciones Lineales** son funciones que mapean vectores de un espacio vectorial $V$ a otro espacio vectorial $W$, preservando las operaciones de suma de vectores y multiplicación por un escalar. En Ciencia de Datos, son fundamentales para procesos como la reducción de dimensionalidad (PCA), rotaciones de datos y cambios de base.

## Definición Formal

Una función $T: V \rightarrow W$ es una transformación lineal si para todos los vectores $\mathbf{u}, \mathbf{v} \in V$ y cualquier escalar $c$, se cumplen las siguientes condiciones:

1. **Aditividad:** $T(\mathbf{u} + \mathbf{v}) = T(\mathbf{u}) + T(\mathbf{v})$
2. **Homogeneidad:** $T(c\mathbf{v}) = cT(\mathbf{v})$

## Representación Matricial

Cualquier transformación lineal entre espacios de dimensión finita puede representarse mediante una multiplicación de una matriz $A$ por un vector $\mathbf{x}$:

$$T(\mathbf{x}) = A\mathbf{x}$$

Donde $A$ es la **matriz estándar** de la transformación.

## Propiedades Clave

- **Núcleo (Kernel):** El conjunto de todos los vectores en $V$ que se mapean al vector cero en $W$.
- **Imagen (Rango):** El conjunto de todos los vectores en $W$ que son el resultado de aplicar $T$ a algún vector en $V$.
- **Isomorfismo:** Una transformación lineal que es biyectiva (inyectiva y sobreyectiva).

## Implementación en Python

Para Ciencia de Datos, utilizamos `numpy` para representar estas transformaciones como operaciones de matrices.

```python
import numpy as np

# Definimos un vector v
v = np.array([1, 2])

# Definimos una matriz de transformación A (Ejemplo: Rotación de 90°)
theta = np.radians(90)
A = np.array([
    [np.cos(theta), -np.sin(theta)],
    [np.sin(theta),  np.cos(theta)]
])

# Aplicamos la transformación lineal T(v) = Av
Tv = A @ v

print(f"Vector original: {v}")
print(f"Vector transformado: {Tv}")
```

## Implementación en MATLAB

MATLAB facilita la visualización y el cálculo directo de estas operaciones.

```matlab
% Definir el vector original
v = [1; 2];

% Definir matriz de escalamiento (Transformación Lineal)
% Escala x por 2 y y por 0.5
A = [2 0; 0 0.5];

% Aplicar la transformación
Tv = A * v;

disp('Vector Transformado:');
disp(Tv);

% Visualización rápida
quiver(0,0, v(1), v(2), 'r', 'LineWidth', 2); hold on;
quiver(0,0, Tv(1), Tv(2), 'b', 'LineWidth', 2);
legend('Original', 'Transformado');
grid on; axis equal;
```

## Aplicaciones en Ciencia de Datos

- **PCA (Análisis de Componentes Principales):** Utiliza transformaciones lineales para proyectar datos en direcciones de máxima varianza.
- **Procesamiento de Imágenes:** Las rotaciones, traslaciones y cambios de escala son transformaciones lineales (o afines).
- **Redes Neuronales:** Las capas densas calculan $f(Ax + b)$, donde $Ax$ es la parte de la transformación lineal.
