---
tags: [numpy, python, data-science, cheat-sheet]
---

# 04 — Inspección y Atributos de Arrays

> Continúa de [[03 - Tipos de Datos (dtype)]]. El primer paso ante cualquier array nuevo, antes de operar sobre él.

## Atributos básicos

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.ndim         # 2 — número de dimensiones (ejes)
arr.shape         # (2, 3) — tamaño de cada dimensión
arr.size           # 6 — número total de elementos
arr.dtype           # dtype('int64')
arr.itemsize          # 8 — bytes por elemento
arr.nbytes             # 48 — bytes totales (size * itemsize)
```

## Longitud y forma

```python
len(arr)              # tamaño del PRIMER eje únicamente (equivalente a arr.shape[0])
arr.shape[0]            # número de filas, en un array 2D
arr.shape[1]             # número de columnas, en un array 2D
```

`len()` sobre un array multidimensional solo informa del primer eje — usar `.shape` para la forma completa, especialmente en arrays de 3+ dimensiones donde `len()` es ambiguo respecto a qué se está midiendo.

## Vista rápida del contenido

```python
print(arr)                  # representación completa (o truncada si es muy grande)
arr[:5]                       # primeros 5 elementos (1D) o primeras 5 filas (2D)
np.set_printoptions(precision=3, suppress=True)   # controla cómo se imprimen floats: 3 decimales, sin notación científica
```

## Inspección de memoria y contigüidad

```python
arr.flags                    # C_CONTIGUOUS, F_CONTIGUOUS, OWNDATA, WRITEABLE...
arr.strides                   # bytes a saltar por eje — ver arquitectura interna en 01
arr.base                       # None si es dueño de su memoria; si no, referencia al array del que es VISTA
```

`arr.base is None` es la forma más directa de comprobar si un array es dueño de su propia memoria o es una vista de otro array — pieza clave para razonar sobre mutabilidad compartida (ver [[09 - Copias vs Vistas]]).

## Comprobaciones de tipo y forma

```python
arr.dtype == np.float64
np.issubdtype(arr.dtype, np.integer)     # más robusto que comparar contra un dtype exacto — cubre int8/16/32/64 a la vez
arr.shape == (2, 3)
arr.flags["C_CONTIGUOUS"]
```

## Estadísticas descriptivas rápidas

```python
arr.min(), arr.max()
arr.mean(), arr.std(), arr.var()
arr.sum()
arr.argmin(), arr.argmax()      # ÍNDICE del mínimo/máximo, no el valor
```

Todas estas también existen como funciones libres (`np.min(arr)`, `np.mean(arr)`) equivalentes a los métodos — ambas formas son intercambiables, aunque el método (`arr.mean()`) es ligeramente más idiomático. Ver el detalle del parámetro `axis` en agregaciones multidimensionales en [[10 - Operaciones Matemáticas y Vectorización]].

## Ver también

- [[03 - Tipos de Datos (dtype)]]
- [[05 - Indexado y Slicing]]
- [[09 - Copias vs Vistas]]
- [[10 - Operaciones Matemáticas y Vectorización]]
