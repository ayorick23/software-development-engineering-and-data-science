---
tags: [optuna, hiperparametros, buenas-practicas, comparativa, cheat-sheet]
---

# 11 — Buenas Prácticas, Errores Comunes y Comparativa

> Cierre del cheat-sheet. Se apoya en todos los archivos anteriores, especialmente [[03 - Samplers en Profundidad]] y [[04 - Pruners en Profundidad]].

## El error más caro: optimizar contra el set equivocado

Ya cubierto en detalle en [[45-Optimizacion-de-Hiperparametros]], pero vale repetirlo porque es el error más frecuente y más costoso:

```python
# INCORRECTO: el mismo conjunto usado para elegir hiperparámetros se usa para el reporte final
mejor_modelo = optuna_search(datos_completos)
mae_final = evaluar(mejor_modelo, datos_test)   # ¡el resultado está sesgado optimistamente!

# CORRECTO: tres particiones separadas — train / validation / test
# Optuna solo ve train y validation durante los 100 trials.
# test se evalúa UNA SOLA VEZ, al final, con los hiperparámetros ya decididos.
```

Optimizar contra el mismo conjunto que luego se reporta como "desempeño final" es una forma sutil de *data leakage* — el número que se reporta ya no es una estimación honesta del desempeño en datos nuevos, porque los hiperparámetros fueron elegidos específicamente para funcionar bien en ese conjunto.

## No usar `log=True` donde corresponde

```python
# SUB-ÓPTIMO: explora linealmente, sobre-muestrea valores grandes de learning_rate
lr = trial.suggest_float("learning_rate", 0.001, 0.3)

# CORRECTO: explora proporcionalmente en cada orden de magnitud
lr = trial.suggest_float("learning_rate", 0.001, 0.3, log=True)
```

Sin `log=True`, un rango como `[0.001, 0.3]` dedica la mayoría de los trials a valores entre 0.15 y 0.3, cuando en la práctica la región interesante para `learning_rate` suele estar en 0.001–0.05. Ver [[02 - Definición del Espacio de Búsqueda]].

## Espacio de búsqueda demasiado amplio "por si acaso"

```python
# Ineficiente: rangos absurdamente amplios diluyen el presupuesto de trials
n_estimators = trial.suggest_int("n_estimators", 1, 10000)

# Mejor: acotar con conocimiento del dominio (documentación del algoritmo, literatura, experiencia previa)
n_estimators = trial.suggest_int("n_estimators", 100, 800)
```

Un espacio de búsqueda demasiado amplio no es "más seguro" — obliga al sampler a gastar más trials solo para descubrir en qué región concentrarse, dejando menos presupuesto para refinar dentro de esa región. Acotar el espacio con criterio (basado en la teoría del algoritmo o en búsquedas previas) suele encontrar mejores resultados con el mismo número de trials.

## No aprovechar pruning en entrenamientos largos

Si cada trial toma minutos u horas (deep learning, boosting con miles de árboles) y no se usa pruning, se desperdicia tiempo de cómputo completando trials que ya eran claramente malos desde temprano. Ver [[04 - Pruners en Profundidad]] — casi siempre vale la pena instrumentar `trial.report()`/`should_prune()` cuando hay una noción natural de progreso incremental (épocas, rondas de boosting, folds).

## Registrar cada trial en el Model Registry de MLflow

```python
# INCORRECTO: satura el Registry con decenas/cientos de versiones de prueba
def objective(trial):
    modelo = entrenar(trial)
    mlflow.sklearn.log_model(modelo, "model", registered_model_name="mi-modelo")  # ¡NO en cada trial!
    return evaluar(modelo)

# CORRECTO: solo registrar el modelo ganador, después de terminar la búsqueda
study.optimize(objective, n_trials=100)
modelo_final = reentrenar_con_mejores_params(study.best_params)
with mlflow.start_run():
    mlflow.sklearn.log_model(modelo_final, "model", registered_model_name="mi-modelo")
```

Ver `MLflow/15 - Buenas Prácticas, Seguridad y Comparativa.md`.

## No fijar `seed` cuando se necesita reproducibilidad

```python
# Sin seed: cada corrida explora un orden distinto de trials, resultados no exactamente reproducibles
sampler = optuna.samplers.TPESampler()

# Con seed: mismo orden de exploración entre corridas (útil en tests automatizados del pipeline)
sampler = optuna.samplers.TPESampler(seed=42)
```

## Confundir `n_trials` con "presupuesto de cómputo real"

Cien trials de un modelo que tarda 1 segundo en entrenar es un problema completamente distinto a cien trials de un modelo que tarda 10 minutos. Antes de lanzar una búsqueda larga, conviene estimar el tiempo total (`n_trials × tiempo_promedio_por_trial`) y decidir si `timeout` es un límite más apropiado que `n_trials` fijo — especialmente en pipelines de CI/CD con ventanas de tiempo acotadas.

## Comparativa: Optuna vs. otras librerías de tuning

| Librería | Fortaleza relativa | Cuándo preferirla sobre Optuna |
|---|---|---|
| **Hyperopt** | Pionera en TPE, API más simple (`fmin`) | Proyectos legacy ya construidos sobre Hyperopt; para proyectos nuevos, Optuna generalmente ofrece mejor ergonomía (define-by-run, pruning nativo, visualización) |
| **Ray Tune** | Escalado masivo en clusters, integra múltiples algoritmos (incluido Optuna como backend) | Búsquedas a gran escala en un cluster de Ray ya existente, con necesidad de orquestar cientos de workers |
| **scikit-optimize (skopt)** | Optimización bayesiana con procesos gaussianos, buena para espacios muy continuos de baja dimensión | Espacios de búsqueda pequeños (pocos hiperparámetros) donde interpretabilidad del modelo bayesiano importa más que velocidad |
| **GridSearchCV / RandomizedSearchCV (sklearn)** | Cero dependencias externas, ya integrado en el ecosistema sklearn | Espacios muy pequeños y baratos de evaluar exhaustivamente, o cuando agregar una dependencia nueva no se justifica |
| **Keras Tuner** | Integración nativa con el ecosistema Keras/TensorFlow | Proyectos 100% Keras donde la integración out-of-the-box importa más que flexibilidad multi-framework |

**Por qué Optuna suele ganar como default moderno**: define-by-run soporta espacios condicionales de forma natural (algo que Grid/Random Search y muchas alternativas bayesianas clásicas manejan mal), el pruning está integrado de forma nativa y simple, la visualización es rica sin configuración adicional, y es agnóstico de framework (funciona igual con sklearn, XGBoost, PyTorch, TensorFlow) — no ata el proyecto a un ecosistema específico como sí lo hace Keras Tuner o Ray Tune con Ray.

## Checklist antes de lanzar una búsqueda larga en producción

1. ¿El espacio de búsqueda usa `log=True` donde corresponde (learning rates, regularización)?
2. ¿Los datos de validación usados dentro de `objective` están completamente separados del set de test final?
3. Si el entrenamiento es largo, ¿está instrumentado el pruning (`trial.report`/`should_prune`)?
4. ¿El estudio usa `storage` persistente, no solo memoria, para sobrevivir a un crash?
5. ¿Se fijó `seed` si la búsqueda necesita ser reproducible?
6. ¿Solo el modelo ganador final se registra en el Model Registry, no cada trial?
7. ¿`n_trials`/`timeout` están calibrados contra el tiempo real disponible (ventana de CI/CD, presupuesto de cómputo)?

## Ver también

- [[01 - Introducción y Conceptos Fundamentales]]
- [[04 - Pruners en Profundidad]]
- `Machine Learning/45-Optimizacion-de-Hiperparametros.md`
- `Machine Learning/37-Validacion-Rigurosa-en-ML.md`
- `MLflow/15 - Buenas Prácticas, Seguridad y Comparativa.md`
