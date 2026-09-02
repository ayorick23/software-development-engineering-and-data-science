---
tags: [optuna, hiperparametros, cheat-sheet]
---

# 02 — Definición del Espacio de Búsqueda

> Continúa de [[Python/Optuna/01 - Introducción y Conceptos Fundamentales]].

Optuna usa el enfoque **define-by-run**: el espacio de búsqueda se declara *dentro* de la función objetivo, llamando métodos `trial.suggest_*()`. Esto es distinto a `GridSearchCV`/`RandomizedSearchCV` de sklearn, donde el espacio se declara *antes*, como un diccionario estático fuera de la función.

## Los métodos `suggest_*`

### `suggest_int` — enteros

```python
def objective(trial):
    n_estimators = trial.suggest_int("n_estimators", 100, 600)
    max_depth = trial.suggest_int("max_depth", 3, 15)

    # Con step (solo múltiplos de 50 entre 100 y 600):
    n_estimators = trial.suggest_int("n_estimators", 100, 600, step=50)

    # En escala logarítmica (útil cuando el rango abarca varios órdenes de magnitud):
    batch_size = trial.suggest_int("batch_size", 16, 512, log=True)
```

### `suggest_float` — números reales

```python
def objective(trial):
    learning_rate = trial.suggest_float("learning_rate", 0.001, 0.3)

    # Escala logarítmica — CRÍTICO para learning_rate, regularización, etc.
    # donde diferencias entre 0.001 y 0.01 importan tanto como entre 0.1 y 1.0
    learning_rate = trial.suggest_float("learning_rate", 1e-4, 1e-1, log=True)

    # Con step (grid discreto dentro de un rango continuo):
    subsample = trial.suggest_float("subsample", 0.5, 1.0, step=0.1)
```

> **Regla práctica**: usa `log=True` para cualquier hiperparámetro donde el efecto es multiplicativo, no aditivo — `learning_rate`, `reg_alpha`, `reg_lambda`, tasas de regularización en general. Sin `log=True`, Optuna muestrea uniformemente en el rango lineal, lo que sobre-explora valores grandes y sub-explora los pequeños, justo donde suele estar el óptimo en estos hiperparámetros.

### `suggest_categorical` — valores discretos no numéricos

```python
def objective(trial):
    optimizer_name = trial.suggest_categorical("optimizer", ["adam", "sgd", "rmsprop"])
    booster = trial.suggest_categorical("booster", ["gbtree", "dart", "gblinear"])

    # También funciona con booleanos:
    use_scaling = trial.suggest_categorical("use_scaling", [True, False])
```

### Métodos deprecados (aparecen en código legacy)

```python
# suggest_uniform, suggest_loguniform, suggest_discrete_uniform están DEPRECADOS
# desde Optuna 3.0 — usar suggest_float con log=True/step en su lugar:
trial.suggest_uniform("x", 0, 1)          # DEPRECADO → trial.suggest_float("x", 0, 1)
trial.suggest_loguniform("x", 1e-4, 1e-1) # DEPRECADO → trial.suggest_float("x", 1e-4, 1e-1, log=True)
```

## Espacios de búsqueda condicionales (dinámicos)

La ventaja central de define-by-run: un hiperparámetro puede depender del valor de otro, algo que un diccionario estático de `GridSearchCV` no puede expresar de forma natural.

```python
def objective(trial):
    classifier_name = trial.suggest_categorical("classifier", ["RandomForest", "SVC"])

    if classifier_name == "RandomForest":
        n_estimators = trial.suggest_int("rf_n_estimators", 100, 500)
        max_depth = trial.suggest_int("rf_max_depth", 3, 20)
        model = RandomForestClassifier(n_estimators=n_estimators, max_depth=max_depth)
    else:
        C = trial.suggest_float("svc_c", 1e-3, 1e3, log=True)
        kernel = trial.suggest_categorical("svc_kernel", ["linear", "rbf"])
        model = SVC(C=C, kernel=kernel)

    return evaluar(model)
```

Optuna solo registra los parámetros efectivamente sugeridos en cada trial — si un trial usó `RandomForest`, ese trial no tendrá valores para `svc_c`/`svc_kernel` en su registro, y viceversa. Esto es fundamental para que la búsqueda de arquitecturas (no solo hiperparámetros de una arquitectura fija) funcione correctamente.

## Espacios de búsqueda dinámicos por número de capas (deep learning)

```python
def objective(trial):
    n_layers = trial.suggest_int("n_layers", 1, 5)

    layers = []
    for i in range(n_layers):
        n_units = trial.suggest_int(f"n_units_l{i}", 16, 256, log=True)
        dropout = trial.suggest_float(f"dropout_l{i}", 0.0, 0.5)
        layers.append((n_units, dropout))

    model = construir_red(layers)
    return entrenar_y_evaluar(model)
```

Cada capa genera nombres de parámetros únicos (`n_units_l0`, `n_units_l1`, ...), lo que permite que el espacio de búsqueda tenga dimensionalidad variable entre trials — imposible de expresar con una grilla fija.

## Distribuciones — la representación interna

Cada llamada a `suggest_*` internamente crea un objeto `Distribution` (`IntDistribution`, `FloatDistribution`, `CategoricalDistribution`). Rara vez se instancian a mano, pero son accesibles para inspección:

```python
print(trial.distributions)
# {'n_estimators': IntDistribution(high=600, log=False, low=100, step=1), ...}
```

## Reutilizar un valor sugerido dentro del mismo trial

Si se llama `trial.suggest_*` dos veces con el **mismo nombre** y la **misma distribución**, Optuna retorna el mismo valor ya sugerido (no vuelve a muestrear) — útil quando la lógica de la función objetivo referencia el mismo hiperparámetro en más de un punto:

```python
def objective(trial):
    lr = trial.suggest_float("lr", 1e-4, 1e-1, log=True)
    print(f"Entrenando con lr={lr}")
    # ... más adelante en la misma función ...
    lr_again = trial.suggest_float("lr", 1e-4, 1e-1, log=True)  # devuelve el MISMO valor, no re-muestrea
    assert lr == lr_again
```

## Fijar valores durante debugging — `enqueue_trial`

Para forzar que el primer trial (o varios) use valores específicos conocidos, en vez de dejar que el sampler elija libremente desde el inicio — útil para verificar que el pipeline funciona antes de dejar correr la búsqueda completa:

```python
study = optuna.create_study(direction="minimize")
study.enqueue_trial({"n_estimators": 300, "max_depth": 6, "learning_rate": 0.05})
study.optimize(objective, n_trials=100)   # el primer trial usa esos valores exactos
```

Se profundiza en [[09 - Callbacks, Constraints y Configuración Avanzada]].

## Ver también

- [[Python/Optuna/01 - Introducción y Conceptos Fundamentales]]
- [[03 - Samplers en Profundidad]]
- [[09 - Callbacks, Constraints y Configuración Avanzada]]
