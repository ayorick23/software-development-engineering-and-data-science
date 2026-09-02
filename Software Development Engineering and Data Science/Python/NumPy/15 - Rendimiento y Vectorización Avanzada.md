---
tags: [numpy, python, data-science, performance, cheat-sheet]
---

# 15 — Rendimiento y Vectorización Avanzada

> Continúa de [[14 - Entrada y Salida de Datos]].

## Vectorización vs loops: la regla de oro

```python
import time

arr = np.arange(1_000_000)

# LENTO — loop de Python, cada iteración pasa por el intérprete
inicio = time.time()
resultado = [x ** 2 for x in arr]
print(time.time() - inicio)     # típicamente ~0.1-0.3s

# RÁPIDO — vectorizado, corre en un loop compilado de C
inicio = time.time()
resultado = arr ** 2
print(time.time() - inicio)      # típicamente ~0.001s — 100x+ más rápido
```

Cualquier operación expresable como aritmética/ufunc vectorizada debe escribirse así — un loop explícito de Python sobre un `ndarray` desperdicia precisamente la ventaja de rendimiento por la que se eligió NumPy en primer lugar.

## Preasignar memoria en vez de crecer arrays iterativamente

```python
# LENTO — np.append() copia TODO el array en cada llamada (los arrays de NumPy son de tamaño FIJO)
resultado = np.array([])
for x in datos:
    resultado = np.append(resultado, x * 2)

# RÁPIDO — preasignar el tamaño final una sola vez
resultado = np.empty(len(datos))
for i, x in enumerate(datos):
    resultado[i] = x * 2

# ÓPTIMO — vectorizado directamente, sin loop
resultado = datos * 2
```

`np.append()`/`np.concatenate()` dentro de un loop es una trampa de rendimiento común: como los arrays de NumPy tienen tamaño fijo, cada `append` en realidad **reasigna y copia todo el array completo** — con `n` iteraciones, el costo total es `O(n²)`, no `O(n)`.

## Layout de memoria: acceso contiguo vs "saltando"

```python
matriz = np.random.rand(1000, 1000)

matriz.sum(axis=1)    # recorre cada FILA de forma contigua en memoria (C-order) — más rápido
matriz.sum(axis=0)      # recorre "saltando" entre filas para sumar columnas — más lento (peor localidad de caché)
```

En un array en **C-order** (el default de NumPy), los elementos de una misma fila son contiguos en memoria — iterar/agregar a lo largo de filas aprovecha mejor la caché del procesador que agregar a lo largo de columnas. La diferencia es medible en matrices grandes; ver el detalle de `strides` en [[Python/NumPy/01 - Introducción y Arquitectura Interna]].

## `einsum()` — expresar operaciones complejas de forma explícita y eficiente

```python
A = np.random.rand(3, 4)
B = np.random.rand(4, 5)

np.einsum("ij,jk->ik", A, B)          # equivalente a A @ B, pero explícito sobre qué índices se contraen
np.einsum("ii->i", matriz_cuadrada)     # extrae la diagonal — equivalente a np.diag()
np.einsum("ij->", matriz)                 # suma TODOS los elementos — equivalente a matriz.sum()
```

`einsum` (*Einstein summation*) expresa productos y contracciones de tensores mediante una notación de índices — para operaciones complejas que combinan varias transposiciones/sumas/productos, puede ser más rápido que encadenar varias operaciones NumPy por separado, porque evita crear arrays intermedios.

## Evitar copias innecesarias

```python
# Cada operación intermedia crea un array temporal nuevo
resultado = (arr * 2 + 1) ** 2

# Con ufuncs "in-place" (operador con =) se reutiliza el mismo buffer, sin arrays temporales
arr *= 2
arr += 1
arr **= 2
```

Los operadores in-place (`*=`, `+=`, `**=`) evitan la creación de arrays intermedios en cadenas de operaciones aritméticas — relevante en arrays muy grandes donde cada temporal representa una allocación de memoria significativa (pero **cuidado**: modifican el array original, lo cual puede ser justo lo que no se quiere si otra variable comparte esa memoria — ver [[09 - Copias vs Vistas]]).

## Cuándo NumPy ya no basta

| Síntoma | Alternativa |
|---|---|
| Loop de Python irreducible (lógica no vectorizable) sobre millones de elementos | **Numba** (`@numba.jit`) — compila la función Python a código máquina |
| Cómputo que satura una CPU y se necesita paralelismo | **Numba** con `parallel=True`, o Dask |
| Dataset no cabe en RAM | Dask arrays (API calcada de NumPy) |
| Se necesita GPU | **CuPy** (API prácticamente idéntica a NumPy, corre en GPU) |

```python
from numba import jit

@jit(nopython=True)
def suma_iterativa(arr):
    total = 0.0
    for x in arr:
        total += x
    return total
```

`@numba.jit` compila la función a código máquina en su primera llamada — permite loops explícitos de Python a velocidad cercana a C, para la minoría de casos donde de verdad no existe una forma vectorizada razonable de expresar la lógica.

## Ver también

- [[Python/NumPy/01 - Introducción y Arquitectura Interna]]
- [[08 - Broadcasting]]
- [[09 - Copias vs Vistas]]
- [[Python/NumPy/16 - Integración con el Ecosistema]]
- [[Python/Pandas/15 - Rendimiento y Optimización|Python/Pandas]]
