---
tags: [optuna, hiperparametros, multi-objetivo, pareto, cheat-sheet]
---

# 06 — Optimización Multi-Objetivo

> Se apoya en [[03 - Samplers en Profundidad]].

En problemas reales frecuentemente hay más de una métrica que importa simultáneamente — por ejemplo, minimizar el error de un modelo **y** minimizar su latencia de inferencia, o minimizar error **y** minimizar el tamaño del modelo. Optuna soporta esto de forma nativa, sin necesidad de combinar las métricas en una sola función escalar de forma manual.

## El problema de combinar métricas a mano

Un enfoque ingenuo es crear una métrica compuesta:

```python
# Enfoque frágil — el peso 0.7/0.3 es una decisión arbitraria, difícil de justificar
def objective(trial):
    mae = entrenar_y_evaluar_error(trial)
    latencia_ms = medir_latencia(trial)
    return 0.7 * mae + 0.3 * (latencia_ms / 100)   # ¿por qué esos pesos y no otros?
```

El problema: los pesos son arbitrarios, y distintas combinaciones de pesos llevan a distintos "óptimos" — se pierde información sobre el trade-off real entre ambos objetivos.

## `directions` — múltiples objetivos, cada uno con su propia dirección

```python
import optuna

def objective(trial):
    params = sugerir_parametros(trial)
    modelo = entrenar(params)

    mae = evaluar_error(modelo)
    latencia_ms = medir_latencia_inferencia(modelo)

    return mae, latencia_ms   # retorna una TUPLA, no un solo valor

study = optuna.create_study(directions=["minimize", "minimize"])
study.optimize(objective, n_trials=200)
```

Nótese: se usa `directions` (plural) en vez de `direction`, y la función objetivo retorna una tupla con un valor por objetivo.

## La Frontera de Pareto — no hay un único "mejor" trial

Con múltiples objetivos en conflicto, no existe una sola combinación que sea la mejor en todo — existe un conjunto de soluciones **Pareto-óptimas**: trials donde no se puede mejorar un objetivo sin empeorar otro.

```python
pareto_trials = study.best_trials   # lista de trials en la frontera de Pareto (no un solo "best_trial")

for trial in pareto_trials:
    print(f"Trial {trial.number}: MAE={trial.values[0]:.3f}, Latencia={trial.values[1]:.1f}ms")
    print(f"  Params: {trial.params}")
```

`study.best_trials` (plural) reemplaza a `study.best_trial` (singular, que solo existe en optimización de un objetivo) — retorna todos los trials que forman la frontera de Pareto, para que el humano elija el trade-off que mejor se ajuste al contexto de negocio.

## Visualizar la Frontera de Pareto

```python
import optuna.visualization as vis

fig = vis.plot_pareto_front(study, target_names=["MAE", "Latencia (ms)"])
fig.show()
```

Genera un scatter plot 2D (o 3D con tres objetivos) donde cada punto es un trial, y los puntos en la frontera de Pareto se resaltan visualmente — permite decidir manualmente el trade-off (ej. "acepto 2ms más de latencia a cambio de 5% menos de error") en vez de que un peso arbitrario lo decida de antemano.

## `NSGAIISampler` — el sampler recomendado para multi-objetivo

```python
from optuna.samplers import NSGAIISampler

study = optuna.create_study(
    directions=["minimize", "minimize"],
    sampler=NSGAIISampler(
        population_size=50,     # tamaño de la "población" del algoritmo genético
        seed=42,
    ),
)
```

`TPESampler` también soporta multi-objetivo desde versiones recientes de Optuna, pero `NSGAIISampler` (Non-dominated Sorting Genetic Algorithm II) fue diseñado específicamente para este caso y suele converger a una frontera de Pareto más diversa y representativa.

## Tres o más objetivos — `NSGAIIISampler`

```python
from optuna.samplers import NSGAIIISampler

def objective(trial):
    params = sugerir_parametros(trial)
    modelo = entrenar(params)
    return evaluar_error(modelo), medir_latencia(modelo), medir_tamano_modelo_mb(modelo)

study = optuna.create_study(
    directions=["minimize", "minimize", "minimize"],
    sampler=NSGAIIISampler(),
)
```

`NSGAIII` extiende `NSGAII` para escalar mejor cuando hay muchos objetivos simultáneos (más de 3), usando puntos de referencia distribuidos en el espacio de objetivos para mantener diversidad en la frontera.

## Elegir un trial final de la frontera de Pareto

Una vez obtenida la frontera, la elección final suele ser una decisión de negocio, no puramente técnica:

```python
# Ejemplo: de todos los Pareto-óptimos, elegir el de menor error
# entre los que cumplen una restricción dura de latencia
candidatos_validos = [t for t in study.best_trials if t.values[1] <= 50.0]  # latencia <= 50ms
mejor = min(candidatos_validos, key=lambda t: t.values[0])   # el de menor MAE entre esos

print(f"Modelo elegido: {mejor.params}")
```

## Pruning en contextos multi-objetivo

El soporte de pruning con múltiples objetivos es más limitado que en el caso de un solo objetivo — no todos los pruners funcionan igual de bien. En la práctica, es común desactivar pruning en búsquedas multi-objetivo o usarlo con cautela, verificando que el pruner elegido soporte explícitamente el caso multi-objetivo en la versión de Optuna instalada.

## Ver también

- [[03 - Samplers en Profundidad]]
- [[07 - Visualización y Análisis de Resultados]]
- [[09 - Callbacks, Constraints y Configuración Avanzada]] (restricciones/constraints)
