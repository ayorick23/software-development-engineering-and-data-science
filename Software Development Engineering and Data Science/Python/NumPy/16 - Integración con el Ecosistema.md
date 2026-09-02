---
tags: [numpy, python, data-science, integrations, cheat-sheet]
---

# 16 — Integración con el Ecosistema

> Continúa de [[15 - Rendimiento y Vectorización Avanzada]]. NumPy es la base de facto de casi todo el stack científico de Python — este es el mapa de esas conexiones.

## Pandas — construido directamente sobre NumPy

```python
serie = pd.Series([1, 2, 3])
serie.to_numpy()                 # Series -> ndarray
df[["precio", "stock"]].to_numpy()   # DataFrame -> matriz 2D

df = pd.DataFrame(np.random.randn(5, 3), columns=["a", "b", "c"])   # ndarray -> DataFrame
```

Cada columna numérica de un DataFrame de Pandas es, por debajo, un `ndarray` (o un Extension Array cuando aplica) — ver la arquitectura completa en [[Python/Pandas/01 - Introducción y Arquitectura Interna|Python/Pandas]].

## Matplotlib — consume arrays de NumPy directamente

```python
import matplotlib.pyplot as plt

x = np.linspace(0, 2 * np.pi, 100)
y = np.sin(x)
plt.plot(x, y)
```

Toda la API de Matplotlib está diseñada para recibir `ndarray`s como entrada — es, junto con Pandas, el consumidor más directo de arrays de NumPy en el stack de visualización (ver [[Python/Matplotlib/01 - Introducción y Arquitectura|Python/Matplotlib]]).

## Scikit-learn — arrays de NumPy como formato interno

```python
from sklearn.linear_model import LinearRegression

X = np.random.rand(100, 3)     # features — shape (n_muestras, n_features)
y = np.random.rand(100)          # target — shape (n_muestras,)

modelo = LinearRegression()
modelo.fit(X, y)
modelo.coef_                       # devuelve un ndarray con los coeficientes aprendidos
```

Scikit-learn acepta DataFrames de Pandas en su API moderna, pero internamente **todo** se convierte a arrays de NumPy antes de entrenar — entender la forma `(n_muestras, n_features)` esperada por `.fit()` es directamente el resultado de cómo NumPy representa matrices. Ver [[01 - Introducción, Filosofía y la API Consistente|Scikit-learn]].

## SciPy — extiende NumPy con álgebra lineal avanzada y estadística

```python
from scipy import stats
from scipy.optimize import curve_fit

stats.zscore(arr)                     # z-scores, útil para detección de outliers
curve_fit(funcion_modelo, x_datos, y_datos)   # ajuste de curvas no lineales sobre arrays de NumPy
```

SciPy toma como entrada y devuelve `ndarray`s en toda su API — es, conceptualmente, "todo lo que NumPy deja fuera" (optimización, estadística avanzada, procesamiento de señales) construido sobre el mismo array base.

## PyTorch y TensorFlow — conversión de/hacia tensores

```python
import torch
tensor = torch.from_numpy(arr)          # ndarray -> tensor de PyTorch, SIN copiar memoria (comparten buffer)
arr_de_vuelta = tensor.numpy()            # tensor -> ndarray, también sin copia

import tensorflow as tf
tensor_tf = tf.convert_to_tensor(arr)
```

`torch.from_numpy()` comparte memoria con el array de NumPy original (es una vista, no una copia) — modificar uno modifica el otro, el mismo principio de [[09 - Copias vs Vistas]] aplicado entre librerías distintas. Los tensores de PyTorch/TensorFlow añaden sobre esta misma idea de array N-dimensional el cálculo de gradientes automático y ejecución en GPU.

## CuPy — el "NumPy para GPU"

```python
import cupy as cp
arr_gpu = cp.array([1, 2, 3])     # misma API que NumPy, pero corre en la GPU
resultado = cp.sum(arr_gpu ** 2)
```

CuPy replica intencionalmente la API de NumPy casi función por función — migrar código de NumPy a GPU frecuentemente es tan simple como cambiar el `import numpy as np` por `import cupy as np`, sin reescribir la lógica.

## Ver también

- [[15 - Rendimiento y Vectorización Avanzada]]
- [[Python/NumPy/01 - Introducción y Arquitectura Interna]]
- [[Python/Pandas/16 - Integración con el Ecosistema|Python/Pandas]]
- [[01 - Introducción, Filosofía y la API Consistente|Scikit-learn]]
- [[Machine Learning/07-Librerias-de-Data-Science-y-ML#NumPy|Machine Learning/07 - Librerías]]
