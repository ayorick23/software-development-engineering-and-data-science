---
tags: [numpy, python, data-science, best-practices, cheat-sheet]
---

# 17 — Buenas Prácticas, Errores Comunes y Comparativa

> Cierra la serie iniciada en [[Python/NumPy/01 - Introducción y Arquitectura Interna]].

## Mutación accidental a través de una vista

```python
def normalizar(arr):
    arr -= arr.mean()      # BUG: si 'arr' es una vista, esto muta el array del LLAMADOR también
    return arr

datos = np.arange(10.0)
sub = datos[2:8]
normalizar(sub)              # 'datos' original también cambió, probablemente sin que se esperara
```

```python
# CORRECTO — copiar explícitamente al entrar a una función que no debe tener efectos secundarios
def normalizar(arr):
    arr = arr.copy()
    arr -= arr.mean()
    return arr
```

Ver el detalle completo de qué operaciones devuelven vista vs copia en [[09 - Copias vs Vistas]].

## Confundir `*` (elemento a elemento) con `@` (matricial)

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

A * B     # [[5, 12], [21, 32]]  — elemento a elemento
A @ B      # [[19, 22], [43, 50]] — producto matricial real
```

Como ambas operaciones son válidas sintácticamente sobre matrices cuadradas del mismo tamaño, este error **no lanza excepción** — simplemente produce un resultado numéricamente incorrecto que puede pasar desapercibido. Ver [[11 - Álgebra Lineal (np.linalg)]].

## Comparar floats con `==`

```python
# INCORRECTO — errores de precisión de punto flotante
0.1 + 0.2 == 0.3        # False

# CORRECTO
np.isclose(0.1 + 0.2, 0.3)       # True
np.allclose(arr_a, arr_b, atol=1e-8)
```

## Overflow silencioso en dtypes pequeños

```python
arr = np.array([200], dtype=np.uint8)
arr * 2          # array([144], dtype=uint8) — 400 no cabe en uint8 (max 255), da la vuelta SIN error
```

Ver el detalle completo en [[03 - Tipos de Datos (dtype)]] — downcasting de memoria sin verificar el rango real de los datos es una fuente recurrente de bugs silenciosos, especialmente en pipelines que combinan NumPy con optimización de memoria de Pandas.

## Broadcasting silencioso hacia una forma no intencionada

```python
a = np.array([1, 2, 3])           # shape (3,)
b = np.array([[1], [2], [3]])       # shape (3, 1)

a + b
# array([[2, 3, 4],
#        [3, 4, 5],
#        [4, 5, 6]])    <- shape (3, 3), probablemente NO lo que se esperaba (a menudo se espera shape (3,))
```

Este resultado **no es un error** — es broadcasting funcionando exactamente según sus reglas (ver [[08 - Broadcasting]]), pero casi nunca es la intención real cuando ocurre por accidente. Revisar `.shape` de los operandos antes de una operación aritmética entre arrays de formas "casi iguales" evita esta clase de sorpresa.

## `np.append()`/`np.concatenate()` dentro de un loop

Ya cubierto en detalle en [[15 - Rendimiento y Vectorización Avanzada#Preasignar memoria en vez de crecer arrays iterativamente|Rendimiento]] — recordatorio rápido: preasignar con `np.empty()`/`np.zeros()` y llenar por índice, o vectorizar directamente, en vez de crecer un array iterativamente.

## Checklist antes de dar por "listo" código NumPy en un pipeline

- [ ] ¿Las funciones que reciben arrays y los modifican hacen `.copy()` explícito si no deben mutar el argumento?
- [ ] ¿Se usó `@` para producto matricial y `*` solo para elemento a elemento, intencionalmente?
- [ ] ¿Las comparaciones de floats usan `np.isclose`/`np.allclose`, nunca `==` directo?
- [ ] ¿Los dtypes reducidos (`int8`, `float32`...) fueron verificados contra el rango real de los datos?
- [ ] ¿Hay algún loop de Python que en realidad se pueda vectorizar?

## Comparativa: NumPy vs alternativas

| | NumPy | CuPy | Numba | Dask Array |
|---|---|---|---|---|
| Dónde corre | CPU | GPU | CPU (JIT-compilado) | CPU, distribuido |
| API | Estándar de facto | Casi idéntica a NumPy | Decoradores sobre funciones Python | Calcada de NumPy, perezosa |
| Cuándo usar | Default para todo lo que quepa en memoria | Cómputo masivo con GPU disponible | Loops irreducibles que no se pueden vectorizar | Arrays más grandes que la RAM |
| Curva de aprendizaje | — | Mínima si ya se sabe NumPy | Baja (decorador + tipos) | Baja (misma API, `.compute()` al final) |

**Regla práctica:** empezar siempre en NumPy puro. Migrar a Numba solo para el puñado de funciones con loops verdaderamente no vectorizables. Migrar a CuPy cuando el cuello de botella es cómputo masivo y hay GPU disponible. Migrar a Dask Array cuando el dataset ya no cabe en RAM.

## Ver también

- [[Python/NumPy/01 - Introducción y Arquitectura Interna]]
- [[08 - Broadcasting]]
- [[09 - Copias vs Vistas]]
- [[15 - Rendimiento y Vectorización Avanzada]]
- [[Python/Pandas/17 - Buenas Prácticas, Errores Comunes y Comparativa|Python/Pandas]]
