---
tags: [dvc, mlops, experiments, cheat-sheet]
---

# 06 — DVC Experiments: Iteración Rápida sin Comprometer Git

> Continúa de [[05 - Params, Metrics y Plots]]. Esta es la funcionalidad más potente y menos conocida de DVC.

## El problema que resuelve

Probar 30 combinaciones de hiperparámetros usando solo `dvc repro` + commits de Git manuales implicaría 30 commits (o 30 branches) solo para experimentación exploratoria — ensucia el historial de Git con intentos fallidos que nadie necesita conservar permanentemente. **DVC Experiments** permite ejecutar y comparar decenas de variaciones sin generar ningún commit hasta que explícitamente se decide conservar una.

## `dvc exp run` — ejecutar un experimento sin comprometer nada a Git

```bash
dvc exp run
```

Ejecuta el pipeline completo (equivalente a `dvc repro`) pero guarda el resultado como un **experimento**, no como un commit de Git — el workspace vuelve a su estado original después, y el experimento queda registrado internamente para poder consultarlo o descartarlo después.

### Sobrescribir parámetros sin editar `params.yaml`

```bash
dvc exp run --set-param entrenamiento.max_depth=8 --set-param entrenamiento.learning_rate=0.03
```

Permite probar una combinación puntual de hiperparámetros directamente desde la línea de comandos, sin necesitar editar `params.yaml` y luego revertir el cambio manualmente — ideal para exploración rápida.

### Ejecutar múltiples combinaciones en cola

```bash
dvc exp run --queue --set-param entrenamiento.max_depth=4
dvc exp run --queue --set-param entrenamiento.max_depth=6
dvc exp run --queue --set-param entrenamiento.max_depth=8

dvc exp run --run-all   # ejecuta TODOS los experimentos en cola, secuencialmente (o en paralelo con -j)
dvc exp run --run-all -j 4   # hasta 4 experimentos en paralelo
```

Patrón directo para una búsqueda de hiperparámetros simple tipo grid search, sin necesitar un script externo que orqueste cada corrida — la cola de DVC lo maneja nativamente.

## `dvc exp show` — comparar todos los experimentos ejecutados

```bash
dvc exp show
```

```
Experiment          Created    mae     rmse    max_depth   learning_rate
workspace            -         12.1    17.9    6            0.05
main                 -          14.1    19.8    5            0.05
├── exp-a3f21        2h ago     12.4    18.7    6            0.05
├── exp-b7c19        2h ago     11.9    17.5    8            0.03
└── exp-d4e88        2h ago     13.2    19.1    4            0.05
```

Una tabla completa con métricas, parámetros y metadata de cada experimento — permite comparar decenas de corridas de un vistazo, ordenar por la métrica de interés, y decidir cuál merece convertirse en un commit real.

```bash
dvc exp show --sort-by mae --sort-order asc   # ordenar por la métrica que importa
dvc exp show --only-changed   # mostrar solo columnas de params/metrics que variaron entre experimentos
```

## `dvc exp diff` — comparar dos experimentos específicos en detalle

```bash
dvc exp diff exp-a3f21 exp-b7c19
```

Igual que `dvc metrics diff`/`dvc params diff` (ver [[05 - Params, Metrics y Plots]]), pero comparando experimentos en vez de commits — muestra exactamente qué cambió y qué efecto tuvo, entre dos corridas exploratorias que nunca llegaron a ser commits de Git.

## Promover un experimento a un commit real

Tras identificar el experimento ganador en `dvc exp show`, se conserva permanentemente:

```bash
dvc exp apply exp-b7c19   # aplica los cambios de ese experimento al workspace actual

git add params.yaml dvc.lock
git commit -m "Adopto max_depth=8, learning_rate=0.03 — mejor MAE en validación"
```

`dvc exp apply` es el paso que convierte un experimento "descartable" en cambios reales del workspace — solo entonces vale la pena un commit de Git, evitando el ruido de las 29 combinaciones que no se adoptaron.

## Convertir un experimento en un branch de Git (cuando se necesita seguir iterando sobre él)

```bash
dvc exp branch exp-b7c19 rama-mejor-modelo
git checkout rama-mejor-modelo
```

Útil cuando un experimento resulta prometedor pero requiere más trabajo antes de mergear a main — en vez de aplicar directamente, se convierte en un branch real de Git donde se puede seguir iterando con el flujo normal de Git.

## Limpiar experimentos descartados

```bash
dvc exp remove exp-d4e88          # elimina un experimento específico
dvc exp remove --all             # limpia TODOS los experimentos no aplicados

dvc exp gc                        # limpieza automática de experimentos huérfanos/antiguos
```

Los experimentos no aplicados no ensucian el historial de Git (nunca fueron commits), pero sí ocupan espacio en el almacenamiento interno de DVC — vale limpiarlos periódicamente tras sesiones extensas de exploración.

## Integración con Optuna — DVC como capa de reproducibilidad sobre la búsqueda

```python
# objective.py, invocado dentro de cada dvc exp run
import optuna
import subprocess
import json

def objective(trial):
    params = {
        "max_depth": trial.suggest_int("max_depth", 3, 10),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
    }
    with open("params.yaml", "w") as f:
        yaml.dump({"entrenamiento": params}, f)

    subprocess.run(["dvc", "repro"], check=True)   # ejecuta el pipeline versionado con DVC

    with open("metrics/resultados.json") as f:
        return json.load(f)["mae"]
```

En la práctica, Optuna (ver `Optuna/01 - Introducción y Conceptos Fundamentales.md`) suele preferirse para búsquedas de hiperparámetros grandes y sofisticadas (con TPE, pruning), mientras `dvc exp` brilla para exploración manual más pequeña donde la trazabilidad de datos/pipeline importa tanto como el resultado numérico — no son mutuamente excluyentes, y es válido combinar ambas: Optuna decide qué probar, DVC garantiza que cada intento sea reproducible de extremo a extremo.

## Ver también

- [[04 - DVC Pipelines - dvc.yaml y Reproducibilidad]]
- [[05 - Params, Metrics y Plots]]
- `Optuna/01 - Introducción y Conceptos Fundamentales.md`
- `MLflow/02 - Tracking - Fundamentos y API de Logging.md`
