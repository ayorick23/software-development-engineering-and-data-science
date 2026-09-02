---
tags: [numpy, python, data-science, dtype, cheat-sheet]
---

# 03 — Tipos de Datos (dtype)

> Continúa de [[02 - Creación de Arrays]].

## El catálogo de `dtype`

| Categoría | Ejemplos | Notas |
|---|---|---|
| Enteros con signo | `int8`, `int16`, `int32`, `int64` | El número indica bits — `int8` va de -128 a 127 |
| Enteros sin signo | `uint8`, `uint16`, `uint32`, `uint64` | Sin negativos, rango positivo el doble de amplio |
| Punto flotante | `float16`, `float32`, `float64` | `float64` es el default de Python/NumPy |
| Complejos | `complex64`, `complex128` | Parte real + imaginaria |
| Booleano | `bool_` | 1 byte por elemento (no 1 bit) |
| Texto | `str_` (Unicode), `bytes_` | Tamaño fijo por array — ver advertencia abajo |
| Genérico Python | `object` | Caja para cualquier objeto Python — pierde las ventajas de NumPy |

```python
arr.dtype                          # inspeccionar
np.array([1, 2, 3], dtype=np.int32)   # especificar en la creación
np.dtype(np.float64).itemsize          # bytes por elemento -> 8
```

## Casting con `astype()`

```python
arr = np.array([1.7, 2.3, 3.9])
arr.astype(int)          # [1, 2, 3] — TRUNCA, no redondea
np.round(arr).astype(int)   # [2, 2, 4] — forma correcta de redondear antes de convertir

arr_str = np.array(["1", "2", "3"])
arr_str.astype(int)        # [1, 2, 3] — conversión de string a numérico
```

`astype()` **siempre devuelve una copia** nueva por default (`copy=True`); el array original nunca se modifica in-place por una conversión de tipo.

## El peligro del overflow silencioso

```python
arr = np.array([200], dtype=np.int8)
arr + 100
# array([44], dtype=int8)   <- ¡se desbordó silenciosamente! 200+100=300, pero int8 solo llega a 127
```

A diferencia de Python puro (donde los enteros tienen precisión arbitraria), los enteros de NumPy tienen **rango fijo** según su dtype — una operación que excede ese rango **no lanza error**, simplemente da la vuelta (*wraps around*). Elegir un dtype demasiado pequeño para downcasting de memoria (ver [[15 - Rendimiento y Vectorización Avanzada]]) sin verificar el rango real de los datos es una fuente común de bugs silenciosos.

## `NaN` y tipos flotantes

```python
np.nan                       # float('nan') — SOLO existe en tipos flotantes, no en enteros
np.array([1, 2, np.nan])      # fuerza el dtype completo a float64, aunque 1 y 2 sean "enteros"
np.isnan(arr)                  # máscara booleana de NaN
np.nansum(arr), np.nanmean(arr)   # agregaciones que IGNORAN NaN (a diferencia de sum/mean normales)
```

`NaN` no puede representarse en un array `int` — es la misma razón por la que una columna entera de Pandas con nulos se "degrada" a `float64` (ver [[Python/Pandas/01 - Introducción y Arquitectura Interna|Pandas]]).

## Arrays de texto: la trampa del tamaño fijo

```python
arr = np.array(["ana", "juan", "elizabeth"])
arr.dtype        # dtype('<U9') — reservó 9 caracteres para CADA elemento, según el string más largo

arr[0] = "alexandra"   # se trunca a 9 caracteres: 'alexandr'
```

A diferencia de una lista de Python (donde cada string puede tener cualquier longitud), un array NumPy de strings reserva un ancho **fijo** determinado por el elemento más largo en el momento de la creación — asignar un string más largo después lo trunca silenciosamente. Para texto de longitud variable en un contexto tabular, usar el dtype `string` de Pandas (ver [[Python/Pandas/01 - Introducción y Arquitectura Interna#El sistema de dtype NumPy vs Extension Arrays|Extension Arrays de Pandas]]) es preferible.

## Arrays estructurados — filas heterogéneas tipo "tabla"

```python
dtype_estructurado = np.dtype([("nombre", "U10"), ("edad", "i4"), ("salario", "f8")])
empleados = np.array([("Ana", 28, 45000.0), ("Luis", 35, 52000.0)], dtype=dtype_estructurado)

empleados["nombre"]     # array(['Ana', 'Luis'], dtype='<U10')
empleados[0]["edad"]     # 28
```

Los arrays estructurados permiten filas con **columnas de distinto tipo** dentro de un solo `ndarray` — es el antecesor conceptual directo de un DataFrame de Pandas, aunque en la práctica casi siempre conviene usar Pandas directamente para este caso de uso en vez de arrays estructurados "a mano".

## Ver también

- [[02 - Creación de Arrays]]
- [[Python/NumPy/01 - Introducción y Arquitectura Interna]]
- [[15 - Rendimiento y Vectorización Avanzada]]
- [[Python/Pandas/01 - Introducción y Arquitectura Interna|Python/Pandas]]
