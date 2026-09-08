---
tags: [sympy, calculo, cheat-sheet]
---

# 03 — Cálculo Simbólico

> Para el fundamento teórico de estas operaciones, ver [[Calculus Fundamentals/Derivadas e Integrales Fundamentales]]. Este archivo cubre cómo obtener el resultado exacto automáticamente en vez de a mano.

## Derivadas

```python
import sympy as sp

x = sp.symbols('x')
f = x**3 + 2*x**2 - 5*x + 1

sp.diff(f, x)          # 3*x**2 + 4*x - 5
sp.diff(f, x, 2)        # segunda derivada: 6*x + 4
sp.diff(f, x, x, x)     # tercera derivada, forma alternativa: 6
```

### Derivadas parciales

```python
x, y = sp.symbols('x y')
g = x**2 * y + sp.sin(x * y)

sp.diff(g, x)          # derivada parcial respecto a x
sp.diff(g, y)          # derivada parcial respecto a y
sp.diff(g, x, y)       # derivada mixta: d²g/dx dy
```

## Integrales

```python
f = x**2

sp.integrate(f, x)                # x**3/3  -- integral indefinida (sin constante de integración)
sp.integrate(f, (x, 0, 3))        # 9       -- integral definida entre 0 y 3
```

### Integrales dobles

```python
y = sp.symbols('y')
sp.integrate(x * y, (x, 0, 1), (y, 0, 2))
```

Muchas integrales no tienen una forma cerrada — cuando SymPy no puede resolverla simbólicamente, devuelve la expresión sin evaluar (`Integral(...)`) en vez de fallar; en ese caso hay que recurrir a integración numérica con [[Python/SciPy/07 - Integración y Ecuaciones Diferenciales (scipy.integrate)|scipy.integrate.quad]].

## Límites

```python
sp.limit(sp.sin(x) / x, x, 0)        # 1 -- el límite clásico, sin necesidad de L'Hôpital manual
sp.limit(1 / x, x, 0, '+')           # oo (infinito) -- límite por la derecha
sp.limit((1 + 1/x)**x, x, sp.oo)     # E -- el número e, como límite
```

## Series de Taylor

```python
f = sp.exp(x)
sp.series(f, x, 0, 5)   # expansión de e^x alrededor de x=0, hasta el término de orden 4 (+ O(x**5))
```

Útil para verificar de dónde salen aproximaciones usadas en optimización numérica (por ejemplo, por qué un método de primer orden usa el gradiente y uno de segundo orden usa también la matriz Hessiana).

## Ecuaciones diferenciales (forma cerrada, cuando existe)

```python
t = sp.symbols('t')
y = sp.Function('y')

ecuacion = sp.Eq(y(t).diff(t), -0.5 * y(t))   # dy/dt = -0.5y
sp.dsolve(ecuacion, y(t))                      # Eq(y(t), C1*exp(-0.5*t))
```

Cuando la ecuación no tiene solución cerrada (la mayoría de los casos reales), se resuelve numéricamente con [[Python/SciPy/07 - Integración y Ecuaciones Diferenciales (scipy.integrate)|scipy.integrate.solve_ivp]].

## Ver también

- [[04 - Resolución de Ecuaciones y Sistemas]]
- [[07 - De lo Simbólico a lo Numérico (lambdify)]]
