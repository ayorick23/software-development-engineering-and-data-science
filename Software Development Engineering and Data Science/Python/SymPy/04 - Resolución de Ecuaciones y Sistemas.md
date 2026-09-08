---
tags: [sympy, ecuaciones, cheat-sheet]
---

# 04 — Resolución de Ecuaciones y Sistemas

## `solve()` — la función clásica

```python
import sympy as sp

x = sp.symbols('x')

sp.solve(x**2 - 4, x)          # [-2, 2]
sp.solve(sp.Eq(x**2, 4), x)    # equivalente, usando sp.Eq explícitamente
sp.solve(x**2 + 1, x)          # [-I, I] -- raíces complejas, no un error
```

`solve()` acepta tanto una expresión igualada implícitamente a cero (`x**2 - 4`) como un objeto `Eq` explícito — ambas formas son equivalentes, pero `Eq` es más legible cuando la ecuación original no está ya despejada a cero.

## Sistemas de ecuaciones

```python
x, y = sp.symbols('x y')

sistema = [sp.Eq(x + y, 5), sp.Eq(x - y, 1)]
sp.solve(sistema, [x, y])   # {x: 3, y: 2}
```

### Sistemas lineales — `linsolve`

Más eficiente que `solve()` genérico cuando se sabe de antemano que el sistema es lineal:

```python
from sympy import linsolve

linsolve([x + y - 5, x - y - 1], [x, y])   # FiniteSet((3, 2))
```

## `solveset` — la alternativa moderna a `solve`

`solveset` tiene una API más consistente (siempre devuelve un conjunto, con un dominio explícito) y es la dirección recomendada por SymPy para casos nuevos, aunque `solve` sigue siendo más flexible para sistemas mixtos:

```python
from sympy import solveset, S

solveset(x**2 - 4, x, domain=S.Reals)     # {-2, 2}
solveset(sp.sin(x), x, domain=S.Reals)    # infinitas soluciones -- devuelve un conjunto paramétrico, no una lista
```

Para ecuaciones con infinitas soluciones (como `sin(x) = 0`), `solveset` devuelve una representación paramétrica del conjunto completo en vez de listar valores — otra razón por la que puede ser más apropiado que `solve()` para ecuaciones trigonométricas.

## Ecuaciones sin solución simbólica: `nsolve`

Cuando no existe una forma cerrada (ecuaciones trascendentes complicadas), se resuelve numéricamente con un punto de partida, similar a `scipy.optimize.root`:

```python
from sympy import nsolve

nsolve(sp.cos(x) - x, x, 1)   # aproximación numérica cerca de x0=1
```

Si la ecuación tiene una solución numérica pero no cerrada, esta es la vía correcta — intentar forzar `solve()` sobre ella puede colgarse o no devolver nada útil.

## Ver también

- [[03 - Cálculo Simbólico]]
- [[05 - Álgebra Lineal Simbólica]]
- [[Python/SciPy/03 - Optimización (scipy.optimize)]] — la contraparte puramente numérica de `nsolve`
