---
Fecha de creación: 2026-04-25T18:00:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-04-25
---
[[Clase 12 - Diagonalización de Matrices|← Clase anterior]] | [[Clase 14 - Análisis de Componentes Principales (PCA)|Clase siguiente →]]

# Descomposición en Valores Singulares (SVD)
(generaliza la [[Clase 12 - Diagonalización de Matrices|Diagonalización]] de la Clase 12 a matrices que no son cuadradas — como prácticamente cualquier dataset real, donde el número de observaciones casi nunca coincide con el número de variables)

## El Problema con la Diagonalización

La diagonalización de la Clase 12 solo funciona para matrices **cuadradas**, y ni siquiera todas ellas son diagonalizables. Sin embargo, en ciencia de datos casi ninguna matriz de datos es cuadrada: una tabla con $m$ observaciones (filas) y $n$ variables (columnas) es, en general, $m \times n$ con $m \neq n$. La **SVD** resuelve esto: **toda** matriz $A_{m\times n}$, sin excepción, admite la factorización:

$$
A = U\Sigma V^T
$$

## Las Tres Matrices de la SVD

- **$U$** ($m\times m$): matriz ortogonal cuyas columnas son los **vectores singulares izquierdos**.
- **$\Sigma$** ($m\times n$): matriz "diagonal" rectangular con los **valores singulares** $\sigma_1 \geq \sigma_2 \geq \dots \geq 0$ en la diagonal (siempre no negativos, ordenados de mayor a menor), y ceros en el resto.
- **$V^T$** ($n\times n$): transpuesta de una matriz ortogonal cuyas columnas ($V$) son los **vectores singulares derechos**.

## Relación con Eigenvalores y Eigenvectores

La SVD no aparece de la nada — se construye a partir de lo visto en las [[Clase 10 - Valores y Vectores Propios|Clases 10]] y [[Clase 12 - Diagonalización de Matrices|12]]:

- Las columnas de $V$ son los **vectores propios** de la matriz simétrica $A^TA$ ($n \times n$).
- Las columnas de $U$ son los vectores propios de $AA^T$ ($m \times m$).
- Los valores singulares son las raíces cuadradas de los valores propios (no negativos) de $A^TA$ (o, equivalentemente, de $AA^T$): $\sigma_i = \sqrt{\lambda_i}$.

Como $A^TA$ es siempre simétrica, la [[Clase 12 - Diagonalización de Matrices#Caso Especial: Matrices Simétricas|Clase 12]] garantiza que sus vectores propios son reales y ortogonales — precisamente lo que se necesita para que $V$ sea una matriz ortogonal.

## SVD Reducida (Truncada) y Compresión

La aplicación más importante de la SVD en ciencia de datos es que permite construir la **mejor aproximación posible** de $A$ usando solo los $k$ valores singulares más grandes (y sus vectores correspondientes), descartando el resto:

$$
A \approx A_k = U_k \Sigma_k V_k^T
$$

Como los valores singulares están ordenados de mayor a menor, los primeros capturan la mayor parte de la "información" (varianza) de la matriz original, y los últimos suelen corresponder a ruido. Esto es la base matemática de:

- **Compresión de imágenes:** una imagen es una matriz de píxeles; reteniendo solo los primeros $k$ valores singulares se reconstruye una versión muy similar con una fracción del almacenamiento.
- **Sistemas de recomendación:** factorizar la matriz usuario-producto para encontrar patrones latentes (ver la tabla de aplicaciones de la [[Clase 01 - Intro Álgebra Lineal Data Science#Aplicaciones Directa en Ciencia de Datos y Machine Learning|Clase 01]]).
- **Reducción de ruido:** eliminar los valores singulares pequeños filtra variaciones poco significativas de los datos.

## Implementación en Python

```python
import numpy as np

A = np.array([[3, 1],
              [1, 3],
              [1, 1]])  # matriz 3x2, no cuadrada

U, S, Vt = np.linalg.svd(A)

print("U:\n", U)
print("Valores singulares:", S)
print("V^T:\n", Vt)

# Reconstrucción con SVD truncada (k=1, solo el valor singular más grande)
k = 1
A_aprox = U[:, :k] @ np.diag(S[:k]) @ Vt[:k, :]
print("Aproximación de rango 1:\n", A_aprox)
```

## Implementación en MATLAB

```matlab
A = [3 1; 1 3; 1 1];

[U, S, V] = svd(A);

disp('Valores singulares:');
disp(diag(S))

% Aproximación de rango 1
k = 1;
A_aprox = U(:,1:k) * S(1:k,1:k) * V(:,1:k)';
disp(A_aprox)
```

## SVD como Herramienta General de PCA

Aunque el **Análisis de Componentes Principales** se puede calcular diagonalizando la matriz de covarianza (Clase 12), en la práctica las librerías de machine learning suelen calcular PCA usando SVD directamente sobre la matriz de datos centrada, porque es numéricamente más estable. Este es exactamente el puente hacia la [[Clase 14 - Análisis de Componentes Principales (PCA)|Clase 14]], la última del curso.
