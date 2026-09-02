---
tags: [numpy, python, data-science, cheat-sheet]
---

# 02 — Creación de Arrays

> Continúa de [[Python/NumPy/01 - Introducción y Arquitectura Interna]].

## Desde datos existentes

```python
np.array([1, 2, 3])                        # desde una lista
np.array([[1, 2], [3, 4]])                  # desde una lista de listas -> 2D
np.array((1, 2, 3))                          # desde una tupla
np.array([1, 2, 3], dtype=np.float32)        # forzando dtype en la creación
np.asarray(lista)                             # como np.array, pero NO copia si el input ya es un ndarray del dtype pedido
```

`np.asarray()` es preferible a `np.array()` dentro de funciones que reciben datos ya en formato NumPy — evita una copia innecesaria si el input ya cumple los requisitos, mientras que `np.array()` copia por default (`copy=True`).

## Arrays con valores predefinidos

```python
np.zeros((3, 3))              # matriz 3x3 de ceros, dtype float64 por default
np.zeros((3, 3), dtype=int)   # de enteros
np.ones((2, 4))                # matriz de unos
np.full((2, 3), 7)              # matriz rellena con un valor constante
np.empty((2, 2))                # SIN inicializar — valores "basura" ya presentes en memoria, más rápido cuando se va a sobreescribir todo de inmediato
np.zeros_like(otro_array)       # misma forma y dtype que otro_array, lleno de ceros
np.ones_like(otro_array)
np.full_like(otro_array, 7)
```

`np.empty()` es más rápido que `np.zeros()` porque no gasta tiempo inicializando memoria — usarlo solo cuando el array se va a llenar completamente justo después, nunca si se van a leer valores antes de asignarlos.

## Secuencias numéricas

```python
np.arange(0, 10, 2)          # [0, 2, 4, 6, 8] — como range() de Python, pero devuelve un ndarray; fin EXCLUSIVO
np.arange(10)                 # equivalente a arange(0, 10, 1)
np.linspace(0, 1, 5)          # [0. , 0.25, 0.5 , 0.75, 1. ] — 5 valores espaciados uniformemente; fin INCLUSIVO
np.logspace(0, 3, 4)           # [1, 10, 100, 1000] — espaciado uniforme en escala LOGARÍTMICA (exponentes de 0 a 3)
```

| | `arange` | `linspace` |
|---|---|---|
| Se especifica | paso (`step`) | número de puntos (`num`) |
| Extremo final | Exclusivo | Inclusivo (`endpoint=True` default) |
| Uso típico | Secuencias de enteros, pasos exactos | Muestreo uniforme para gráficas, floats |

## Matrices especiales de álgebra lineal

```python
np.identity(3)         # matriz identidad 3x3
np.eye(3)               # equivalente, pero admite desplazamiento de la diagonal
np.eye(3, k=1)           # diagonal desplazada una posición hacia arriba
np.diag([1, 2, 3])       # matriz diagonal a partir de un vector
np.diag(matriz)           # extrae la diagonal de una matriz existente (uso inverso)
```

Ver el uso de estas matrices en operaciones reales en [[11 - Álgebra Lineal (np.linalg)]].

## Arrays de números aleatorios

```python
rng = np.random.default_rng(seed=42)          # Generator moderno — ver 13 para el detalle completo
rng.random((3, 3))                              # floats uniformes en [0, 1)
rng.integers(0, 10, size=(3, 3))                # enteros aleatorios
rng.normal(loc=0, scale=1, size=(3, 3))          # distribución normal
```

Cobertura completa de la API moderna `Generator` vs la legacy `np.random.seed()`/`np.random.rand()` en [[13 - Números Aleatorios (Generator API)]].

## Desde un rango repetido o "tileado"

```python
np.repeat([1, 2, 3], 3)          # [1 1 1 2 2 2 3 3 3] — repite cada elemento
np.tile([1, 2, 3], 3)             # [1 2 3 1 2 3 1 2 3] — repite la secuencia completa
np.tile([[1, 2], [3, 4]], (2, 1))  # repite un bloque 2D en la dirección de las filas
```

## Ver también

- [[Python/NumPy/01 - Introducción y Arquitectura Interna]]
- [[03 - Tipos de Datos (dtype)]]
- [[11 - Álgebra Lineal (np.linalg)]]
- [[13 - Números Aleatorios (Generator API)]]
