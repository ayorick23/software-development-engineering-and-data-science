---
Fecha de creación: 2026-05-02T18:00:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-05-02
---
[[Clase 13 - Descomposición en Valores Singulares (SVD)|← Clase anterior]]

# Análisis de Componentes Principales (PCA)
(la aplicación que motivó todo el curso — mencionada desde la [[Clase 01 - Intro Álgebra Lineal Data Science#Aplicaciones Directa en Ciencia de Datos y Machine Learning|Clase 01]] — y que reúne vectores, matrices, [[Clase 10 - Valores y Vectores Propios|valores propios]] y [[Clase 13 - Descomposición en Valores Singulares (SVD)|SVD]] en una sola técnica)

## El Problema: Reducción de Dimensionalidad

Un dataset real puede tener decenas o cientos de variables (columnas). Muchas de ellas están correlacionadas entre sí y, en cierto sentido, son "redundantes". El **PCA** busca encontrar un número menor de nuevas variables — las **componentes principales** — que resuman la mayor cantidad posible de la variabilidad (información) original, permitiendo visualizar, comprimir o acelerar el entrenamiento de modelos sobre datos de alta dimensión.

## Idea Geométrica

Retomando la [[Clase 07 - Geometría de Transformaciones Lineales|Clase 07]]: PCA busca un nuevo sistema de ejes coordenados para los datos, rotado respecto al original, de forma que:

- El **primer eje** (primera componente principal) apunte en la dirección donde los datos tienen **mayor varianza**.
- El **segundo eje** sea perpendicular al primero y capture la mayor varianza restante.
- Y así sucesivamente para cada eje siguiente.

Al proyectar los datos sobre solo los primeros $k$ ejes (en vez de los $n$ originales), se obtiene una versión de menor dimensión que conserva la mayor parte posible de la información.

## Procedimiento Paso a Paso

### 1. Centrar los Datos

Restar la media de cada variable, para que el dataset quede centrado en el origen:

$$
X_{centrado} = X - \bar{X}
$$

### 2. Calcular la Matriz de Covarianza

$$
C = \frac{1}{n-1}X_{centrado}^T X_{centrado}
$$

Esta matriz $C$ ($n\times n$, con $n$ = número de variables) es **siempre simétrica** — la condición exacta que, según la [[Clase 12 - Diagonalización de Matrices#Caso Especial: Matrices Simétricas|Clase 12]], garantiza valores propios reales y vectores propios ortogonales.

### 3. Calcular Valores y Vectores Propios de la Covarianza

Usando lo visto en la [[Clase 10 - Valores y Vectores Propios|Clase 10]]:

$$
Cv_i = \lambda_i v_i
$$

- Cada **vector propio** $v_i$ es una **componente principal**: una dirección en el espacio original de variables.
- Cada **valor propio** $\lambda_i$ representa **cuánta varianza** de los datos captura esa componente.

### 4. Ordenar y Seleccionar Componentes

Se ordenan los valores propios de mayor a menor (igual que los valores singulares en la [[Clase 13 - Descomposición en Valores Singulares (SVD)|Clase 13]]) y se seleccionan los $k$ vectores propios correspondientes a los $k$ valores propios más grandes.

### 5. Proyectar los Datos

$$
X_{PCA} = X_{centrado} \cdot V_k
$$

donde $V_k$ es la matriz cuyas columnas son los $k$ vectores propios seleccionados. El resultado $X_{PCA}$ tiene solo $k$ columnas en vez de las $n$ originales.

## Varianza Explicada

La proporción de la información original retenida por las primeras $k$ componentes se mide como:

$$
\text{Varianza explicada} = \frac{\sum_{i=1}^k \lambda_i}{\sum_{i=1}^n \lambda_i}
$$

Es común elegir $k$ como el menor número de componentes que retenga, por ejemplo, el 95% de la varianza total — un balance entre reducir dimensiones y no perder información relevante.

## PCA vía SVD (el Enfoque Usado en la Práctica)

Como se adelantó en la [[Clase 13 - Descomposición en Valores Singulares (SVD)#SVD como Herramienta General de PCA|Clase 13]], en vez de calcular explícitamente la matriz de covarianza y diagonalizarla, las librerías modernas aplican SVD directamente sobre $X_{centrado}$:

$$
X_{centrado} = U\Sigma V^T
$$

Los vectores singulares derechos ($V$) son exactamente las componentes principales, y los valores singulares se relacionan con los valores propios de la covarianza mediante $\lambda_i = \dfrac{\sigma_i^2}{n-1}$. Este enfoque es más estable numéricamente y evita calcular $X^TX$ explícitamente.

## Implementación en Python

```python
import numpy as np

# Dataset de ejemplo: 5 observaciones, 3 variables
X = np.array([
    [2.5, 2.4, 1.0],
    [0.5, 0.7, 0.3],
    [2.2, 2.9, 1.2],
    [1.9, 2.2, 0.9],
    [3.1, 3.0, 1.5]
])

# 1. Centrar los datos
X_centrado = X - np.mean(X, axis=0)

# 2. SVD directamente sobre los datos centrados
U, S, Vt = np.linalg.svd(X_centrado)

# 3. Proyectar sobre las primeras k=2 componentes principales
k = 2
X_pca = X_centrado @ Vt[:k].T

print("Datos originales (3 variables):\n", X)
print("Datos reducidos (2 componentes principales):\n", X_pca)

# Varianza explicada
varianza_explicada = (S**2) / np.sum(S**2)
print("Varianza explicada por componente:", varianza_explicada)
```

### Con Scikit-learn (uso típico en la industria)

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)

print("Varianza explicada:", pca.explained_variance_ratio_)
```

## Aplicaciones

- **Visualización:** reducir datos de decenas de variables a 2 o 3 componentes para graficarlos.
- **Compresión y velocidad de entrenamiento:** menos variables significa modelos de machine learning más rápidos de entrenar.
- **Eliminación de ruido y multicolinealidad:** al descartar componentes de baja varianza, se descarta también buena parte del ruido.
- **Preprocesamiento para otros algoritmos:** muchos modelos (clustering, regresión) funcionan mejor con variables no correlacionadas, que es justamente lo que producen las componentes principales.

## Cierre del Curso

Este curso construyó, paso a paso, las herramientas necesarias para llegar hasta aquí: los [[Clase 02 - Vectores en Python y MATLAB|vectores]] y [[Clase 03 - Matrices en Python y MATLAB|matrices]] que representan los datos, los [[Clase 04 - Inversa, Adjunta y Sistemas Lineales|sistemas lineales]] y [[Clase 07 - Geometría de Transformaciones Lineales|transformaciones]] que los manipulan, los [[Clase 09 - Espacios y Subespacios Vectoriales|espacios vectoriales]] que los contienen, y finalmente los [[Clase 10 - Valores y Vectores Propios|valores propios]] y la [[Clase 13 - Descomposición en Valores Singulares (SVD)|SVD]] que revelan su estructura interna — todo convergiendo en PCA, una de las técnicas más usadas en ciencia de datos real.
