---
Fecha de creación: 2026-03-28T18:00:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-03-28
---
[[Clase 09 - Espacios y Subespacios Vectoriales|← Clase anterior]] | [[Clase 12 - Diagonalización de Matrices|Clase siguiente →]]

# Valores y Vectores Propios (Eigenvalues y Eigenvectors)
(usa los [[Clase 09 - Espacios y Subespacios Vectoriales#Subespacio|Subespacios]] de la Clase 09 — el espacio propio de cada eigenvalor es, precisamente, un subespacio vectorial)

## ¿Qué es un Vector Propio?

Para una matriz cuadrada $A_{n\times n}$, un **vector propio** (_eigenvector_) es un vector no nulo $v$ que, al ser transformado por $A$, **no cambia de dirección** — solo se escala:

$$
Av = \lambda v
$$

El escalar $\lambda$ que lo acompaña es su **valor propio** (_eigenvalue_) correspondiente. Geométricamente, retomando la [[Clase 07 - Geometría de Transformaciones Lineales|Clase 07]]: de todas las direcciones posibles del espacio, los vectores propios son las únicas direcciones que la transformación $A$ preserva; el resto de los vectores rotan o cambian de dirección al aplicarles $A$.

## Cálculo de Valores Propios

Reescribiendo $Av = \lambda v$ como $Av - \lambda v = 0$, es decir $(A-\lambda I)v = 0$. Para que exista una solución $v \neq 0$, la matriz $(A-\lambda I)$ debe ser singular (ver [[Clase 04 - Inversa, Adjunta y Sistemas Lineales#Matriz Singular|Matriz Singular]] de la Clase 04):

$$
\det(A-\lambda I) = 0
$$

Esta ecuación se llama **ecuación característica**, y es un polinomio de grado $n$ en $\lambda$ (el **polinomio característico**). Sus raíces son los valores propios de $A$.

### Ejemplo

Para $A = \begin{bmatrix} 4 & 1 \\ 2 & 3 \end{bmatrix}$:

$$
\det(A-\lambda I) = \det\begin{bmatrix} 4-\lambda & 1 \\ 2 & 3-\lambda \end{bmatrix} = (4-\lambda)(3-\lambda) - 2 = \lambda^2 -7\lambda+10 = 0
$$

$$
(\lambda-5)(\lambda-2) = 0 \implies \lambda_1 = 5,\ \lambda_2 = 2
$$

## Cálculo de Vectores Propios

Para cada valor propio $\lambda_i$ encontrado, se sustituye en $(A-\lambda_i I)v = 0$ y se resuelve el sistema lineal homogéneo resultante (usando [[Clase 04 - Inversa, Adjunta y Sistemas Lineales#Método de Gauss-Jordan para Inversa|Gauss-Jordan]], como en la Clase 04).

**Para $\lambda_1=5$:**

$$
(A-5I)v = \begin{bmatrix} -1 & 1 \\ 2 & -2 \end{bmatrix}v = 0 \implies -v_1+v_2=0 \implies v_1=v_2
$$

$$
v_1 = \begin{bmatrix}1\\1\end{bmatrix} \text{ (y cualquier múltiplo escalar)}
$$

**Para $\lambda_2=2$:**

$$
(A-2I)v = \begin{bmatrix} 2 & 1 \\ 2 & 1 \end{bmatrix}v = 0 \implies 2v_1+v_2=0
$$

$$
v_2 = \begin{bmatrix}1\\-2\end{bmatrix}
$$

## Espacio Propio (_Eigenspace_)

El conjunto de todos los vectores propios asociados a un mismo $\lambda_i$ (junto con el vector cero) forma un **subespacio vectorial** — el espacio propio de $\lambda_i$ — cumpliendo exactamente las tres propiedades de subespacio vistas en la [[Clase 09 - Espacios y Subespacios Vectoriales#Subespacio|Clase 09]].

## Implementación en Python

```python
import numpy as np

A = np.array([[4, 1],
              [2, 3]])

valores, vectores = np.linalg.eig(A)

print("Valores propios:", valores)
print("Vectores propios (columnas):\n", vectores)
```

## Implementación en MATLAB

```matlab
A = [4 1; 2 3];

[V, D] = eig(A);

disp('Vectores propios (columnas de V):');
disp(V)
disp('Valores propios (diagonal de D):');
disp(D)
```

## ¿Por Qué Importan en Ciencia de Datos?

Los valores y vectores propios son el fundamento matemático directo de:

- **PCA (Análisis de Componentes Principales):** los vectores propios de la matriz de covarianza de un dataset son las direcciones de máxima varianza (ver [[Clase 01 - Intro Álgebra Lineal Data Science#Conexión Directa con Ciencia de Datos|tabla de conexiones]] de la Clase 01).
- **PageRank de Google:** el ranking de páginas web es, esencialmente, el vector propio principal de la matriz de enlaces entre páginas.
- **Estabilidad de sistemas dinámicos:** en redes neuronales recurrentes, los valores propios de la matriz de pesos determinan si el sistema es estable o si las activaciones "explotan" o "desaparecen".
