---
tags: [machine-learning, hiperparametros, optuna, optimizacion]
---

# 45 — Optimización de Hiperparámetros: Grid Search, Random Search, Optimización Bayesiana (Optuna)

> Nota del mentor: mencionamos Optuna de pasada en la nota de librerías, pero nunca profundizamos en cómo realmente se usa — y es exactamente la herramienta que le da rigor científico a algo que muchos hacen "a mano", probando valores de `max_depth` y `learning_rate` por intuición y suerte. Esta nota cierra ese cabo suelto, conectando directamente con la validación rigurosa de la nota 37.

## 1. El problema — el espacio de hiperparámetros es enorme y probarlos a mano no escala

Para un solo `XGBRegressor`, ya tienes `n_estimators`, `max_depth`, `learning_rate`, `reg_alpha`, `reg_lambda`, `subsample`, `colsample_bytree`, `gamma` — ocho hiperparámetros, cada uno con un rango razonable de valores. Probar combinaciones a mano, aunque sea con buena intuición, deja la mayor parte del espacio de búsqueda sin explorar y es imposible de reproducir sistemáticamente entre sesiones.

## 2. Grid Search — exhaustivo, pero exponencialmente costoso

```python
from sklearn.model_selection import GridSearchCV

parametros = {
    "max_depth": [3, 5, 7],
    "learning_rate": [0.01, 0.05, 0.1],
    "n_estimators": [200, 300, 500],
}
# 3 × 3 × 3 = 27 combinaciones, cada una evaluada con cross-validation completo

grid = GridSearchCV(XGBRegressor(), parametros, cv=5, scoring="neg_mean_absolute_error")
grid.fit(X_train, y_train)
print(grid.best_params_)
```

Prueba **todas** las combinaciones posibles de la grilla — garantiza encontrar la mejor combinación **dentro de los valores que definiste**, pero el costo crece exponencialmente con cada hiperparámetro adicional (con 8 hiperparámetros y solo 3 valores cada uno, ya son 3⁸ = 6,561 combinaciones). Impracticable más allá de 3-4 hiperparámetros con pocos valores cada uno.

## 3. Random Search — sorprendentemente más eficiente de lo que parece

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import uniform, randint

distribuciones = {
    "max_depth": randint(3, 10),
    "learning_rate": uniform(0.01, 0.3),
    "n_estimators": randint(100, 600),
}

random_search = RandomizedSearchCV(
    XGBRegressor(), distribuciones, n_iter=50, cv=5, scoring="neg_mean_absolute_error"
)
random_search.fit(X_train, y_train)
```

En vez de probar cada combinación de una grilla fija, muestrea aleatoriamente `n_iter` combinaciones de distribuciones continuas. Contra la intuición, investigación empírica (Bergstra & Bengio, 2012) demostró que Random Search **encuentra soluciones comparables o mejores que Grid Search con muchas menos evaluaciones**, porque no todos los hiperparámetros importan igual — Grid Search desperdicia evaluaciones probando exhaustivamente combinaciones de hiperparámetros poco relevantes, mientras Random Search cubre más variedad del espacio realmente importante.

## 4. Optimización Bayesiana con Optuna — buscar con memoria de lo ya probado

La limitación compartida de Grid y Random Search: **cada evaluación es independiente**, ninguna aprende de las anteriores. La optimización bayesiana sí — usa los resultados de evaluaciones previas para decidir inteligentemente qué combinación probar después, concentrando la búsqueda en las regiones del espacio que parecen más prometedoras.

```python
import optuna

def objetivo(trial):
    parametros = {
        "max_depth": trial.suggest_int("max_depth", 3, 10),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
        "n_estimators": trial.suggest_int("n_estimators", 100, 600),
        "reg_alpha": trial.suggest_float("reg_alpha", 1e-3, 10.0, log=True),
        "reg_lambda": trial.suggest_float("reg_lambda", 1e-3, 10.0, log=True),
        "subsample": trial.suggest_float("subsample", 0.6, 1.0),
    }
    modelo = XGBRegressor(**parametros)

    # CRÍTICO: usa walk-forward validation, NUNCA K-Fold estándar (ver nota 37)
    resultados_walk_forward = walk_forward_validation(
        datos, modelo, ventana_entrenamiento_dias=90, ventana_validacion_dias=14
    )
    return resultados_walk_forward["mae"].mean()

estudio = optuna.create_study(direction="minimize")
estudio.optimize(objetivo, n_trials=100)

print(estudio.best_params)
print(estudio.best_value)
```

Optuna internamente usa un algoritmo llamado **TPE (Tree-structured Parzen Estimator)** por defecto: modela la distribución de probabilidad de qué combinaciones de hiperparámetros tienden a dar buenos resultados versus malos, y usa ese modelo para proponer la siguiente combinación a probar — cada trial nuevo es informado por todos los anteriores, a diferencia de Random Search donde cada muestra es ciega a las demás.

## 5. Pruning — abandonar trials poco prometedores temprano

```python
def objetivo_con_pruning(trial):
    modelo = XGBRegressor(
        max_depth=trial.suggest_int("max_depth", 3, 10),
        learning_rate=trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
        n_estimators=trial.suggest_int("n_estimators", 100, 600),
    )
    for i, (train_idx, val_idx) in enumerate(walk_forward_splits(datos)):
        modelo.fit(datos.iloc[train_idx][features], datos.iloc[train_idx][target])
        mae_fold = evaluar(modelo, datos.iloc[val_idx])

        trial.report(mae_fold, i)
        if trial.should_prune():  # Optuna decide si este trial ya no vale la pena continuar
            raise optuna.TrialPruned()

    return mae_fold

estudio = optuna.create_study(direction="minimize", pruner=optuna.pruners.MedianPruner())
```

Si un trial va claramente peor que la mediana de los trials anteriores en las primeras validaciones de walk-forward, Optuna lo abandona sin completar el resto de las particiones — ahorra tiempo de cómputo significativo al no desperdiciar recursos completando evaluaciones que ya se ven claramente inferiores.

## 6. Integración con MLflow — no pierdas el rastro de la búsqueda

```python
import mlflow

def objetivo(trial):
    with mlflow.start_run(nested=True):
        parametros = {...}
        mlflow.log_params(parametros)
        modelo = XGBRegressor(**parametros)
        mae = evaluar_walk_forward(modelo, datos)
        mlflow.log_metric("mae", mae)
    return mae

with mlflow.start_run(run_name="optuna_search_xgboost"):
    estudio = optuna.create_study(direction="minimize")
    estudio.optimize(objetivo, n_trials=100)
    mlflow.log_params(estudio.best_params)
```

Esto conecta directamente con [[15-MLflow-en-Profundidad]]: cada trial de Optuna queda registrado como un run anidado, dándote trazabilidad completa de toda la búsqueda de hiperparámetros — consultable después para entender no solo cuál fue la mejor combinación, sino cómo se comportó el desempeño a través de todo el espacio explorado.

## 7. El error más caro en optimización de hiperparámetros — optimizar contra el set equivocado

```python
# INCORRECTO: optimizar contra el mismo set de test que usarás para el reporte final
mejor_modelo = optuna_search(datos_completos)
mae_final = evaluar(mejor_modelo, datos_test)  # ¡mismo set usado implícitamente durante la búsqueda!

# CORRECTO: tres particiones separadas
# train → para ajustar el modelo en cada trial
# validation → para que Optuna decida qué hiperparámetros son mejores
# test → nunca tocado durante la búsqueda, solo al final para el reporte honesto
```

Si usas el mismo conjunto para elegir hiperparámetros y para reportar el desempeño final, estás **optimizando implícitamente contra ese conjunto** — el MAE que reportas está sesgado optimistamente, un tipo sutil de leakage muy relacionado con lo visto en [[37-Validacion-Rigurosa-en-ML]]. La disciplina correcta: separa un conjunto de test que Optuna nunca ve durante los 100 trials, y evalúa el modelo final solo una vez contra él, al terminar toda la búsqueda.

## Ver también
- Cheat-sheet técnico completo de Optuna (sintaxis, API, ejemplos): `Optuna/01 - Introducción y Conceptos Fundamentales.md`
- [[37-Validacion-Rigurosa-en-ML]]
- [[36-Ensambles-en-Profundidad]]
- [[15-MLflow-en-Profundidad]]
- [[38-Metricas-de-Regresion-Cuando-Usar-y-Cuando-No]]
