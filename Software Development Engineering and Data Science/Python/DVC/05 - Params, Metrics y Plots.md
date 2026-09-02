---
tags: [dvc, mlops, metrics, plots, cheat-sheet]
---

# 05 — Params, Metrics y Plots

> Continúa de [[04 - DVC Pipelines - dvc.yaml y Reproducibilidad]].

Una vez que un pipeline produce métricas y gráficas como parte de sus `outs` (ver `dvc.yaml`), DVC ofrece comandos dedicados para compararlas entre distintas versiones del código/datos — sin necesitar abrir cada archivo de métricas manualmente.

## Declarar métricas en el pipeline

```yaml
# dvc.yaml
stages:
  evaluar:
    cmd: python scripts/evaluar.py
    deps: [models/modelo.pkl]
    metrics:
      - metrics/resultados.json:
          cache: false   # las métricas suelen versionarse en Git directamente, no en el cache de datos
```

```python
# scripts/evaluar.py
import json

resultados = {"mae": 12.4, "rmse": 18.7, "r2": 0.89}
with open("metrics/resultados.json", "w") as f:
    json.dump(resultados, f, indent=2)
```

`cache: false` es la convención habitual para métricas: son archivos pequeños, tiene sentido que Git los versione directamente (para poder hacer `git diff` legible sobre ellos) en vez de pasar por el cache de DVC diseñado para archivos grandes.

## `dvc metrics show` — ver las métricas actuales

```bash
dvc metrics show
```

```
Path                       mae      rmse     r2
metrics/resultados.json    12.4     18.7     0.89
```

## `dvc metrics diff` — comparar métricas entre commits

```bash
dvc metrics diff HEAD~1 HEAD
```

```
Path                     Metric    HEAD~1    HEAD    Change
metrics/resultados.json  mae       14.1      12.4    -1.7
metrics/resultados.json  rmse      19.8      18.7    -1.1
metrics/resultados.json  r2        0.85      0.89    +0.04
```

Responde directamente "¿mejoró o empeoró el modelo entre estas dos versiones del código/datos?" sin tener que revisar manualmente dos archivos JSON — el `Change` calculado automáticamente hace evidente de un vistazo la dirección de la mejora.

```bash
dvc metrics diff --all   # muestra todas las métricas, no solo las que cambiaron
```

## `dvc params diff` — comparar hiperparámetros entre commits

```bash
dvc params diff HEAD~1 HEAD
```

```
Path         Param                          HEAD~1    HEAD
params.yaml  entrenamiento.max_depth        5         6
params.yaml  entrenamiento.dias_atras       60        90
```

Análogo a `dvc metrics diff`, pero para `params.yaml` — útil para correlacionar visualmente "qué hiperparámetro cambió" junto con "qué métrica cambió" entre dos versiones, sin necesitar abrir ambos archivos por separado.

## Declarar plots en el pipeline

```yaml
stages:
  evaluar:
    cmd: python scripts/evaluar.py
    plots:
      - plots/curva_residuos.csv:
          x: prediccion
          y: residuo
          title: "Residuos vs. Predicción"
```

```python
# scripts/evaluar.py — genera un CSV simple, DVC se encarga de graficarlo
import pandas as pd

df_residuos = pd.DataFrame({"prediccion": y_pred, "residuo": y_test - y_pred})
df_residuos.to_csv("plots/curva_residuos.csv", index=False)
```

DVC no requiere que el script genere una imagen — basta con datos tabulares (CSV/JSON/YAML), y DVC construye la visualización interactiva (vía Vega-Lite) a partir de esos datos.

## `dvc plots show` — generar la visualización

```bash
dvc plots show
```

Genera un archivo HTML interactivo local con la gráfica configurada — abrible directamente en el navegador, sin necesitar Jupyter ni un servidor adicional corriendo.

## `dvc plots diff` — comparar curvas entre versiones, superpuestas

```bash
dvc plots diff HEAD~1 HEAD
```

Superpone en la misma gráfica los datos de ambas versiones (por ejemplo, la curva de residuos del modelo actual vs. el modelo anterior) — mucho más informativo que comparar dos números sueltos cuando el comportamiento del modelo importa a lo largo de un rango (residuos por rango de predicción, curva ROC completa, importancia de features).

## Confusion matrix como plot — patrón común en clasificación

```yaml
plots:
  - plots/confusion_matrix.csv:
      template: confusion
      x: predicted
      y: actual
```

```python
import pandas as pd
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, y_pred)
# DVC espera formato "long" (una fila por celda de la matriz), no la matriz 2D directamente
filas = [{"actual": i, "predicted": j, "value": cm[i, j]} for i in range(len(cm)) for j in range(len(cm))]
pd.DataFrame(filas).to_csv("plots/confusion_matrix.csv", index=False)
```

DVC incluye plantillas (`template:`) predefinidas para visualizaciones comunes de ML (matriz de confusión, curva de calibración) — evita tener que configurar manualmente cada eje/tipo de gráfica desde cero.

## Comparar todo a la vez — el flujo típico de revisión de un experimento

```bash
dvc metrics diff HEAD~1 HEAD
dvc params diff HEAD~1 HEAD
dvc plots diff HEAD~1 HEAD
```

Este trío de comandos es la forma estándar de responder, tras cualquier cambio de código/datos/hiperparámetros: "¿qué cambió, y el modelo mejoró o empeoró como resultado?" — completamente desde la terminal, sin necesitar una UI externa para el caso de comparación simple entre dos commits (para exploración más rica de decenas de experimentos, ver [[06 - DVC Experiments - Iteración Rápida sin Comprometer Git]]).

## Ver también

- [[04 - DVC Pipelines - dvc.yaml y Reproducibilidad]]
- [[06 - DVC Experiments - Iteración Rápida sin Comprometer Git]]
- `MLflow/04 - Tracking - Búsqueda, Comparación y Organización.md` (comparación equivalente en MLflow)
