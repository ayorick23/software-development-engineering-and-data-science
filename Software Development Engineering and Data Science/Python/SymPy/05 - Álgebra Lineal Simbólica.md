---
tags: [sympy, algebra-lineal, cheat-sheet]
---

# 05 — Álgebra Lineal Simbólica

> Para álgebra lineal numérica (arrays de valores concretos), ver [[Python/NumPy/11 - Álgebra Lineal (np.linalg)|NumPy]] y [[Python/SciPy/02 - Álgebra Lineal (scipy.linalg)|SciPy]]. Este archivo cubre matrices con entradas simbólicas (variables, no solo números).

## Construir una matriz simbólica

```python
import sympy as sp

x, y = sp.symbols('x y')

M = sp.Matrix([[1, x], [y, 2]])
M   # Matrix([[1, x], [y, 2]])
```

A diferencia de un array de NumPy, las entradas pueden ser expresiones simbólicas, no solo números.

## Operaciones básicas

```python
M.T              # transpuesta
M.det()          # determinante -- aquí, una expresión en x, y: 2 - x*y
M.inv()          # inversa -- una matriz de expresiones racionales en x, y

A = sp.Matrix([[1, 2], [3, 4]])
B = sp.Matrix([[5, 6], [7, 8]])
A * B            # multiplicación matricial
A + B            # suma
```

## Resolver sistemas lineales simbólicos

```python
A = sp.Matrix([[2, 1], [1, 3]])
b = sp.Matrix([5, 10])

A.solve(b)   # solución exacta de Ax = b, como vector de fracciones/expresiones
```

## Autovalores y autovectores simbólicos

```python
M = sp.Matrix([[2, 1], [1, 2]])

M.eigenvals()      # {3: 1, 1: 1} -- autovalor: multiplicidad
M.eigenvects()      # lista de (autovalor, multiplicidad, [autovectores])
```

Con una matriz simbólica (entradas con variables), los autovalores se devuelven como expresión — útil para entender cómo cambia el comportamiento de un sistema al variar un parámetro, algo que un cálculo puramente numérico no puede mostrar de una vez.

## Matriz Jacobiana

Central en optimización y en redes neuronales (la base conceptual detrás de backpropagation):

```python
x, y = sp.symbols('x y')
f = sp.Matrix([x**2 + y, sp.sin(x) * y])

f.jacobian([x, y])   # matriz de todas las derivadas parciales
```

## Cuándo preferir `sympy.Matrix` sobre NumPy

Usar SymPy cuando la matriz contiene variables simbólicas y se necesita el resultado como fórmula exacta (por ejemplo, derivar la fórmula del determinante de una matriz 3×3 genérica). Para matrices numéricas de tamaño real (miles de filas/columnas), `sympy.Matrix` es órdenes de magnitud más lento que NumPy/SciPy — convertir a numérico con [[07 - De lo Simbólico a lo Numérico (lambdify)|lambdify]] en cuanto se tienen valores concretos.

## Ver también

- [[Python/NumPy/11 - Álgebra Lineal (np.linalg)]]
- [[07 - De lo Simbólico a lo Numérico (lambdify)]]
