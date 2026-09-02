---
tags: [optuna, hiperparametros, pruning, cheat-sheet]
---

# 04 — Pruners en Profundidad

> Continúa de [[03 - Samplers en Profundidad]].

**Pruning** (poda) es la capacidad de Optuna de **abandonar un trial antes de que termine**, si sus resultados intermedios ya muestran que va peor que trials anteriores. Es especialmente valioso en entrenamientos largos (redes neuronales, boosting con muchos árboles) donde completar un trial mediocre hasta el final desperdicia tiempo de cómputo que podría usarse probando otras combinaciones.

## El mecanismo: `trial.report()` + `trial.should_prune()`

Pruning requiere que la función objetivo reporte resultados **intermedios**, no solo el resultado final:

```python
def objective(trial):
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 1000),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
    }
    model = XGBRegressor(**params, n_estimators=1)  # se construye incrementalmente

    for step in range(100):   # ej. 100 rondas de boosting, evaluadas una a una
        model.n_estimators = step + 1
        model.fit(X_train, y_train, xgb_model=model.get_booster() if step > 0 else None)
        mae_intermedio = mean_absolute_error(y_val, model.predict(X_val))

        trial.report(mae_intermedio, step)   # informa el resultado parcial a Optuna

        if trial.should_prune():             # Optuna decide si vale la pena seguir
            raise optuna.TrialPruned()        # abandona el trial limpiamente

    return mae_intermedio
```

- `trial.report(value, step)`: registra el valor de la métrica en el paso `step` (época, ronda de boosting, fold de cross-validation).
- `trial.should_prune()`: consulta al *pruner* configurado si, dado el historial de otros trials en ese mismo `step`, este trial ya se ve suficientemente mal como para abandonarlo.
- `raise optuna.TrialPruned()`: es la forma correcta de terminar el trial anticipadamente — Optuna lo registra con estado `PRUNED` (distinto de `FAIL`, que indica un error real).

## `MedianPruner` — el más usado

```python
from optuna.pruners import MedianPruner

pruner = MedianPruner(
    n_startup_trials=5,    # los primeros 5 trials nunca se podan (se necesita historial primero)
    n_warmup_steps=10,     # dentro de cada trial, los primeros 10 steps nunca se podan
    interval_steps=1,      # cada cuántos steps se evalúa la condición de poda
)
study = optuna.create_study(direction="minimize", pruner=pruner)
```

Poda un trial si, en el `step` actual, su valor intermedio es peor que la **mediana** de los valores de otros trials en ese mismo step. Simple, robusto, buen default.

## `PercentilePruner` — más agresivo o más conservador que la mediana

```python
from optuna.pruners import PercentilePruner

pruner = PercentilePruner(
    percentile=25.0,   # poda si el trial está peor que el percentil 25 (más agresivo que la mediana=50)
    n_startup_trials=5,
    n_warmup_steps=10,
)
```

Con `percentile=25.0`, solo sobrevive el 25% superior de los trials en cada checkpoint — poda mucho más agresivamente que `MedianPruner` (equivalente a `percentile=50.0`).

## `SuccessiveHalvingPruner` — asignación adaptativa de recursos

```python
from optuna.pruners import SuccessiveHalvingPruner

pruner = SuccessiveHalvingPruner(
    min_resource=1,        # mínimo de recursos (steps) antes de considerar podar
    reduction_factor=4,    # en cada ronda, sobrevive 1/reduction_factor de los trials
    min_early_stopping_rate=0,
)
```

Basado en el algoritmo **Successive Halving**: empieza evaluando muchos trials con pocos recursos (steps), y en cada "ronda" descarta la mayoría, dando más recursos solo a los sobrevivientes. Eficiente cuando evaluar con pocos recursos ya es informativo sobre el desempeño final relativo.

## `HyperbandPruner` — Successive Halving sin tener que fijar el trade-off de antemano

```python
from optuna.pruners import HyperbandPruner

pruner = HyperbandPruner(
    min_resource=1,
    max_resource=100,       # ej. máximo de épocas/rondas de boosting
    reduction_factor=3,
)
study = optuna.create_study(direction="minimize", pruner=pruner, sampler=optuna.samplers.TPESampler())
```

Ejecuta múltiples configuraciones de Successive Halving en paralelo (distintos "brackets"), cada uno con un balance distinto entre número de trials y recursos por trial — evita tener que adivinar manualmente el mejor `reduction_factor`/`min_resource`. Es el pruner recomendado para entrenamientos largos tipo deep learning con muchas épocas.

## `ThresholdPruner` — podar por un umbral absoluto

```python
from optuna.pruners import ThresholdPruner

pruner = ThresholdPruner(upper=50.0)   # poda si el valor intermedio supera 50.0 (ej. MAE disparatado)
# también acepta `lower=` para el caso de maximización
```

Útil para descartar rápidamente trials que divergen claramente (ej. `loss` que se va a `NaN` o valores absurdamente altos), sin necesidad de comparar contra otros trials.

## `PatientPruner` — envoltorio de tolerancia (early stopping clásico)

```python
from optuna.pruners import PatientPruner, MedianPruner

pruner = PatientPruner(
    MedianPruner(),   # el pruner base cuya decisión se retrasa
    patience=5,        # espera 5 steps consecutivos de empeoramiento antes de podar
)
```

Envuelve a otro pruner y añade una lógica de "paciencia" similar al `EarlyStopping` clásico de Keras: no poda ante el primer step malo, espera confirmación de que la tendencia es consistente.

## `NopPruner` — desactivar pruning explícitamente

```python
from optuna.pruners import NopPruner
study = optuna.create_study(pruner=NopPruner())   # equivalente a no pasar pruner
```

## Interacción entre pruning y cross-validation / walk-forward

Cuando cada trial evalúa varios folds (cross-validation o walk-forward validation), lo habitual es reportar el resultado **acumulado por fold**, no por época dentro de un fold:

```python
def objective(trial):
    params = sugerir_parametros(trial)
    resultados = []

    for fold_idx, (train_idx, val_idx) in enumerate(walk_forward_splits(datos)):
        modelo = entrenar(params, datos.iloc[train_idx])
        mae_fold = evaluar(modelo, datos.iloc[val_idx])
        resultados.append(mae_fold)

        trial.report(sum(resultados) / len(resultados), fold_idx)   # promedio acumulado hasta este fold
        if trial.should_prune():
            raise optuna.TrialPruned()

    return sum(resultados) / len(resultados)
```

Esto conecta directamente con la disciplina de validación temporal vista en `Machine Learning/37-Validacion-Rigurosa-en-ML.md`: el pruning se aplica **entre folds de walk-forward**, no dentro de un solo fold, para no romper la naturaleza secuencial de los datos.

## Ver también

- [[03 - Samplers en Profundidad]]
- [[09 - Callbacks, Constraints y Configuración Avanzada]]
- `Machine Learning/45-Optimizacion-de-Hiperparametros.md`
- `Machine Learning/37-Validacion-Rigurosa-en-ML.md`
