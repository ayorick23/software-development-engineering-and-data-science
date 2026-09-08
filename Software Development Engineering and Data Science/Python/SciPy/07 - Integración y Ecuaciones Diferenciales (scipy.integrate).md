---
tags: [scipy, integracion, ecuaciones-diferenciales, cheat-sheet]
---

# 07 — Integración y Ecuaciones Diferenciales (`scipy.integrate`)

## Integración numérica de una función

```python
from scipy import integrate
import numpy as np

# Integral definida de f(x) = x^2 entre 0 y 3
resultado, error_estimado = integrate.quad(lambda x: x**2, 0, 3)
# resultado ~ 9.0
```

`quad` devuelve una tupla `(valor, error_estimado)` — el segundo elemento es una estimación del error numérico, útil para verificar la confiabilidad del resultado.

### Integrales dobles y triples

```python
# Integral doble de f(x, y) sobre un rectángulo
resultado, error = integrate.dblquad(
    lambda y, x: x * y,      # OJO: el orden de argumentos es (y, x), no (x, y)
    0, 1,                    # límites de x
    lambda x: 0, lambda x: 2 # límites de y (pueden depender de x)
)
```

### Integrar datos discretos (sin una función analítica)

```python
x = np.linspace(0, 10, 100)
y = np.sin(x)

integrate.trapezoid(y, x)   # regla del trapecio
integrate.simpson(y, x)     # regla de Simpson, generalmente más precisa
```

## Ecuaciones diferenciales ordinarias (EDOs)

`solve_ivp` ("initial value problem") resuelve numéricamente $\frac{dy}{dt} = f(t, y)$ dado un valor inicial:

```python
from scipy.integrate import solve_ivp

def decaimiento(t, y):
    return -0.5 * y   # dy/dt = -0.5 y (decaimiento exponencial)

solucion = solve_ivp(decaimiento, t_span=[0, 10], y0=[100], t_eval=np.linspace(0, 10, 50))

solucion.t   # puntos de tiempo evaluados
solucion.y   # valores de y en esos puntos (array 2D: una fila por variable de estado)
```

### Sistemas de EDOs

`solve_ivp` generaliza directamente a sistemas — `y0` y el valor de retorno de la función pasan a ser vectores:

```python
def lotka_volterra(t, y, alpha, beta, delta, gamma):
    presa, depredador = y
    d_presa = alpha * presa - beta * presa * depredador
    d_depredador = delta * presa * depredador - gamma * depredador
    return [d_presa, d_depredador]

solucion = solve_ivp(
    lotka_volterra, t_span=[0, 50], y0=[10, 5],
    args=(1.1, 0.4, 0.1, 0.4), t_eval=np.linspace(0, 50, 500)
)
```

### Elegir el método de integración

```python
solve_ivp(f, t_span, y0, method='RK45')   # por defecto — Runge-Kutta explícito, bueno para la mayoría de casos
solve_ivp(f, t_span, y0, method='Radau')  # para problemas "stiff" (dinámicas con escalas de tiempo muy distintas)
```

Si `RK45` produce resultados que oscilan de forma no física o el solver reduce el paso de tiempo hasta casi detenerse, es una señal de que el problema es *stiff* y conviene cambiar a `Radau` o `BDF`.

## Ver también

- [[03 - Optimización (scipy.optimize)]]
- [[08 - Procesamiento de Señales (scipy.signal)]]
