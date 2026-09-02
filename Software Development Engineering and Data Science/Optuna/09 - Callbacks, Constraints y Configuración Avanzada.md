---
tags: [optuna, hiperparametros, callbacks, constraints, cheat-sheet]
---

# 09 — Callbacks, Constraints y Configuración Avanzada

> Se apoya en [[01 - Introducción y Conceptos Fundamentales]] y [[02 - Definición del Espacio de Búsqueda]].

Este archivo cubre mecanismos de control fino sobre el proceso de optimización: callbacks a nivel de estudio, restricciones sobre el espacio válido, la interfaz de bajo nivel `ask`/`tell`, y manejo robusto de errores.

## Callbacks de estudio

Se ejecutan después de **cada trial completado** — útiles para logging custom, alertas, o detener la búsqueda anticipadamente bajo una condición propia.

```python
def callback_detener_si_converge(study, trial):
    if study.best_value is not None and study.best_value < 10.0:
        print(f"Objetivo alcanzado en el trial {trial.number}, deteniendo búsqueda.")
        study.stop()

study.optimize(objective, n_trials=500, callbacks=[callback_detener_si_converge])
```

```python
def callback_logging_custom(study, trial):
    print(f"Trial {trial.number} completado: valor={trial.value:.4f}, "
          f"mejor hasta ahora={study.best_value:.4f}")

study.optimize(objective, n_trials=100, callbacks=[callback_logging_custom])
```

Se pueden pasar múltiples callbacks a la vez — se ejecutan en orden después de cada trial.

## `study.stop()` — detención condicional desde dentro de un callback o la propia función objetivo

```python
def objective(trial):
    resultado = entrenar_y_evaluar(trial)
    if resultado < UMBRAL_ACEPTABLE:
        trial.study.stop()   # detiene el estudio después de este trial
    return resultado
```

Distinto de `raise optuna.TrialPruned()` (que abandona solo el trial actual) — `study.stop()` detiene toda la búsqueda, útil cuando ya se alcanzó un resultado suficientemente bueno y seguir buscando no justifica el costo de cómputo.

## Manejo de excepciones — `catch`

Por defecto, si la función objetivo lanza una excepción no capturada, **todo el estudio se detiene**. En espacios de búsqueda condicionales, es común que ciertas combinaciones sean inválidas (ej. una configuración de red neuronal que causa error de memoria) — para que esos casos se registren como `FAIL` sin abortar la búsqueda completa:

```python
study.optimize(
    objective,
    n_trials=100,
    catch=(RuntimeError, ValueError),   # estas excepciones se capturan; el trial se marca FAIL y continúa
)
```

Sin `catch`, cualquier excepción no relacionada con `optuna.TrialPruned` termina abruptamente `study.optimize()` completo, perdiendo los trials restantes planeados.

## Restricciones (Constraints) — descartar regiones inválidas del espacio

Cuando ciertas combinaciones de hiperparámetros son técnicamente válidas de generar pero no deseables (ej. un modelo que cumple la métrica de error pero excede un presupuesto de memoria), se pueden declarar restricciones explícitas en vez de simplemente retornar un valor malo:

```python
def objective(trial):
    params = sugerir_parametros(trial)
    modelo = entrenar(params)

    mae = evaluar_error(modelo)
    memoria_mb = medir_memoria(modelo)

    # Restricción: memoria_mb - 500 <= 0 significa que la restricción SE CUMPLE
    trial.set_user_attr("constraint", (memoria_mb - 500,))
    return mae

def constraints_func(trial):
    return trial.user_attrs["constraint"]   # tupla de valores; <= 0 significa válido

sampler = optuna.samplers.TPESampler(constraints_func=constraints_func)
study = optuna.create_study(direction="minimize", sampler=sampler)
study.optimize(objective, n_trials=200)
```

El sampler usa esta información para **guiar la búsqueda lejos de regiones inválidas**, en vez de simplemente descartar el resultado después de calcularlo — más eficiente que penalizar manualmente la métrica retornada.

## `set_user_attr` / `set_system_attr` — metadata adicional por trial

```python
def objective(trial):
    params = sugerir_parametros(trial)
    modelo = entrenar(params)

    trial.set_user_attr("tiempo_entrenamiento_seg", tiempo_medido)
    trial.set_user_attr("n_features_usadas", X_train.shape[1])

    return evaluar(modelo)

# Consultar después:
for t in study.trials:
    print(t.number, t.user_attrs.get("tiempo_entrenamiento_seg"))
```

Útil para guardar información de diagnóstico que no es el objetivo en sí, pero ayuda a entender el comportamiento de la búsqueda a posteriori (por ejemplo, correlacionar tiempo de entrenamiento con hiperparámetros para detectar configuraciones impracticablemente lentas).

## La interfaz `ask` / `tell` — control manual del loop de optimización

Cuando `study.optimize()` no da suficiente control (por ejemplo, se necesita integrar Optuna dentro de un framework de entrenamiento externo que ya tiene su propio loop):

```python
study = optuna.create_study(direction="minimize")

for _ in range(100):
    trial = study.ask()   # obtiene un nuevo trial SIN ejecutar ninguna función objetivo

    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 500),
        "max_depth": trial.suggest_int("max_depth", 3, 15),
    }

    modelo = entrenar(params)          # el usuario controla completamente la ejecución
    resultado = evaluar(modelo)

    study.tell(trial, resultado)        # informa el resultado de vuelta al estudio
```

Este patrón es la base de integraciones custom con orquestadores externos (Ray Tune, sistemas de colas de trabajo distribuido) donde Optuna solo decide *qué probar*, pero no controla *cómo* ni *cuándo* se ejecuta la evaluación.

### `tell` con estado explícito (para trials fallidos o podados manualmente)

```python
study.tell(trial, state=optuna.trial.TrialState.FAIL)      # marcar como fallido sin valor
study.tell(trial, state=optuna.trial.TrialState.PRUNED)    # marcar como podado manualmente
```

## `enqueue_trial` — inyectar combinaciones específicas conocidas

Ya mencionado en [[02 - Definición del Espacio de Búsqueda]] — vale profundizar el caso de uso de "sembrar" la búsqueda con configuraciones conocidas de antemano (ej. la configuración actual en producción, como punto de comparación garantizado dentro del mismo estudio):

```python
study = optuna.create_study(direction="minimize")

# Sembrar con la configuración actual de producción, para comparar directamente:
study.enqueue_trial({"n_estimators": 300, "max_depth": 6, "learning_rate": 0.05})

study.optimize(objective, n_trials=100)   # el primer trial usa esos valores; el resto, el sampler decide
```

## Logging interno de Optuna — silenciar o aumentar verbosidad

```python
import optuna

optuna.logging.set_verbosity(optuna.logging.WARNING)   # suprime el log de cada trial completado
optuna.logging.set_verbosity(optuna.logging.DEBUG)      # máximo detalle, útil para depurar
```

Por defecto Optuna imprime una línea por cada trial completado (`Trial N finished with value...`) — en búsquedas de cientos de trials dentro de un pipeline automatizado, es común bajar la verbosidad para no saturar los logs del pipeline.

## Ver también

- [[01 - Introducción y Conceptos Fundamentales]]
- [[02 - Definición del Espacio de Búsqueda]]
- [[04 - Pruners en Profundidad]]
- [[11 - Buenas Prácticas, Errores Comunes y Comparativa]]
