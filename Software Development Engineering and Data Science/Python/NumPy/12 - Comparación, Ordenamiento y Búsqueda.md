---
tags: [numpy, python, data-science, sorting, cheat-sheet]
---

# 12 — Comparación, Ordenamiento y Búsqueda

> Continúa de [[11 - Álgebra Lineal (np.linalg)]].

## Ordenamiento

```python
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])

np.sort(arr)              # devuelve una COPIA ordenada, no modifica arr
arr.sort()                  # ordena IN-PLACE, modifica arr y devuelve None

np.sort(arr)[::-1]           # orden descendente (NumPy no tiene un argumento reverse= directo)
```

## `argsort()` — obtener los ÍNDICES que ordenarían el array

```python
arr = np.array([30, 10, 20])
indices = np.argsort(arr)          # array([1, 2, 0]) — el índice del menor, luego el siguiente, etc.
arr[indices]                          # array([10, 20, 30]) — aplicar los índices reproduce el sort

# Uso real: ordenar UN array según el orden de OTRO (ej. ordenar nombres según edad)
edades = np.array([30, 10, 20])
nombres = np.array(["Ana", "Luis", "Eva"])
orden = np.argsort(edades)
nombres[orden]                         # array(['Luis', 'Eva', 'Ana']) — nombres ordenados por edad ascendente
```

`argsort()` es la herramienta clave cuando se necesita ordenar un array **en función de** los valores de otro array paralelo — sin él, habría que reconstruir pares manualmente.

## Ordenamiento en matrices 2D

```python
matriz = np.array([[3, 1], [1, 2], [2, 3]])
np.sort(matriz, axis=0)          # ordena cada COLUMNA de forma independiente
np.sort(matriz, axis=1)            # ordena cada FILA de forma independiente
```

## `np.unique()` — valores únicos

```python
arr = np.array([1, 2, 2, 3, 3, 3, 4])
np.unique(arr)                          # array([1, 2, 3, 4])
valores, conteos = np.unique(arr, return_counts=True)   # también la frecuencia de cada valor
valores, indices = np.unique(arr, return_index=True)      # índice de la PRIMERA aparición de cada valor
```

## Búsqueda

```python
arr = np.array([10, 20, 30, 40, 50])

np.where(arr == 30)               # (array([2]),) — índices donde se cumple la condición
np.searchsorted(arr, 25)            # 2 — índice donde INSERTAR 25 para mantener el orden (requiere arr ya ordenado)
np.searchsorted(arr, [15, 35])        # también acepta un array de valores a buscar
```

`searchsorted()` usa búsqueda binaria (`O(log n)`) y **requiere que el array de entrada ya esté ordenado** — es mucho más rápido que `np.where()` para localizar posiciones de inserción en arrays grandes ya ordenados.

## Reducciones booleanas: `any()` y `all()`

```python
arr = np.array([1, 2, -3, 4])

(arr > 0).any()          # True — ¿al menos UN elemento cumple?
(arr > 0).all()            # False — ¿TODOS los elementos cumplen?
np.any(arr > 0, axis=0)      # también aceptan axis en arrays multidimensionales
```

## Comparación de arrays completos

```python
np.array_equal(a, b)             # True si mismas forma y valores EXACTOS
np.allclose(a, b, atol=1e-8)       # True si son "aproximadamente iguales" — imprescindible con floats
```

## `np.count_nonzero()` — contar sin crear arrays intermedios

```python
np.count_nonzero(arr > 0)          # cuenta cuántos elementos cumplen la condición, más directo que (arr > 0).sum()
```

## Ver también

- [[10 - Operaciones Matemáticas y Vectorización]]
- [[06 - Indexado Avanzado]]
- [[15 - Rendimiento y Vectorización Avanzada]]
