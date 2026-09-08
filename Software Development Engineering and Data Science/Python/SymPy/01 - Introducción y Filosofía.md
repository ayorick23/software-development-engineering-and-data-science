---
tags: [sympy, matematica-simbolica, cheat-sheet]
---

# 01 — Introducción y Filosofía

> Para el fundamento teórico de derivadas e integrales que SymPy calcula simbólicamente, ver [[Calculus Fundamentals/Derivadas e Integrales Fundamentales]]. Para la contraparte numérica (arrays, cómputo con números de punto flotante), ver [[Python/NumPy/01 - Introducción y Arquitectura Interna|NumPy]].

## ¿Qué es SymPy y en qué se diferencia de NumPy?

**SymPy** es una biblioteca de **matemática simbólica** (Computer Algebra System / CAS): en vez de trabajar con números aproximados de punto flotante, trabaja con **expresiones matemáticas exactas** — variables, ecuaciones, derivadas, en su forma simbólica, tal como se escribirían a mano en una pizarra.

```python
import numpy as np
import sympy as sp

# NumPy — numérico, aproximado, rápido
np.sqrt(2)          # 1.4142135623730951 (float64, con error de redondeo)

# SymPy — simbólico, exacto
sp.sqrt(2)          # sqrt(2)  <- se mantiene como expresión exacta
sp.sqrt(2).evalf()  # 1.41421356237310  <- solo aquí se convierte a decimal
```

| | NumPy | SymPy |
|---|---|---|
| Representa | Números concretos (arrays) | Expresiones matemáticas |
| Precisión | Punto flotante (con error de redondeo) | Exacta (fracciones, raíces, $\pi$ simbólico) |
| Uso típico | Cómputo masivo sobre datos | Álgebra, cálculo, resolver ecuaciones a mano-pero-automatizado |
| Velocidad | Muy rápido (vectorizado, compilado) | Más lento — no está pensado para grandes volúmenes de datos |

SymPy no reemplaza a NumPy/SciPy para cómputo numérico masivo; **complementa** ese flujo cuando hace falta manipular una fórmula *antes* de evaluarla numéricamente — derivar una expresión, simplificarla, o resolver una ecuación de forma cerrada.

## Instalación

```bash
pip install sympy
```

## El objeto central: `Symbol`

A diferencia de una variable de Python (que apunta a un valor concreto), un `Symbol` de SymPy representa una incógnita matemática sin valor asignado:

```python
x, y = sp.symbols('x y')

expresion = x**2 + 2*x*y + y**2
expresion   # x**2 + 2*x*y + y**2  <- se queda simbólica, no se evalúa
```

## `pretty printing` — ver las expresiones como en un libro de texto

```python
sp.init_printing()   # en un notebook/Jupyter, renderiza las expresiones con notación matemática (LaTeX)

sp.Integral(x**2, x)   # se muestra como ∫x² dx en vez de como texto plano
```

## Cuándo usar SymPy

- Derivar o integrar una función simbólicamente antes de implementarla en código numérico.
- Resolver ecuaciones o sistemas de forma exacta (sin aproximación iterativa).
- Verificar a mano un cálculo de cálculo/álgebra lineal antes de confiar en una implementación numérica.
- Generar la fórmula de una derivada compleja para después evaluarla rápidamente con NumPy (`lambdify`, ver [[07 - De lo Simbólico a lo Numérico (lambdify)]]).

Cuándo **no**: cuando el problema ya involucra datos numéricos reales (usar NumPy/SciPy), o cuando el volumen de cómputo es grande (SymPy no está optimizado para eso).

## Ver también

- [[Calculus Fundamentals/Derivadas e Integrales Fundamentales]]
- [[07 - De lo Simbólico a lo Numérico (lambdify)]]
