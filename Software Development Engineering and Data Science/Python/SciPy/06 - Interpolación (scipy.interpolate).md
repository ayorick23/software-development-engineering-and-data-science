---
tags: [scipy, interpolacion, cheat-sheet]
---

# 06 — Interpolación (`scipy.interpolate`)

## El problema que resuelve

Dado un conjunto de puntos conocidos $(x_i, y_i)$, estimar el valor de $y$ en un punto $x$ **entre** los puntos conocidos (interpolación) o **fuera** de su rango (extrapolación, generalmente menos confiable). Aparece al rellenar huecos en series de tiempo, generar curvas suaves para visualización, o remuestrear datos a una resolución distinta.

## Interpolación 1D

```python
import numpy as np
from scipy.interpolate import interp1d

x = np.array([0, 1, 2, 3, 4])
y = np.array([0, 1, 4, 9, 16])   # y = x^2, muestreado

f = interp1d(x, y, kind='linear')     # 'linear', 'nearest', 'cubic', 'quadratic'
f(2.5)   # valor interpolado en x=2.5

f_cubica = interp1d(x, y, kind='cubic')   # pasa una curva suave por los puntos, más precisa para datos suaves
```

Por defecto, `interp1d` lanza un error al evaluar fuera del rango `[min(x), max(x)]`. Para permitir extrapolación explícitamente:

```python
f = interp1d(x, y, kind='linear', fill_value='extrapolate')
```

## Splines cúbicos

Una alternativa más moderna y con mejor comportamiento numérico que `interp1d(kind='cubic')`:

```python
from scipy.interpolate import CubicSpline

cs = CubicSpline(x, y)
cs(2.5)              # valor interpolado
cs(2.5, 1)           # primera derivada en x=2.5
cs(2.5, 2)           # segunda derivada
```

## Interpolación en 2D/N-D

Útil para datos en una grilla (por ejemplo, un mapa de temperatura o elevación con mediciones dispersas):

```python
from scipy.interpolate import griddata

puntos = np.random.rand(50, 2) * 10       # 50 puntos (x, y) dispersos
valores = np.sin(puntos[:, 0]) * np.cos(puntos[:, 1])

grilla_x, grilla_y = np.mgrid[0:10:100j, 0:10:100j]   # grilla regular fina
grilla_z = griddata(puntos, valores, (grilla_x, grilla_y), method='cubic')
```

## Ver también

- [[03 - Optimización (scipy.optimize)]] — `curve_fit` ajusta una función paramétrica; interpolación no asume una forma funcional, solo pasa por los puntos
- [[Python/NumPy/07 - Manipulación de Forma (Reshaping)]]
