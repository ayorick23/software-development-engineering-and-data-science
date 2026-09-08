---
tags: [sympy, algebra, cheat-sheet]
---

# 02 — Álgebra y Simplificación

## Expandir y factorizar

```python
import sympy as sp

x, y = sp.symbols('x y')

expr = (x + y)**3
sp.expand(expr)   # x**3 + 3*x**2*y + 3*x*y**2 + y**3

expr2 = x**2 - y**2
sp.factor(expr2)   # (x - y)*(x + y)
```

## Simplificar

```python
expr = (x**2 - 1) / (x - 1)
sp.simplify(expr)   # x + 1

sp.simplify(sp.sin(x)**2 + sp.cos(x)**2)   # 1 — reconoce identidades trigonométricas
```

`simplify()` prueba varias estrategias internamente y puede ser lenta en expresiones grandes; cuando se sabe qué transformación específica se necesita, usar la función dedicada (`expand`, `factor`, `trigsimp`, `logcombine`) es más rápido y predecible.

## Sustitución de valores o subexpresiones

```python
expr = x**2 + 2*x + 1

expr.subs(x, 3)          # 16 -- sustituye x=3
expr.subs(x, y + 1)      # sustituye x por otra expresión simbólica

# Sustituir varios símbolos a la vez
(x + y).subs({x: 1, y: 2})   # 3
```

## Trabajar con fracciones y potencias

```python
sp.Rational(1, 3)              # 1/3 exacto, no 0.333...
sp.Rational(1, 3) + sp.Rational(1, 6)   # 1/2 -- aritmética exacta de fracciones

sp.powsimp(x**2 * x**3)        # x**5
sp.radsimp(1 / (sp.sqrt(2)))   # sqrt(2)/2 -- racionaliza el denominador
```

## Polinomios

```python
poly = sp.Poly(x**3 - 6*x**2 + 11*x - 6, x)

poly.roots()          # raíces del polinomio (aquí: 1, 2, 3)
sp.roots(poly)         # equivalente, con multiplicidades
sp.div(x**3 - 1, x - 1)   # división polinomial: (cociente, resto)
```

## Comparar expresiones por equivalencia, no por igualdad literal

```python
expr1 = (x + 1)**2
expr2 = x**2 + 2*x + 1

expr1 == expr2                     # False -- son literalmente distintas como objetos
sp.simplify(expr1 - expr2) == 0    # True -- son matemáticamente equivalentes
```

Este es un error común: comparar dos expresiones simbólicas con `==` compara su representación, no su valor matemático. Para verificar equivalencia matemática, restar y simplificar (o usar `sp.Eq(...).simplify()`).

## Ver también

- [[03 - Cálculo Simbólico]]
- [[04 - Resolución de Ecuaciones y Sistemas]]
