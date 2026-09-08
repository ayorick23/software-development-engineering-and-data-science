---
tags: [scipy, optimizacion, cheat-sheet]
---

# 03 — Optimización (`scipy.optimize`)

> Para optimización específicamente de hiperparámetros de modelos de ML, ver [[Machine Learning/45-Optimizacion-de-Hiperparametros]] y [[Python/Optuna/01 - Introducción y Conceptos Fundamentales|Optuna]] — `scipy.optimize` resuelve el problema más general de minimizar cualquier función matemática, no solo hiperparámetros.

## El problema que resuelve

`scipy.optimize` responde a una pregunta recurrente: dada una función $f(x)$, ¿qué valor de $x$ la minimiza (o maximiza, minimizando $-f(x)$)? Aparece en ajuste de curvas, calibración de modelos físicos, y como motor interno de otras librerías (scikit-learn usa optimizadores de SciPy para regresión logística, SVM, etc.).

## `minimize()` — la función central

```python
from scipy.optimize import minimize

def f(x):
    return (x[0] - 3)**2 + (x[1] + 1)**2   # mínimo conocido en (3, -1)

resultado = minimize(f, x0=[0, 0])   # x0: punto de partida
resultado.x        # array([3., -1.]) aprox.
resultado.fun       # valor de f en el mínimo
resultado.success   # bool: si convergió
```

`x0` (el punto inicial) importa: en funciones no convexas, un mal `x0` puede converger a un mínimo local en vez del global.

### Elegir el método

```python
minimize(f, x0, method='Nelder-Mead')   # no requiere derivadas, robusto pero lento
minimize(f, x0, method='BFGS')          # usa gradiente (numérico si no se da), rápido
minimize(f, x0, method='L-BFGS-B', bounds=[(0, 10), (-5, 5)])   # con límites por variable
```

| Método | Cuándo usarlo |
|---|---|
| `Nelder-Mead` | Función no diferenciable o ruidosa; sin gradiente disponible |
| `BFGS` / `L-BFGS-B` | Función suave; `L-BFGS-B` además soporta límites (`bounds`) |
| `SLSQP` | Se necesitan restricciones de igualdad/desigualdad además de límites |
| `trust-constr` | Restricciones complejas, problemas grandes |

### Con restricciones

```python
restricciones = {'type': 'eq', 'fun': lambda x: x[0] + x[1] - 1}   # x0 + x1 = 1
minimize(f, x0=[0, 0], constraints=[restricciones], method='SLSQP')
```

## `curve_fit()` — ajustar una función a datos

El uso más común de `scipy.optimize` en análisis de datos: encontrar los parámetros de una función que mejor ajustan un conjunto de puntos observados (mínimos cuadrados no lineales).

```python
import numpy as np
from scipy.optimize import curve_fit

def gaussiana(x, a, mu, sigma):
    return a * np.exp(-(x - mu)**2 / (2 * sigma**2))

x_datos = np.linspace(-5, 5, 50)
y_datos = gaussiana(x_datos, 2, 0, 1) + np.random.normal(0, 0.1, 50)   # datos con ruido

parametros_optimos, covarianza = curve_fit(gaussiana, x_datos, y_datos, p0=[1, 0, 1])
a_ajustado, mu_ajustado, sigma_ajustado = parametros_optimos

errores_estandar = np.sqrt(np.diag(covarianza))   # incertidumbre de cada parámetro
```

`p0` (estimación inicial de los parámetros) es opcional pero recomendable: sin ella, `curve_fit` asume 1.0 para todos, lo que puede no converger en funciones con escalas muy distintas.

## `root()` — encontrar raíces de una función

Distinto de minimizar: aquí se busca $x$ tal que $f(x) = 0$.

```python
from scipy.optimize import root, brentq

# Multivariable
def sistema(x):
    return [x[0]**2 + x[1]**2 - 1, x[0] - x[1]]

sol = root(sistema, x0=[0.5, 0.5])

# Escalar, con un intervalo conocido que contiene la raíz (más robusto que root())
raiz = brentq(lambda x: x**2 - 2, a=0, b=2)   # ~1.41421 (raíz de 2)
```

## Diagnóstico de convergencia

Siempre revisar `result.success` antes de confiar en `result.x` — un optimizador puede terminar sin converger y devolver un resultado silenciosamente incorrecto si no se verifica:

```python
resultado = minimize(f, x0=[100, 100])
if not resultado.success:
    print(f"No convergió: {resultado.message}")
```

## Ver también

- [[04 - Estadística I - Distribuciones (scipy.stats)]] — `curve_fit` se usa a menudo para ajustar densidades de distribuciones a histogramas
- [[Machine Learning/45-Optimizacion-de-Hiperparametros]]
- [[12 - Integración con el Ecosistema]]
