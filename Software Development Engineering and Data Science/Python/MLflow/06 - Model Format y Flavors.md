---
tags: [mlflow, mlops, models, cheat-sheet]
---

# 06 — Model Format y Flavors

> Continúa de [[01 - Introducción y Arquitectura General]]. Precede a [[07 - Model Registry]] y [[09 - Model Serving y Despliegue]].

**MLflow Models** define un formato estándar para empaquetar modelos entrenados, de forma que sean servibles desde múltiples herramientas (REST API, batch scoring, Spark UDF) sin importar en qué framework se entrenaron.

## La carpeta de un modelo MLflow

Cuando haces `mlflow.xgboost.log_model(model, "model")`, se genera una carpeta con esta estructura:

```
model/
├── MLmodel              # archivo YAML "manifiesto" — el corazón del formato
├── model.xgb             # el modelo serializado en formato nativo de la librería
├── conda.yaml             # entorno conda para reproducir el modelo
├── python_env.yaml        # entorno pip alternativo
├── requirements.txt       # dependencias pip
└── input_example.json     # (opcional) ejemplo de entrada para pruebas
```

### El archivo `MLmodel`

```yaml
artifact_path: model
flavors:
  python_function:
    loader_module: mlflow.xgboost
    python_version: 3.10.12
    data: model.xgb
  xgboost:
    xgb_version: 2.0.3
    data: model.xgb
    model_class: xgboost.core.Booster
signature:
  inputs: '[{"name": "dias_atras", "type": "long"}, {"name": "region", "type": "string"}]'
  outputs: '[{"type": "double"}]'
run_id: a1b2c3d4e5f6
utc_time_created: '2026-08-14 10:15:23.123456'
mlflow_version: 2.16.0
```

Este archivo es lo que le permite a `mlflow models serve` o al Model Registry saber **cómo cargar** el modelo sin conocer de antemano en qué librería fue entrenado.

## El concepto de "flavor"

Un **flavor** es una convención de cómo guardar y cargar un modelo para un framework específico. Un mismo modelo puede tener múltiples flavors simultáneamente.

| Flavor | Librería | Función de logging |
|---|---|---|
| `sklearn` | scikit-learn | `mlflow.sklearn.log_model()` |
| `xgboost` | XGBoost | `mlflow.xgboost.log_model()` |
| `lightgbm` | LightGBM | `mlflow.lightgbm.log_model()` |
| `pytorch` | PyTorch | `mlflow.pytorch.log_model()` |
| `tensorflow` / `keras` | TensorFlow/Keras | `mlflow.tensorflow.log_model()` |
| `spark` | Spark ML | `mlflow.spark.log_model()` |
| `statsmodels` | statsmodels | `mlflow.statsmodels.log_model()` |
| `prophet` | Prophet (forecasting) | `mlflow.prophet.log_model()` |
| `pyfunc` | **Cualquier código Python custom** | `mlflow.pyfunc.log_model()` |

El flavor **`python_function` (pyfunc)** es especial: es un "flavor universal" que todos los demás implementan también. Cualquier modelo guardado con MLflow, sin importar el framework, se puede cargar de forma genérica vía `mlflow.pyfunc.load_model()` — esto es lo que hace posible que `mlflow models serve` funcione igual para un modelo sklearn que para uno de PyTorch.

## Guardar un modelo (`log_model` vs `save_model`)

```python
import mlflow.sklearn

# log_model: guarda el modelo COMO ARTEFACTO de un run activo
with mlflow.start_run():
    mlflow.sklearn.log_model(model, artifact_path="model")

# save_model: guarda el modelo en una ruta local, SIN necesidad de un run activo
mlflow.sklearn.save_model(model, path="./mi_modelo_local")
```

## Signature — el contrato de entrada/salida

El **signature** describe el schema de columnas y tipos que el modelo espera y produce. Es clave para validación en producción: `mlflow models serve` rechaza requests que no coincidan con el signature.

```python
from mlflow.models.signature import infer_signature

predictions = model.predict(X_train)
signature = infer_signature(X_train, predictions)

mlflow.sklearn.log_model(
    model,
    artifact_path="model",
    signature=signature,
    input_example=X_train.iloc[:5],   # ejemplo real, se guarda junto al modelo
)
```

Definir el signature manualmente (útil cuando `infer_signature` no puede deducir bien los tipos):

```python
from mlflow.types.schema import Schema, ColSpec
from mlflow.models.signature import ModelSignature

input_schema = Schema([
    ColSpec("long", "dias_atras"),
    ColSpec("string", "region"),
])
output_schema = Schema([ColSpec("double")])
signature = ModelSignature(inputs=input_schema, outputs=output_schema)
```

## Cargar un modelo guardado

```python
# Genérico — funciona sin importar el framework original:
model = mlflow.pyfunc.load_model("runs:/a1b2c3.../model")
preds = model.predict(X_new)

# Específico del flavor — devuelve el objeto NATIVO de la librería
# (útil si necesitas métodos que pyfunc no expone, ej. feature_importances_)
model = mlflow.xgboost.load_model("runs:/a1b2c3.../model")
importances = model.get_score(importance_type="gain")
```

### URIs para referenciar modelos

```python
"runs:/<run_id>/model"                          # modelo de un run específico
"models:/nombre-modelo/1"                        # versión 1 del modelo en el Registry
"models:/nombre-modelo/Production"                # alias/stage "Production" del Registry
"models:/nombre-modelo@champion"                  # alias con sintaxis @ (enfoque moderno)
"s3://bucket/path/to/model"                       # ruta directa en artifact store
"./modelo_local"                                  # ruta en filesystem local
```

## Modelos custom con `pyfunc` — cuando ningún flavor nativo alcanza

Para lógica de inferencia que combina varios modelos, reglas de negocio, o preprocesamiento no estándar:

```python
import mlflow.pyfunc

class ModeloConReglas(mlflow.pyfunc.PythonModel):
    def load_context(self, context):
        import joblib
        self.model = joblib.load(context.artifacts["modelo_sklearn"])
        self.umbral = 0.5

    def predict(self, context, model_input, params=None):
        probs = self.model.predict_proba(model_input)[:, 1]
        # regla de negocio custom encima de la predicción del modelo
        return (probs > self.umbral).astype(int)

with mlflow.start_run():
    mlflow.pyfunc.log_model(
        artifact_path="modelo_custom",
        python_model=ModeloConReglas(),
        artifacts={"modelo_sklearn": "ruta/al/modelo.pkl"},
        pip_requirements=["scikit-learn==1.5.0", "joblib"],
    )
```

Este patrón es el que se usa cuando el "modelo" en producción en realidad es un **ensamble de reglas + modelo**, o cuando se quiere exponer una función de scoring que no es un `.predict()` directo (por ejemplo, aplicar un `champion/challenger gate` como parte de la inferencia).

## Entornos y reproducibilidad

MLflow guarda automáticamente el entorno necesario para reproducir el modelo:

```python
mlflow.sklearn.log_model(
    model,
    artifact_path="model",
    conda_env=None,                    # None = MLflow infiere el entorno automáticamente
    pip_requirements=["scikit-learn==1.5.0", "pandas==2.2.0"],  # o especifica manualmente
    extra_pip_requirements=["mi-libreria-interna==1.2.0"],
)
```

Cargar el modelo en un entorno distinto y validar que las dependencias coinciden:

```bash
mlflow models predict -m runs:/a1b2c3.../model -i input.json --env-manager conda
```

## Ver también

- [[07 - Model Registry]]
- [[09 - Model Serving y Despliegue]]
- [[05 - Autologging en Profundidad]]
