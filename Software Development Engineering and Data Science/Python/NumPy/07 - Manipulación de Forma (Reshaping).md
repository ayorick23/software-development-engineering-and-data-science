---
tags: [numpy, python, data-science, reshape, cheat-sheet]
---

# 07 — Manipulación de Forma (Reshaping)

> Continúa de [[06 - Indexado Avanzado]].

## `reshape()` — cambiar la forma sin cambiar los datos

```python
arr = np.arange(12)              # [0, 1, ..., 11]
matriz = arr.reshape(3, 4)         # 3 filas, 4 columnas
matriz = arr.reshape(3, -1)          # -1: NumPy calcula automáticamente esa dimensión (aquí, 4)
cubo = arr.reshape(2, 2, 3)            # también funciona para 3+ dimensiones
```

`reshape()` requiere que el número total de elementos coincida exactamente (`3*4 == 12`) — y devuelve, siempre que sea posible, una **vista** del array original, no una copia (ver [[09 - Copias vs Vistas]]).

## Aplanar: `ravel()` vs `flatten()`

```python
matriz.ravel()       # devuelve VISTA cuando es posible — más rápido, comparte memoria
matriz.flatten()      # devuelve SIEMPRE una COPIA — más seguro si se va a modificar el resultado sin afectar el original
```

| | `ravel()` | `flatten()` |
|---|---|---|
| Memoria | Vista (si es posible) | Siempre copia |
| Velocidad | Más rápida | Más lenta (copia todo) |
| Seguridad | Modificar el resultado puede afectar el original | Independiente, seguro |

## Transponer y reordenar ejes

```python
matriz.T                       # transponer — para 2D, intercambia filas y columnas (es una VISTA, solo recalcula strides)
np.transpose(matriz)             # equivalente explícito
cubo.transpose(2, 0, 1)            # para 3+D: reordena los ejes según los índices dados
np.swapaxes(cubo, 0, 1)              # intercambia dos ejes específicos
```

## `expand_dims()` y `squeeze()` — agregar/quitar dimensiones de tamaño 1

```python
arr = np.array([1, 2, 3])          # shape (3,)
np.expand_dims(arr, axis=0)          # shape (1, 3) — útil para que un vector "encaje" en operaciones que esperan 2D
np.expand_dims(arr, axis=1)           # shape (3, 1)
arr[:, np.newaxis]                      # forma alternativa equivalente a expand_dims(arr, axis=1), muy usada en la práctica

matriz_de_una_fila = np.array([[1, 2, 3]])   # shape (1, 3)
matriz_de_una_fila.squeeze()                    # shape (3,) — elimina TODAS las dimensiones de tamaño 1
```

## Concatenar y apilar arrays

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.concatenate([a, b])                # [1, 2, 3, 4, 5, 6] — une a lo largo de un eje EXISTENTE
np.vstack([a, b])                       # apila verticalmente -> array([[1,2,3],[4,5,6]]) (crea un eje nuevo)
np.hstack([a, b])                        # apila horizontalmente -> [1,2,3,4,5,6] (igual que concatenate en 1D)
np.stack([a, b], axis=0)                   # crea un eje NUEVO explícito -> shape (2, 3)
np.column_stack([a, b])                      # apila como columnas -> shape (3, 2)
```

`concatenate` une a lo largo de un eje que **ya existe** en los arrays de entrada; `stack` siempre crea un eje **nuevo** — esa es la diferencia conceptual clave entre ambas familias de funciones.

## Dividir arrays

```python
arr = np.arange(9)
np.split(arr, 3)                     # 3 partes iguales -> [array([0,1,2]), array([3,4,5]), array([6,7,8])]
np.split(arr, [3, 7])                  # cortes en posiciones específicas -> partes de tamaño 3, 4, 2
np.array_split(arr, 4)                   # como split, pero TOLERA tamaños desiguales (split exige división exacta)

matriz = np.arange(16).reshape(4, 4)
np.hsplit(matriz, 2)                       # divide en columnas
np.vsplit(matriz, 2)                         # divide en filas
```

## Ver también

- [[06 - Indexado Avanzado]]
- [[08 - Broadcasting]]
- [[09 - Copias vs Vistas]]
