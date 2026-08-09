---
Fecha de creación: 2026-04-18T18:00:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-04-18
---
[[Clase 10 - Valores y Vectores Propios|← Clase anterior]] | [[Clase 13 - Descomposición en Valores Singulares (SVD)|Clase siguiente →]]

# Diagonalización de Matrices
(construida directamente a partir de los [[Clase 10 - Valores y Vectores Propios|Valores y Vectores Propios]] de la Clase 10)

## ¿Qué es Diagonalizar una Matriz?

**Diagonalizar** una matriz cuadrada $A$ significa reescribirla como el producto:

$$
A = PDP^{-1}
$$

donde $D$ es una **matriz diagonal** (ver [[Clase 03 - Matrices en Python y MATLAB#Tipos de Matrices|Matriz Diagonal]] de la Clase 03) que contiene los valores propios de $A$ en su diagonal, y $P$ es la matriz cuyas columnas son los vectores propios correspondientes, en el mismo orden.

$$
D = \begin{bmatrix} \lambda_1 & 0 & \cdots \\ 0 & \lambda_2 & \cdots \\ \vdots & \vdots & \ddots \end{bmatrix}, \qquad P = \begin{bmatrix} | & | & \\ v_1 & v_2 & \cdots \\ | & | & \end{bmatrix}
$$

## Condición de Diagonalizabilidad

Una matriz $A_{n\times n}$ es diagonalizable **si y solo si** tiene $n$ vectores propios **linealmente independientes** — equivalentemente, si sus espacios propios (ver [[Clase 10 - Valores y Vectores Propios#Espacio Propio (_Eigenspace_)|Clase 10]]) en conjunto tienen dimensión $n$. Si un valor propio se repite pero no genera suficientes vectores propios independientes, la matriz **no es diagonalizable**.

## Ejemplo (continuación de la Clase 10)

Para $A = \begin{bmatrix} 4 & 1 \\ 2 & 3 \end{bmatrix}$, con $\lambda_1=5,\ v_1=(1,1)$ y $\lambda_2=2,\ v_2=(1,-2)$:

$$
P = \begin{bmatrix} 1 & 1 \\ 1 & -2 \end{bmatrix}, \qquad D = \begin{bmatrix} 5 & 0 \\ 0 & 2 \end{bmatrix}
$$

Puede verificarse que $A = PDP^{-1}$ calculando $P^{-1}$ (ver [[Clase 04 - Inversa, Adjunta y Sistemas Lineales|Matriz Inversa]] de la Clase 04) y multiplicando.

## ¿Por Qué Diagonalizar?

Una matriz diagonal es enormemente más fácil de manipular que una matriz general. La aplicación más directa: calcular **potencias de una matriz** ($A^k$), algo costoso en general pero trivial una vez diagonalizada:

$$
A^k = PD^kP^{-1}, \qquad D^k = \begin{bmatrix} \lambda_1^k & 0 & \cdots \\ 0 & \lambda_2^k & \cdots \\ \vdots & \vdots & \ddots \end{bmatrix}
$$

en lugar de multiplicar $A$ por sí misma $k$ veces, solo hay que elevar cada valor propio a la potencia $k$ — una diferencia enorme en costo computacional para $k$ grande. Esta misma idea es la base de algoritmos iterativos en machine learning (como métodos de potencias para aproximar el eigenvalor dominante) y de la resolución de sistemas de ecuaciones diferenciales lineales.

## Implementación en Python

```python
import numpy as np

A = np.array([[4, 1],
              [2, 3]])

valores, P = np.linalg.eig(A)
D = np.diag(valores)

# Reconstrucción de A
A_reconstruida = P @ D @ np.linalg.inv(P)
print("A reconstruida:\n", A_reconstruida)

# A^5 de forma eficiente
A5 = P @ np.diag(valores**5) @ np.linalg.inv(P)
print("A^5:\n", A5)
```

## Implementación en MATLAB

```matlab
A = [4 1; 2 3];
[P, D] = eig(A);

% Reconstrucción
A_reconstruida = P * D * inv(P);
disp(A_reconstruida)

% A^5 de forma eficiente
A5 = P * D^5 * inv(P);
disp(A5)
```

## Caso Especial: Matrices Simétricas

Cuando $A$ es una **matriz simétrica** ($A=A^T$, ver [[Clase 03 - Matrices en Python y MATLAB#Tipos de Matrices|Clase 03]]) — como lo es siempre una matriz de covarianza en ciencia de datos — está garantizado que:

- Todos sus valores propios son **números reales**.
- Sus vectores propios pueden elegirse **ortogonales** entre sí, es decir, $P$ puede ser una [[Clase 03 - Matrices en Python y MATLAB#Tipos de Matrices|matriz ortogonal]] ($P^{-1}=P^T$).

Esta propiedad es exactamente lo que hace posible el **PCA**: la matriz de covarianza de un dataset es simétrica por construcción, así que sus vectores propios (las componentes principales) siempre pueden tomarse perpendiculares entre sí, lo que se explora en la [[Clase 14 - Análisis de Componentes Principales (PCA)|Clase 14]].
