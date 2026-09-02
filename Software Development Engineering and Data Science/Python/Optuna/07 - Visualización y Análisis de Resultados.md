---
tags: [optuna, hiperparametros, visualizacion, cheat-sheet]
---

# 07 — Visualización y Análisis de Resultados

> Se apoya en [[Python/Optuna/01 - Introducción y Conceptos Fundamentales]] y [[06 - Optimización Multi-Objetivo]].

Optuna incluye un módulo de visualización (`optuna.visualization`, basado en Plotly) que convierte el historial de trials en gráficas interactivas — esencial para entender *por qué* una combinación de hiperparámetros funcionó mejor que otra, no solo *cuál* fue la mejor.

```bash
pip install plotly   # dependencia de optuna.visualization
```

## `study.trials_dataframe()` — análisis tabular con pandas

La forma más flexible de analizar resultados, cuando las gráficas predefinidas no alcanzan:

```python
df = study.trials_dataframe()
print(df.columns.tolist())
# ['number', 'value', 'datetime_start', 'datetime_complete', 'duration',
#  'params_n_estimators', 'params_max_depth', 'params_learning_rate', 'state']

# Análisis custom con pandas:
df_completos = df[df["state"] == "COMPLETE"]
correlacion = df_completos[["value", "params_learning_rate", "params_max_depth"]].corr()
print(correlacion)

top10 = df_completos.nsmallest(10, "value")   # los 10 mejores trials (si direction="minimize")
```

## `plot_optimization_history` — progreso de la búsqueda

```python
import optuna.visualization as vis

fig = vis.plot_optimization_history(study)
fig.show()
```

Muestra el valor objetivo de cada trial a lo largo del tiempo, junto con la línea del mejor valor acumulado hasta el momento. Permite ver de un vistazo si la búsqueda ya convergió (la línea del mejor valor se aplana) o si más trials probablemente seguirían mejorando el resultado.

## `plot_param_importances` — qué hiperparámetros importan realmente

```python
fig = vis.plot_param_importances(study)
fig.show()
```

Calcula la importancia relativa de cada hiperparámetro sobre el valor objetivo (por defecto usando un modelo de fANOVA internamente). Es habitual descubrir que 2-3 hiperparámetros explican la mayor parte de la varianza en el resultado, mientras otros apenas importan — información valiosa para simplificar el espacio de búsqueda en futuras iteraciones (dejar fijos los que no importan, concentrar el presupuesto de trials en los que sí).

## `plot_slice` — efecto individual de cada hiperparámetro

```python
fig = vis.plot_slice(study, params=["learning_rate", "max_depth"])
fig.show()
```

Un scatter plot por hiperparámetro, mostrando el valor objetivo obtenido para cada valor probado de ese parámetro específico — útil para detectar visualmente el rango donde se concentran los mejores resultados de cada hiperparámetro individual.

## `plot_contour` — interacción entre dos hiperparámetros

```python
fig = vis.plot_contour(study, params=["learning_rate", "n_estimators"])
fig.show()
```

Gráfica de contorno 2D mostrando cómo el valor objetivo varía según la combinación conjunta de dos hiperparámetros — revela interacciones que `plot_slice` (que mira cada parámetro por separado) no puede mostrar, por ejemplo, que un `learning_rate` alto solo funciona bien combinado con un `n_estimators` bajo.

## `plot_parallel_coordinate` — vista multidimensional completa

```python
fig = vis.plot_parallel_coordinate(study, params=["learning_rate", "max_depth", "n_estimators"])
fig.show()
```

Cada trial es una línea que atraviesa un eje vertical por hiperparámetro, coloreada según el valor objetivo. Permite ver patrones a través de **más de dos dimensiones simultáneamente** — la vista más densa de información entre todas las gráficas de Optuna, aunque requiere algo de práctica para leerla con soltura.

## `plot_edf` — función de distribución empírica

```python
fig = vis.plot_edf(study)
fig.show()
```

Muestra qué fracción de los trials logró un valor objetivo igual o mejor que cada punto del eje X — útil para comparar la "robustez" de distintos estudios (ej. TPE vs. Random Search): un estudio cuya curva EDF sube más rápido encontró buenas soluciones con menos trials.

## `plot_intermediate_values` — trayectorias de pruning

```python
fig = vis.plot_intermediate_values(study)
fig.show()
```

Solo relevante si se usó `trial.report()` (ver [[04 - Pruners en Profundidad]]). Muestra la curva de aprendizaje (valor intermedio vs. step) de cada trial, y visualmente resalta cuáles fueron podados y en qué punto — ayuda a validar que el pruner está tomando decisiones sensatas (podar trials que efectivamente iban mal, no trials que solo eran lentos en converger).

## `plot_pareto_front` — frontera de Pareto (multi-objetivo)

```python
fig = vis.plot_pareto_front(study, target_names=["MAE", "Latencia (ms)"])
fig.show()
```

Cubierto en detalle en [[06 - Optimización Multi-Objetivo]].

## Optuna Dashboard — UI web en tiempo real

Para monitorear una búsqueda larga sin escribir código de visualización cada vez:

```bash
pip install optuna-dashboard

optuna-dashboard sqlite:///optuna_studies.db
# o apuntando a Postgres:
optuna-dashboard postgresql://usuario:password@host:5432/optuna_db
```

Levanta una interfaz web (por defecto en `http://127.0.0.1:8080`) con todas las gráficas anteriores generadas automáticamente y actualizadas en vivo mientras el estudio sigue corriendo — el equivalente de Optuna a la UI de MLflow, pero enfocado específicamente en el proceso de búsqueda de hiperparámetros en sí (ver también `MLflow/04 - Tracking - Búsqueda, Comparación y Organización.md` para la UI de comparación de runs de MLflow).

## Exportar gráficas para reportes

```python
fig = vis.plot_optimization_history(study)
fig.write_image("historia_optimizacion.png")   # requiere `pip install kaleido`
fig.write_html("historia_optimizacion.html")    # interactiva, sin dependencias extra
```

## Ver también

- [[Python/Optuna/01 - Introducción y Conceptos Fundamentales]]
- [[04 - Pruners en Profundidad]]
- [[06 - Optimización Multi-Objetivo]]
- [[08 - Integraciones con Frameworks de ML]]
