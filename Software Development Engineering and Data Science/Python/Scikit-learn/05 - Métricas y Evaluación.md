---
tags: [scikit-learn, machine-learning, metricas, evaluacion, cheat-sheet]
---

# 05 — Métricas y Evaluación

> Continúa de [[04 - Model Selection - Validación y Búsqueda]]. Para el criterio de cuándo usar cada métrica de regresión, ver `Machine Learning/38-Metricas-de-Regresion-Cuando-Usar-y-Cuando-No.md`.

El módulo `sklearn.metrics` contiene todas las funciones de evaluación. Este archivo cubre la sintaxis; el criterio de negocio para elegir métrica está en las notas de `Machine Learning/`.

## Métricas de regresión

```python
from sklearn.metrics import (
    mean_absolute_error, mean_squared_error, root_mean_squared_error,
    r2_score, mean_absolute_percentage_error, median_absolute_error,
)

mae = mean_absolute_error(y_test, y_pred)
rmse = root_mean_squared_error(y_test, y_pred)   # scikit-learn >= 1.4; antes: mean_squared_error(squared=False)
r2 = r2_score(y_test, y_pred)
mape = mean_absolute_percentage_error(y_test, y_pred)   # cuidado si y_test tiene valores cercanos a 0
```

## Métricas de clasificación

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    roc_auc_score, log_loss, balanced_accuracy_score, matthews_corrcoef,
)

acc = accuracy_score(y_test, y_pred)
precision = precision_score(y_test, y_pred, average="binary")   # "macro"/"micro"/"weighted" para multiclase
recall = recall_score(y_test, y_pred, average="binary")
f1 = f1_score(y_test, y_pred, average="binary")

auc = roc_auc_score(y_test, y_proba[:, 1])   # requiere PROBABILIDADES, no clases predichas
```

### `average` en clasificación multiclase — diferencias que importan

```python
f1_score(y_test, y_pred, average="macro")     # promedio simple entre clases — trata todas por igual
f1_score(y_test, y_pred, average="weighted")  # promedio ponderado por frecuencia de cada clase
f1_score(y_test, y_pred, average="micro")     # agrega TP/FP/FN globalmente antes de calcular — equivale a accuracy en multiclase de una sola etiqueta
```

Con clases desbalanceadas, `macro` penaliza fuerte el mal desempeño en la clase minoritaria (todas las clases pesan igual); `weighted` refleja mejor el desempeño "típico" ponderado por qué tan frecuente es cada clase.

## `classification_report` — resumen completo de un vistazo

```python
from sklearn.metrics import classification_report

print(classification_report(y_test, y_pred, target_names=["no_churn", "churn"]))
```

```
              precision    recall  f1-score   support
   no_churn       0.91      0.95      0.93       850
      churn       0.72      0.58      0.64       150
    accuracy                           0.89      1000
   macro avg       0.81      0.76      0.78      1000
weighted avg       0.88      0.89      0.88      1000
```

## `confusion_matrix` y su visualización

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay

cm = confusion_matrix(y_test, y_pred)
print(cm)   # filas = clase real, columnas = clase predicha

disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=["no_churn", "churn"])
disp.plot(cmap="Blues")
```

## Curvas ROC y Precision-Recall

```python
from sklearn.metrics import RocCurveDisplay, PrecisionRecallDisplay

RocCurveDisplay.from_predictions(y_test, y_proba[:, 1])
PrecisionRecallDisplay.from_predictions(y_test, y_proba[:, 1])

# o directamente desde el estimador ya entrenado (evita calcular y_proba manualmente):
RocCurveDisplay.from_estimator(modelo, X_test, y_test)
```

`PrecisionRecallDisplay` es preferible a `RocCurveDisplay` cuando las clases están muy desbalanceadas — ROC-AUC puede verse engañosamente alto en ese escenario, mientras Precision-Recall refleja mejor el desempeño real sobre la clase minoritaria.

## Métricas de clustering (sin ground truth)

```python
from sklearn.metrics import silhouette_score, davies_bouldin_score, calinski_harabasz_score

sil = silhouette_score(X, labels)          # rango [-1, 1], mayor es mejor
db = davies_bouldin_score(X, labels)        # menor es mejor
ch = calinski_harabasz_score(X, labels)     # mayor es mejor
```

Útiles cuando no hay etiquetas verdaderas para comparar (clustering no supervisado) — evalúan qué tan compactos y separados quedan los clusters usando solo la estructura de los datos y las asignaciones del algoritmo.

## Métricas de clustering (con ground truth)

```python
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score

ari = adjusted_rand_score(y_true, labels_predichos)   # compara clusters contra etiquetas reales conocidas
```

## `make_scorer` — convertir cualquier función en un scorer usable por `cross_val_score`/`GridSearchCV`

```python
from sklearn.metrics import make_scorer

def mape_ponderado(y_true, y_pred, pesos):
    error_pct = abs(y_pred - y_true) / y_true
    return (error_pct * pesos).sum() / pesos.sum()

scorer_custom = make_scorer(
    mape_ponderado,
    greater_is_better=False,   # se negará internamente, ya que menor error es mejor
    pesos=pesos_por_region,     # kwargs adicionales de la función se pasan aquí
)

cross_val_score(pipeline, X, y, scoring=scorer_custom, cv=5)
```

## `get_scorer_names()` — strings válidos para `scoring=`

```python
from sklearn.metrics import get_scorer_names
print(get_scorer_names())
# ['accuracy', 'balanced_accuracy', 'f1', 'f1_macro', 'neg_mean_absolute_error',
#  'neg_root_mean_squared_error', 'r2', 'roc_auc', 'roc_auc_ovr', ...]
```

Cualquiera de estos strings funciona directamente en `scoring=` de `cross_val_score`, `GridSearchCV`, `RandomizedSearchCV` — evita tener que importar y envolver la función manualmente para los casos estándar.

## Reportes multi-métrica con `cross_validate`

```python
from sklearn.model_selection import cross_validate

resultados = cross_validate(
    pipeline, X_train, y_train, cv=5,
    scoring={
        "mae": "neg_mean_absolute_error",
        "rmse": "neg_root_mean_squared_error",
        "r2": "r2",
    },
)
for metrica in ["mae", "rmse", "r2"]:
    print(f"{metrica}: {resultados[f'test_{metrica}'].mean():.3f}")
```

## Ver también

- [[04 - Model Selection - Validación y Búsqueda]]
- `Machine Learning/38-Metricas-de-Regresion-Cuando-Usar-y-Cuando-No.md`
- `MLflow/11 - Evaluación de Modelos (mlflow.evaluate).md`
