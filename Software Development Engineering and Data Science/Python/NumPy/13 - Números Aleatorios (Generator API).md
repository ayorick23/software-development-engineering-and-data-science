---
tags: [numpy, python, data-science, random, cheat-sheet]
---

# 13 — Números Aleatorios (Generator API)

> Continúa de [[12 - Comparación, Ordenamiento y Búsqueda]].

## La API moderna: `Generator`

Desde NumPy 1.17, la forma recomendada de generar números aleatorios es a través de un objeto `Generator`, creado con `np.random.default_rng()` — reemplaza a las funciones legacy `np.random.seed()` / `np.random.rand()` / `np.random.randint()` que siguen funcionando por compatibilidad, pero ya no son la forma recomendada.

```python
rng = np.random.default_rng(seed=42)   # 'seed' fija la reproducibilidad
```

| | API legacy (`np.random.*`) | API moderna (`Generator`) |
|---|---|---|
| Estado aleatorio | Global, compartido en todo el programa | Local al objeto `rng` — aislado y explícito |
| Reproducibilidad | `np.random.seed(42)` afecta TODO el código | `np.random.default_rng(42)` afecta solo ese `rng` |
| Calidad estadística | Mersenne Twister (más antiguo) | PCG64 (default), más rápido y con mejores propiedades |
| Recomendación oficial | Deprecado para código nuevo | **Preferido** desde NumPy 1.17+ |

**Por qué importa el estado global vs local:** con la API legacy, llamar `np.random.seed(42)` en cualquier parte del código (incluida una librería importada) afecta la secuencia aleatoria de **todo** el programa — una fuente clásica de bugs de reproducibilidad difíciles de rastrear. Un objeto `Generator` aislado evita ese acoplamiento oculto.

## Distribuciones más comunes

```python
rng.random(5)                          # 5 floats uniformes en [0, 1)
rng.random((3, 3))                       # matriz 3x3

rng.integers(0, 10, size=5)               # 5 enteros aleatorios en [0, 10)
rng.integers(0, 10, size=5, endpoint=True)  # incluye el 10 también

rng.normal(loc=0, scale=1, size=1000)       # distribución normal (media, desviación estándar)
rng.uniform(low=0, high=100, size=5)          # uniforme continua en un rango arbitrario
rng.binomial(n=10, p=0.5, size=1000)            # distribución binomial
rng.poisson(lam=3, size=1000)                     # distribución de Poisson
rng.exponential(scale=1.0, size=1000)               # distribución exponencial
```

## Selección y mezcla aleatoria

```python
opciones = np.array(["A", "B", "C", "D"])

rng.choice(opciones)                        # un elemento aleatorio
rng.choice(opciones, size=3)                  # 3 elementos, CON reemplazo por default
rng.choice(opciones, size=3, replace=False)     # SIN reemplazo — como un muestreo sin repetición
rng.choice(opciones, size=3, p=[0.1, 0.1, 0.7, 0.1])   # con probabilidades específicas por elemento

arr = np.arange(10)
rng.shuffle(arr)                              # mezcla IN-PLACE, modifica arr directamente
permutada = rng.permutation(arr)                # devuelve una NUEVA copia mezclada, no modifica arr
```

`shuffle()` muta el array original; `permutation()` devuelve uno nuevo — la misma distinción conceptual que `sort()` vs `np.sort()` (ver [[12 - Comparación, Ordenamiento y Búsqueda]]).

## Reproducibilidad en experimentos de ML

```python
rng = np.random.default_rng(seed=42)
indices_entrenamiento = rng.choice(len(dataset), size=int(0.8 * len(dataset)), replace=False)
```

Fijar una semilla (`seed`) es indispensable para que un experimento de Machine Learning sea reproducible — ver el mismo concepto aplicado a nivel de todo un pipeline (no solo NumPy) en [[04 - Model Selection - Validación y Búsqueda|Scikit-learn]] (`random_state`) y en [[01 - Introducción y Arquitectura General|MLflow]] para registrar esa semilla como parte de la trazabilidad del experimento.

## Ver también

- [[12 - Comparación, Ordenamiento y Búsqueda]]
- [[02 - Creación de Arrays]]
- [[04 - Model Selection - Validación y Búsqueda|Scikit-learn]]
