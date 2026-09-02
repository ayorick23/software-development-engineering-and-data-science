---
tags: [numpy, python, data-science, memory, cheat-sheet]
---

# 09 — Copias vs Vistas

> Continúa de [[08 - Broadcasting]]. La fuente número uno de bugs sutiles en código NumPy.

## El problema en una línea

```python
original = np.array([1, 2, 3, 4, 5])
sub = original[1:3]      # ¿esto es independiente de 'original', o comparte memoria?
sub[0] = 999
print(original)            # [1, 999, 3, 4, 5]  <- ¡SÍ cambió! sub era una VISTA
```

## Qué operaciones devuelven vista vs copia

| Operación | Resultado |
|---|---|
| Slicing básico (`arr[1:3]`) | **Vista** |
| `reshape()` | **Vista** (cuando la memoria es compatible) |
| `.T` / `transpose()` | **Vista** (solo recalcula strides) |
| `ravel()` | **Vista** (cuando es posible) |
| Indexado booleano (`arr[arr > 5]`) | **Copia** siempre |
| Fancy indexing (`arr[[0, 2, 4]]`) | **Copia** siempre |
| `flatten()` | **Copia** siempre |
| `.copy()` | **Copia** explícita, siempre |
| Operaciones aritméticas (`arr + 1`) | **Copia** — crea un array nuevo con el resultado |

**Regla práctica:** el slicing con `:` y el reshape casi siempre devuelven vistas (comparten memoria); el indexado con listas/arrays de índices o con máscaras booleanas siempre devuelve copias.

## Verificar si algo es vista o copia

```python
sub = original[1:3]
sub.base is None          # False -> es una VISTA (tiene una referencia a su 'dueño')
sub.base is original        # True -> específicamente, vista DE 'original'

copia = original[[0, 1]]
copia.base is None           # True -> es dueña de su propia memoria (fue una copia)
```

## Forzar una copia cuando se necesita independencia

```python
independiente = original[1:3].copy()
independiente[0] = -1     # NO afecta a 'original'
```

**Regla práctica:** si se va a modificar un subconjunto de un array y el original **no** debe cambiar, usar `.copy()` explícitamente — depender de que "tal vez" devuelva copia es una apuesta, no una garantía.

## `reshape()` no siempre puede devolver vista

```python
arr = np.arange(12).reshape(3, 4)
transpuesta = arr.T
transpuesta.reshape(12)          # ValueError o COPIA silenciosa según el caso — la memoria transpuesta no es contigua
```

Cuando la memoria subyacente no es contigua en el orden pedido (por ejemplo, después de una transposición), `reshape()` no puede simplemente reinterpretar los strides — NumPy copia los datos automáticamente para poder cumplir la forma solicitada. Verificar con `arr.flags["C_CONTIGUOUS"]` cuándo esto puede ocurrir.

## `np.shares_memory()` — diagnóstico explícito

```python
np.shares_memory(original, sub)          # True si comparten el mismo bloque de memoria
np.may_share_memory(original, sub)        # versión más rápida pero conservadora (puede dar falsos positivos)
```

## Por qué esto importa en funciones

```python
def procesar(arr):
    arr[0] = 0          # BUG SILENCIOSO si 'arr' es una vista del array que llamó a la función
    return arr

datos = np.arange(10)
resultado = procesar(datos[2:5])   # datos[2:5] es una VISTA -> 'datos' original también se modificó
```

Al escribir funciones que reciben arrays de NumPy y los modifican in-place, es responsabilidad de quien llama la función saber si el argumento es una vista o no — una función "pura" que no debe tener efectos secundarios debe hacer `arr = arr.copy()` como primera línea si va a mutar su entrada.

## Ver también

- [[Python/NumPy/01 - Introducción y Arquitectura Interna]]
- [[05 - Indexado y Slicing]]
- [[07 - Manipulación de Forma (Reshaping)]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
