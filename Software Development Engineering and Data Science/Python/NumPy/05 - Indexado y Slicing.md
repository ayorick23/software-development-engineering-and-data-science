---
tags: [numpy, python, data-science, indexing, cheat-sheet]
---

# 05 — Indexado y Slicing

> Continúa de [[04 - Inspección y Atributos de Arrays]].

## Indexado básico (1D)

```python
arr = np.array([10, 20, 30, 40, 50])

arr[0]           # 10 — primer elemento
arr[-1]           # 50 — último elemento
arr[1:4]           # [20, 30, 40] — slice, fin EXCLUSIVO
arr[:3]             # [10, 20, 30] — desde el inicio
arr[2:]              # [30, 40, 50] — hasta el final
arr[::2]              # [10, 30, 50] — cada 2 elementos
arr[::-1]              # [50, 40, 30, 20, 10] — invertido
```

## Indexado multidimensional

```python
matriz = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

matriz[0]                # array([1, 2, 3]) — primera fila completa
matriz[0, 1]               # 2 — fila 0, columna 1 (forma preferida sobre matriz[0][1])
matriz[:, 1]                 # array([2, 5, 8]) — columna 1 completa, todas las filas
matriz[1, :]                  # array([4, 5, 6]) — fila 1 completa, todas las columnas
matriz[0:2, 1:3]                # submatriz: filas 0-1, columnas 1-2
matriz[::-1]                      # filas en orden invertido
```

`matriz[0, 1]` y `matriz[0][1]` dan el mismo resultado, pero `matriz[0][1]` primero crea un array intermedio (`matriz[0]`) y luego indexa sobre él — `matriz[0, 1]` es una sola operación, más eficiente y la forma idiomática en NumPy.

## Slicing en 3+ dimensiones

```python
cubo = np.arange(24).reshape(2, 3, 4)   # forma (2, 3, 4)

cubo[0]              # primer "bloque" 2D, shape (3, 4)
cubo[0, 1]             # fila 1 del primer bloque, shape (4,)
cubo[0, 1, 2]           # un escalar específico
cubo[..., 0]              # Ellipsis (...) — "todos los ejes anteriores", equivalente a cubo[:, :, 0]
```

`...` (Ellipsis) es un atajo para "todos los ejes que no se mencionan explícitamente" — muy útil en arrays de dimensión alta o variable donde escribir `:` por cada eje sería tedioso.

## El slicing devuelve una VISTA, no una copia

```python
sub = matriz[0:2, 0:2]
sub[0, 0] = 999
# matriz TAMBIÉN cambió — el slicing comparte memoria con el original

sub_independiente = matriz[0:2, 0:2].copy()   # copia explícita, ya no comparte memoria
```

Esta es la diferencia de comportamiento más importante entre slicing de NumPy y slicing de listas de Python (que sí copian) — el detalle completo, con más ejemplos de qué operaciones copian y cuáles no, está en [[09 - Copias vs Vistas]].

## Modificar valores vía indexado

```python
arr[0] = 100                    # un elemento
arr[1:3] = [200, 300]            # un rango
matriz[:, 0] = 0                  # toda una columna a cero
matriz[matriz > 5] = 0             # ver indexado booleano en 06
```

## Ver también

- [[04 - Inspección y Atributos de Arrays]]
- [[06 - Indexado Avanzado]]
- [[09 - Copias vs Vistas]]
