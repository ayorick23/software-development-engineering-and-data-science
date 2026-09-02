---
tags: [numpy, python, data-science, ufuncs, cheat-sheet]
---

# 10 — Operaciones Matemáticas y Vectorización

> Continúa de [[09 - Copias vs Vistas]].

## Aritmética elemento a elemento

```python
a = np.array([1, 2, 3])
b = np.array([10, 20, 30])

a + b, a - b, a * b, a / b, a ** 2, a % 2
```

Todas estas operaciones son **ufuncs** (*universal functions*): funciones de NumPy escritas en C que operan elemento por elemento, con soporte automático de [[08 - Broadcasting|broadcasting]] entre formas compatibles.

## Funciones matemáticas element-wise

```python
np.sqrt(a)
np.exp(a)                # e^x
np.log(a)                 # logaritmo natural
np.log10(a), np.log2(a)
np.sin(a), np.cos(a), np.tan(a)
np.abs(a)
np.round(a, decimals=2)
np.clip(a, a_min=0, a_max=10)    # recorta valores fuera del rango [0, 10]
```

## Agregaciones y el parámetro `axis`

```python
matriz = np.array([[1, 2, 3], [4, 5, 6]])

matriz.sum()             # 21 — suma TODOS los elementos
matriz.sum(axis=0)         # [5, 7, 9]  — suma A LO LARGO de las filas -> un resultado por COLUMNA
matriz.sum(axis=1)          # [6, 15]     — suma a lo largo de las columnas -> un resultado por FILA
```

**La regla mnemotécnica de `axis`:** `axis=0` colapsa las **filas** (el resultado tiene una entrada por columna); `axis=1` colapsa las **columnas** (una entrada por fila). Es, en la práctica, el punto de confusión más común de NumPy y Pandas para quien recién empieza — pensarlo como "la dimensión que desaparece", no como "la dirección en la que se mueve".

```python
matriz.mean(axis=0), matriz.std(axis=0), matriz.min(axis=0), matriz.max(axis=0)
matriz.cumsum(axis=1)        # suma acumulada a lo largo de cada fila
matriz.cumprod()               # producto acumulado (aplanado si no se especifica axis)
```

## Producto y operaciones de álgebra lineal básicas

```python
np.dot(a, b)              # producto punto (1D) o producto matricial (2D)
a @ b                       # operador @ — equivalente moderno y preferido a np.dot para matrices
np.cross(a, b)                # producto cruz (vectores 3D)
```

Ver el catálogo completo de álgebra lineal (inversa, determinante, eigenvalores, SVD) en [[11 - Álgebra Lineal (np.linalg)]].

## Operadores de comparación (devuelven arrays booleanos)

```python
a > 2                    # array([False, False, True])
a == b
np.array_equal(a, b)       # compara los arrays COMPLETOS, devuelve un único booleano
np.allclose(a, b, atol=1e-8)   # comparación con tolerancia — esencial para floats (evita comparar == directo)
```

**Regla práctica:** nunca comparar floats con `==` directamente (errores de precisión de punto flotante) — usar `np.allclose()` o `np.isclose()` para comparaciones elemento a elemento con tolerancia.

## `np.vectorize()` — vectorizar una función Python arbitraria

```python
def clasificar(x):
    return "alto" if x > 50 else "bajo"

clasificar_vec = np.vectorize(clasificar)
clasificar_vec(np.array([10, 60, 30, 90]))    # array(['bajo', 'alto', 'bajo', 'alto'])
```

`np.vectorize()` es una **conveniencia de sintaxis**, no una optimización real de rendimiento — internamente sigue ejecutando la función Python elemento por elemento en un loop. Para rendimiento real con lógica condicional, preferir `np.where()`/`np.select()` (ver [[06 - Indexado Avanzado]]) siempre que la lógica lo permita.

## Ufuncs con argumento `out` — evitar allocaciones intermedias

```python
resultado = np.empty_like(a)
np.multiply(a, 2, out=resultado)     # escribe directo en 'resultado', sin crear un array temporal nuevo
```

Útil en loops de procesamiento intensivo donde se quiere reutilizar el mismo buffer de memoria en cada iteración en vez de que cada operación asigne memoria nueva — ver más patrones de rendimiento en [[15 - Rendimiento y Vectorización Avanzada]].

## Ver también

- [[08 - Broadcasting]]
- [[11 - Álgebra Lineal (np.linalg)]]
- [[12 - Comparación, Ordenamiento y Búsqueda]]
- [[15 - Rendimiento y Vectorización Avanzada]]
