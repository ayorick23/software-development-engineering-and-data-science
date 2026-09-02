---
tags: [optuna, hiperparametros, samplers, tpe, cheat-sheet]
---

# 03 — Samplers en Profundidad

> Continúa de [[02 - Definición del Espacio de Búsqueda]].

Un **sampler** es el algoritmo que decide qué combinación de hiperparámetros proponer en cada trial. Es el corazón de Optuna — cambiar de sampler cambia radicalmente la eficiencia de la búsqueda sin tocar el resto del código.

```python
study = optuna.create_study(direction="minimize", sampler=optuna.samplers.TPESampler())
```

## `TPESampler` — el default, optimización bayesiana

**Tree-structured Parzen Estimator.** Modela dos distribuciones a partir del historial de trials: `l(x)` (parámetros que dieron *buenos* resultados) y `g(x)` (parámetros que dieron *malos* resultados). Propone el siguiente trial maximizando el ratio `l(x)/g(x)` — es decir, busca en regiones donde es mucho más probable estar en el grupo "bueno" que en el "malo".

```python
from optuna.samplers import TPESampler

sampler = TPESampler(
    n_startup_trials=10,     # primeros N trials son aleatorios puros, antes de empezar a modelar
    n_ei_candidates=24,      # cuántos candidatos evalúa internamente antes de elegir el mejor
    seed=42,                 # reproducibilidad
    multivariate=True,       # modela correlaciones ENTRE hiperparámetros, no cada uno por separado
    group=True,              # agrupa parámetros condicionales relacionados (útil con espacios dinámicos)
)
study = optuna.create_study(direction="minimize", sampler=sampler)
```

- `n_startup_trials`: los primeros trials son puramente aleatorios porque TPE necesita datos previos antes de poder modelar nada útil — sin esto, el sampler "aprendería" de una muestra demasiado pequeña y sesgada.
- `multivariate=True` (recomendado desde Optuna 3.x): en vez de tratar cada hiperparámetro de forma independiente, modela la distribución conjunta — captura que, por ejemplo, un `learning_rate` alto suele ir bien con un `n_estimators` bajo.

## `RandomSampler` — línea base

```python
from optuna.samplers import RandomSampler
study = optuna.create_study(sampler=RandomSampler(seed=42))
```

Muestrea uniformemente del espacio de búsqueda, sin aprender de trials anteriores. Útil como **baseline** para verificar que TPE realmente está aportando valor sobre búsqueda aleatoria en tu problema específico, y en espacios de búsqueda muy pequeños donde el overhead de modelar no compensa.

## `GridSampler` — grilla exhaustiva, pero con la API de Optuna

```python
from optuna.samplers import GridSampler

search_space = {
    "n_estimators": [100, 300, 500],
    "max_depth": [3, 6, 9],
}
study = optuna.create_study(sampler=GridSampler(search_space))
study.optimize(objective, n_trials=9)   # 3 × 3 combinaciones — como GridSearchCV, pero con logging/pruning de Optuna
```

Reproduce el comportamiento de `GridSearchCV`, pero se beneficia de la infraestructura de Optuna (pruning, paralelización, visualización). El espacio debe ser discreto y completo — no acepta rangos continuos.

## `CmaEsSampler` — para espacios continuos de alta dimensionalidad

```python
from optuna.samplers import CmaEsSampler
study = optuna.create_study(sampler=CmaEsSampler(seed=42))
```

**CMA-ES (Covariance Matrix Adaptation Evolution Strategy)**: un algoritmo evolutivo que ajusta una distribución gaussiana multivariada sobre el espacio de búsqueda, adaptando su forma (covarianza) según qué direcciones han dado mejores resultados. Tiende a superar a TPE en espacios **puramente continuos con muchos hiperparámetros** (deep learning con decenas de parámetros numéricos), pero no maneja bien hiperparámetros categóricos ni espacios condicionales — en esos casos, TPE sigue siendo mejor opción.

## `NSGAIISampler` / `NSGAIIISampler` — optimización multi-objetivo

```python
from optuna.samplers import NSGAIISampler
study = optuna.create_study(directions=["minimize", "minimize"], sampler=NSGAIISampler())
```

Algoritmo genético diseñado específicamente para cuando hay **más de un objetivo simultáneo** (ej. minimizar error y minimizar latencia de inferencia a la vez). Se profundiza en [[06 - Optimización Multi-Objetivo]].

## `QMCSampler` — Quasi-Monte Carlo

```python
from optuna.samplers import QMCSampler
study = optuna.create_study(sampler=QMCSampler(seed=42))
```

Usa secuencias de baja discrepancia (ej. Sobol) en vez de muestreo puramente aleatorio, logrando una cobertura más uniforme del espacio con menos trials que `RandomSampler` — un punto intermedio razonable cuando TPE no aplica bien (por ejemplo, para generar el conjunto inicial de `n_startup_trials`).

## `PartialFixedSampler` — fijar algunos parámetros, buscar el resto

```python
from optuna.samplers import PartialFixedSampler, TPESampler

base_sampler = TPESampler()
sampler = PartialFixedSampler(
    fixed_params={"n_estimators": 300},   # este valor NUNCA cambia entre trials
    base_sampler=base_sampler,             # el resto de hiperparámetros sí se buscan normalmente
)
```

Útil cuando un hiperparámetro ya está bien calibrado (por ejemplo, se decidió por restricciones de latencia en producción) y no tiene sentido seguir explorándolo.

## Cómo elegir sampler — tabla de decisión

| Situación | Sampler recomendado |
|---|---|
| Caso general, primera búsqueda | `TPESampler` (default) |
| Espacio de búsqueda pequeño (< 1000 combinaciones posibles), quieres exhaustividad | `GridSampler` |
| Baseline para comparar contra TPE | `RandomSampler` |
| Muchos hiperparámetros continuos (deep learning), pocos/ningún categórico | `CmaEsSampler` |
| Más de un objetivo a optimizar simultáneamente | `NSGAIISampler` |
| Cobertura inicial más uniforme que aleatoria pura | `QMCSampler` |

## Reproducibilidad — el parámetro `seed`

```python
sampler = TPESampler(seed=42)
study = optuna.create_study(direction="minimize", sampler=sampler)
```

Sin `seed`, cada ejecución del estudio explora un orden distinto de trials — importante fijarlo cuando se necesita reproducir exactamente los mismos resultados (ej. en tests automatizados del pipeline de tuning). Nótese que `seed` garantiza reproducibilidad del **sampler**, no necesariamente del entrenamiento del modelo en sí (que puede tener su propia aleatoriedad interna — semillas de `train_test_split`, inicialización de pesos, etc.).

## Ver también

- [[02 - Definición del Espacio de Búsqueda]]
- [[04 - Pruners en Profundidad]]
- [[06 - Optimización Multi-Objetivo]]
- [[11 - Buenas Prácticas, Errores Comunes y Comparativa]]
