---
tags: [mlflow, mlops, evaluate, validation, cheat-sheet]
---

# 11 — Evaluación de Modelos (`mlflow.evaluate`)

> Se apoya en [[06 - Model Format y Flavors]] y [[02 - Tracking - Fundamentos y API de Logging]].

`mlflow.evaluate()` automatiza el cálculo de métricas estándar, la generación de gráficas de diagnóstico y (opcionalmente) la validación contra umbrales — todo con una sola llamada, en vez de calcular cada métrica manualmente.

## Uso básico — clasificación

```python
import mlflow

with mlflow.start_run():
    mlflow.sklearn.log_model(model, "model")
    model_uri = mlflow.get_artifact_uri("model")

    result = mlflow.evaluate(
        model=model_uri,
        data=df_val,                # DataFrame con features + columna target
        targets="churn",
        model_type="classifier",    # "classifier" | "regressor" | "question-answering" | etc.
        evaluators="default",
    )

print(result.metrics)
```

Con `model_type="classifier"`, automáticamente calcula: accuracy, precision, recall, F1, ROC-AUC, log loss — y genera artefactos: matriz de confusión, curva ROC, curva precision-recall, todos logueados como artifacts del run.

## Uso básico — regresión

```python
result = mlflow.evaluate(
    model=model_uri,
    data=df_val,
    targets="demanda_real",
    model_type="regressor",
)

print(result.metrics)
# {'mean_absolute_error': 12.4, 'root_mean_squared_error': 18.7, 'r2_score': 0.89, ...}
```

Artefactos generados automáticamente: gráfica de residuos, scatter de predicho vs. real.

## Métricas custom

Cuando las métricas estándar no capturan lo que importa al negocio (ej. un MAE ponderado por región):

```python
from mlflow.models import make_metric

def mape_ponderado(eval_df, _builtin_metrics):
    pesos = eval_df["pesos_region"]
    error_pct = abs(eval_df["prediction"] - eval_df["target"]) / eval_df["target"]
    return (error_pct * pesos).sum() / pesos.sum()

metrica_custom = make_metric(
    eval_fn=mape_ponderado,
    greater_is_better=False,
    name="mape_ponderado_negocio",
)

result = mlflow.evaluate(
    model=model_uri,
    data=df_val,
    targets="demanda_real",
    model_type="regressor",
    extra_metrics=[metrica_custom],
)
```

## Model Validation — gates automáticos

`mlflow.evaluate` puede **fallar explícitamente** (lanzar excepción) si el modelo no supera umbrales definidos — patrón directo para un gate de CI/CD antes de promover un modelo:

```python
from mlflow.models import MetricThreshold

thresholds = {
    "mean_absolute_error": MetricThreshold(
        threshold=15.0,
        greater_is_better=False,
    ),
    "r2_score": MetricThreshold(
        threshold=0.80,
        greater_is_better=True,
    ),
}

mlflow.evaluate(
    model=model_uri,
    data=df_val,
    targets="demanda_real",
    model_type="regressor",
    validation_thresholds=thresholds,
)
# Lanza mlflow.models.evaluation.validation.ModelValidationFailedException si no pasa
```

### Comparar contra el modelo en producción (baseline)

```python
baseline_model_uri = "models:/claro-rd-demand-model@champion"

thresholds = {
    "mean_absolute_error": MetricThreshold(
        threshold=15.0,
        min_relative_change=0.05,   # debe ser al menos 5% mejor que el baseline
        greater_is_better=False,
    ),
}

mlflow.evaluate(
    model=challenger_model_uri,
    data=df_val,
    targets="demanda_real",
    model_type="regressor",
    validation_thresholds=thresholds,
    baseline_model=baseline_model_uri,
)
```

Este es el mecanismo formal para el patrón champion/challenger: en vez de comparar métricas manualmente, `mlflow.evaluate` hace la comparación estadística y lanza la excepción si el challenger no gana con margen suficiente.

## Evaluación sin un modelo MLflow — solo con predicciones

Cuando ya tienes las predicciones calculadas (no quieres que MLflow vuelva a correr el modelo):

```python
df_val["prediction"] = model.predict(X_val)

result = mlflow.evaluate(
    data=df_val,
    predictions="prediction",
    targets="demanda_real",
    model_type="regressor",
)
```

## SHAP — explicabilidad automática

Si `shap` está instalado, `mlflow.evaluate` puede generar automáticamente gráficas de importancia de features:

```python
result = mlflow.evaluate(
    model=model_uri,
    data=df_val,
    targets="demanda_real",
    model_type="regressor",
    evaluators="default",
    evaluator_config={"log_explainer": True},
)
```

Esto genera y loguea un *SHAP summary plot* como artefacto — ver también `Machine Learning/39-Interpretabilidad-de-Modelos.md`.

## Integración en un pipeline de reentrenamiento automático

```python
def evaluar_y_promover(challenger_uri, champion_uri, df_val):
    try:
        mlflow.evaluate(
            model=challenger_uri,
            data=df_val,
            targets="demanda_real",
            model_type="regressor",
            baseline_model=champion_uri,
            validation_thresholds={
                "mean_absolute_error": MetricThreshold(threshold=15.0, min_relative_change=0.03, greater_is_better=False),
            },
        )
        return True   # pasó el gate
    except Exception:
        return False  # no supera al champion, no se promueve
```

## Ver también

- [[06 - Model Format y Flavors]]
- [[07 - Model Registry]]
- `Machine Learning/37-Validacion-Rigurosa-en-ML.md`
- `Machine Learning/38-Metricas-de-Regresion-Cuando-Usar-y-Cuando-No.md`
