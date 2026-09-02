---
tags: [pytest, python, testing, hypothesis, cheat-sheet]
---

# 15 — Property-Based Testing con Hypothesis

> Continúa de [[14 - Testing de Bases de Datos e Infraestructura]].

## La diferencia con testing basado en ejemplos

Todo lo visto hasta ahora ([[05 - Parametrización de Tests]] incluida) es **testing basado en ejemplos**: el desarrollador elige casos específicos (`(2, 3, 5)`, `(0, 0, 0)`...) y verifica el resultado esperado para cada uno. **Property-based testing** invierte el enfoque: se declara una **propiedad** que debe cumplirse siempre (para *cualquier* entrada válida), y la librería genera automáticamente cientos de casos — incluyendo casos límite que el desarrollador probablemente no habría pensado a mano.

```python
from hypothesis import given, strategies as st

@given(st.integers(), st.integers())
def test_suma_es_conmutativa(a, b):
    assert sumar(a, b) == sumar(b, a)     # una PROPIEDAD que debe cumplirse para CUALQUIER par de enteros
```

Hypothesis ejecuta esta función cientos de veces con pares `(a, b)` generados automáticamente — incluyendo `0`, números negativos, el entero más grande representable, etc. — sin que el desarrollador tenga que enumerarlos manualmente.

## `@given` y estrategias (`strategies`)

```python
from hypothesis import strategies as st

st.integers()                          # cualquier entero
st.integers(min_value=0, max_value=100)  # rango acotado
st.floats(allow_nan=False)                # floats, excluyendo NaN explícitamente
st.text()                                   # strings arbitrarios (incluye Unicode, strings vacíos, etc.)
st.lists(st.integers())                       # listas de enteros de longitud variable
st.dictionaries(st.text(), st.integers())       # diccionarios con claves string y valores enteros
```

Cada **estrategia** describe el espacio de valores válidos de un parámetro — Hypothesis combina las estrategias declaradas en `@given` para generar automáticamente los casos de prueba, priorizando encontrar **casos límite** que rompan la propiedad (valores extremos, vacíos, cero, negativos).

## El poder real: encontrar y "reducir" (shrink) fallos automáticamente

```python
@given(st.lists(st.integers()))
def test_ordenar_preserva_longitud(lista):
    assert len(ordenar(lista)) == len(lista)
```

Si esta propiedad falla con, por ejemplo, una lista de 47 elementos generada aleatoriamente, Hypothesis no reporta esos 47 elementos tal cual — automáticamente **reduce** (shrinks) el caso fallido al ejemplo **más pequeño y simple** que sigue reproduciendo el fallo (frecuentemente algo como `[0]` o `[]`), facilitando enormemente la depuración.

## Propiedades comunes a buscar

| Tipo de propiedad | Ejemplo |
|---|---|
| Invariante | `ordenar(lista)` siempre tiene la misma longitud que `lista` |
| Idempotencia | `normalizar(normalizar(x)) == normalizar(x)` |
| Inversos | `decodificar(codificar(x)) == x` |
| Conmutatividad/Asociatividad | `sumar(a, b) == sumar(b, a)` |
| Comparación con una implementación de referencia (más lenta pero confiablemente correcta) | `mi_ordenamiento_rapido(lista) == sorted(lista)` |

## Integración con pytest

```python
from hypothesis import given, settings, strategies as st

@settings(max_examples=500, deadline=None)     # más ejemplos que el default (100), sin límite de tiempo por ejemplo
@given(st.integers(min_value=1))
def test_factorial_es_positivo(n):
    assert factorial(n) > 0
```

Los tests de Hypothesis son funciones `test_*` normales descubiertas por pytest sin configuración adicional — `@settings` controla el número de ejemplos generados (`max_examples`) y el tiempo límite por ejemplo (`deadline`), relevante para propiedades sobre operaciones costosas.

## Relación con Pandera

```python
import pandera.pandas as pa
from pandera.strategies import pandas_dtype_strategy

schema = pa.DataFrameSchema({"precio": pa.Column(float, checks=pa.Check.gt(0))})
```

Pandera usa Hypothesis internamente para **generar DataFrames sintéticos que cumplen un schema** — ver la cobertura completa de esta integración específica (generar datos de prueba realistas para pipelines de datos, no solo funciones puras) en [[Pandera/08 - Generación de Datos Sintéticos con Hypothesis|Pandera]]. Este archivo cubre Hypothesis de forma general para cualquier función Python; Pandera lo aplica específicamente al contexto de validación de DataFrames.

## Cuándo usar property-based testing

**Regla práctica:** no reemplaza los tests basados en ejemplos (que documentan casos de negocio concretos y son más legibles para alguien que lee el test), pero es un complemento poderoso para funciones puras con propiedades matemáticas/lógicas claras (parsers, serializadores, algoritmos, transformaciones de datos) — especialmente valioso para encontrar casos límite que nadie pensó en escribir manualmente.

## Ver también

- [[05 - Parametrización de Tests]]
- [[Pandera/08 - Generación de Datos Sintéticos con Hypothesis|Pandera]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
