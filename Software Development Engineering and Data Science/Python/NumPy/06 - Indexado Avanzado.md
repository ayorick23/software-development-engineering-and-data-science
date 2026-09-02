---
tags: [numpy, python, data-science, indexing, cheat-sheet]
---

# 06 — Indexado Avanzado

> Continúa de [[05 - Indexado y Slicing]].

## Indexado booleano (boolean masking)

```python
arr = np.array([10, 20, 30, 40, 50])
mascara = arr > 25                  # array([False, False, True, True, True])

arr[mascara]                          # array([30, 40, 50])
arr[arr > 25]                          # equivalente, escrito directo — el patrón más común
arr[(arr > 20) & (arr < 50)]            # combinar condiciones — & y |, con paréntesis obligatorios
arr[~(arr > 25)]                          # negación con ~
```

A diferencia del slicing simple, el indexado booleano **siempre devuelve una copia**, nunca una vista — modificar el resultado no afecta al array original (ver [[09 - Copias vs Vistas]]).

## Modificación condicional in-place

```python
arr[arr < 0] = 0             # reemplaza todos los negativos por 0, MODIFICA el array original
```

## `np.where()` — selección condicional vectorizada

```python
np.where(arr > 25, "Alto", "Bajo")           # array de strings según condición — equivalente vectorizado a if/else
np.where(arr > 25, arr, 0)                     # conserva el valor original si es True, pone 0 si es False
indices = np.where(arr > 25)                    # sin segundo/tercer argumento: devuelve los ÍNDICES donde la condición es True
```

`np.where(condicion)` (un solo argumento) devuelve una tupla de arrays de índices — útil para saber **dónde** ocurre algo, no solo el valor. Con tres argumentos, funciona como un `if/else` vectorizado — es la base de `np.select` en Pandas (ver [[Python/Pandas/05 - Filtros y Condiciones Avanzadas|Pandas]]).

## `np.select()` — múltiples condiciones tipo if/elif/else

```python
condiciones = [arr > 40, arr > 20]
resultados = ["Alto", "Medio"]
np.select(condiciones, resultados, default="Bajo")
```

## Fancy indexing — indexar con un array de índices

```python
arr = np.array([10, 20, 30, 40, 50])
indices = [0, 2, 4]
arr[indices]                    # array([10, 30, 50]) — selecciona posiciones específicas, en el orden dado

arr[[4, 0, 2]]                    # array([50, 10, 30]) — el orden del índice determina el orden del resultado
arr[[0, 0, 1]]                      # array([10, 10, 20]) — los índices pueden repetirse
```

Igual que el indexado booleano, **fancy indexing siempre devuelve una copia**, nunca una vista.

## Fancy indexing en 2D

```python
matriz = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

matriz[[0, 2]]                    # filas 0 y 2 completas
matriz[[0, 1], [1, 2]]              # NO es una submatriz: toma (0,1) y (1,2) como pares -> array([2, 6])
matriz[np.ix_([0, 2], [0, 1])]       # esto SÍ da la submatriz cruzada: filas {0,2} × columnas {0,1}
```

`matriz[[0, 1], [1, 2]]` es la trampa más común de fancy indexing 2D: NumPy empareja los índices posición a posición (como `zip`), no como un producto cartesiano — para obtener la submatriz cruzada real hace falta `np.ix_()`.

## `np.take()` y `np.put()` — equivalentes funcionales

```python
np.take(arr, [0, 2, 4])           # equivalente a arr[[0, 2, 4]]
np.put(arr, [0, 1], [100, 200])    # equivalente a arr[[0, 1]] = [100, 200], pero modifica in-place
```

## Ver también

- [[05 - Indexado y Slicing]]
- [[07 - Manipulación de Forma (Reshaping)]]
- [[09 - Copias vs Vistas]]
- [[Python/Pandas/05 - Filtros y Condiciones Avanzadas|Python/Pandas]]
