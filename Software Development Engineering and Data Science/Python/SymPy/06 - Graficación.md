---
tags: [sympy, graficacion, cheat-sheet]
---

# 06 — Graficación

## Graficar una expresión simbólica directamente

SymPy incluye un módulo de graficación (`sympy.plotting`) construido sobre Matplotlib, pensado para visualizar rápidamente una expresión sin convertirla manualmente a arrays de puntos:

```python
import sympy as sp

x = sp.symbols('x')
sp.plot(x**2 - 4, (x, -5, 5))   # grafica automáticamente, eligiendo el rango de muestreo
```

## Varias funciones en el mismo gráfico

```python
sp.plot(sp.sin(x), sp.cos(x), (x, -sp.pi, sp.pi))
```

## Gráficos 3D

```python
from sympy.plotting import plot3d

y = sp.symbols('y')
plot3d(x**2 + y**2, (x, -5, 5), (y, -5, 5))
```

## Cuándo usar `sympy.plot` vs Matplotlib directamente

`sympy.plot` es conveniente para una inspección rápida de una expresión simbólica recién derivada o simplificada, sin salir del flujo de trabajo simbólico. Para un gráfico que necesita control fino de estilo, múltiples subplots, o integrarse con datos reales (no solo una fórmula), conviene convertir la expresión a una función numérica con [[07 - De lo Simbólico a lo Numérico (lambdify)|lambdify]] y graficar con [[Python/Matplotlib/01 - Introducción y Arquitectura|Matplotlib]] directamente — se gana control total sobre ejes, estilos y composición de la figura.

```python
import numpy as np
import matplotlib.pyplot as plt

f_numerica = sp.lambdify(x, x**2 - 4, 'numpy')
xs = np.linspace(-5, 5, 200)

plt.plot(xs, f_numerica(xs))
plt.show()
```

## Ver también

- [[07 - De lo Simbólico a lo Numérico (lambdify)]]
- [[Python/Matplotlib/01 - Introducción y Arquitectura]]
