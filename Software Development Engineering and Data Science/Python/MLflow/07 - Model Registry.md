---
tags: [mlflow, mlops, model-registry, cheat-sheet]
---

# 07 — Model Registry

> Continúa de [[06 - Model Format y Flavors]].

El **Model Registry** es el componente que formaliza el ciclo de vida de un modelo: versionado, transición entre etapas y trazabilidad de qué versión está en producción. Requiere un backend store con base de datos relacional (no funciona en modo filesystem puro) — ver [[03 - Tracking - Servidor, Backend Store y Artifact Store]].

## Registered Model vs. Model Version

```
Registered Model: "claro-rd-demand-model"
 ├── Version 1  (de runs:/abc123/model) — Archived
 ├── Version 2  (de runs:/def456/model) — Production
 └── Version 3  (de runs:/ghi789/model) — Staging (en evaluación)
```

- **Registered Model**: el nombre lógico del modelo (ej. `"claro-rd-demand-model"`). Es el contenedor.
- **Model Version**: cada vez que se registra un nuevo modelo bajo ese nombre, se crea una versión incremental (1, 2, 3...) inmutable.

## Registrar un modelo

### Opción 1 — como parte del logging

```python
with mlflow.start_run():
    mlflow.xgboost.log_model(
        model,
        artifact_path="model",
        registered_model_name="claro-rd-demand-model",  # registra automáticamente
    )
```

### Opción 2 — registrar un modelo ya logueado

```python
result = mlflow.register_model(
    model_uri="runs:/a1b2c3d4e5f6/model",
    name="claro-rd-demand-model",
)
print(result.version)   # ej. 3
```

Si el `Registered Model` no existe, se crea automáticamente en el primer registro.

## El enfoque moderno: Aliases (reemplaza a Stages)

Desde MLflow 2.9, el sistema de **Stages** (`Staging`/`Production`/`Archived`) está **deprecado** en favor de **Aliases** — etiquetas arbitrarias y reasignables que apuntan a una versión específica. Es más flexible: puedes tener alias como `champion`, `challenger`, `shadow`, no solo tres nombres fijos.

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()

# Asignar un alias a una versión específica
client.set_registered_model_alias(
    name="claro-rd-demand-model",
    alias="champion",
    version=3,
)

# Cargar el modelo por alias (no por número de versión fijo)
model = mlflow.pyfunc.load_model("models:/claro-rd-demand-model@champion")

# Reasignar el alias a otra versión (esto ES el "promote to production")
client.set_registered_model_alias(
    name="claro-rd-demand-model",
    alias="champion",
    version=4,
)

# Quitar un alias
client.delete_registered_model_alias(name="claro-rd-demand-model", alias="champion")
```

Patrón típico de **champion/challenger** con aliases:

```python
client.set_registered_model_alias("claro-rd-demand-model", "challenger", new_version)
# ... evaluar el challenger contra el champion actual ...
if challenger_supera_gate:
    client.set_registered_model_alias("claro-rd-demand-model", "champion", new_version)
    client.delete_registered_model_alias("claro-rd-demand-model", "challenger")
```

## El enfoque clásico: Stages (aún soportado, deprecado)

Se documenta porque sigue apareciendo en proyectos existentes y en Databricks Managed MLflow.

```python
client.transition_model_version_stage(
    name="claro-rd-demand-model",
    version=3,
    stage="Production",
    archive_existing_versions=True,   # mueve la versión anterior en Production a Archived
)

# Cargar por stage:
model = mlflow.pyfunc.load_model("models:/claro-rd-demand-model/Production")
```

Etapas estándar: `None` → `Staging` → `Production` → `Archived`.

## Metadata y descripciones

```python
client.update_registered_model(
    name="claro-rd-demand-model",
    description="Modelo de forecasting de demanda para la región RD, reentrenado semanalmente.",
)

client.update_model_version(
    name="claro-rd-demand-model",
    version=3,
    description="Entrenado con ventana de 90 días, incluye feature de estacionalidad.",
)

# Tags a nivel de versión (útil para trazabilidad de qué gate superó)
client.set_model_version_tag(
    name="claro-rd-demand-model", version=3,
    key="validation_mae", value="12.4",
)
```

## Consultar el Registry

```python
# Todas las versiones de un modelo
versions = client.search_model_versions("name='claro-rd-demand-model'")
for v in versions:
    print(v.version, v.current_stage, v.aliases)

# Todos los modelos registrados
for rm in client.search_registered_models():
    print(rm.name, rm.latest_versions)

# Obtener la versión detrás de un alias
mv = client.get_model_version_by_alias("claro-rd-demand-model", "champion")
print(mv.version, mv.run_id)
```

## Trazabilidad modelo ↔ run

Cada `ModelVersion` guarda el `run_id` del que proviene, lo que permite ir de "qué está en producción" hacia "con qué código, datos y métricas se entrenó":

```python
mv = client.get_model_version_by_alias("claro-rd-demand-model", "champion")
run = client.get_run(mv.run_id)
print(run.data.params)     # hiperparámetros exactos del modelo en producción
print(run.data.metrics)    # métricas de validación
print(run.info.artifact_uri)
```

## Webhooks y automatización de transición (Databricks)

En Databricks Managed MLflow, el Registry soporta **webhooks** que disparan automáticamente pipelines de CI/CD cuando una versión cambia de stage (ej. correr tests de integración automáticamente al promover a `Staging`). En MLflow open-source self-hosted, esto se implementa manualmente sondeando el Registry desde el pipeline de CI/CD — ver [[14 - Integraciones con el Ecosistema]].

## Eliminar versiones y modelos

```python
client.delete_model_version(name="claro-rd-demand-model", version=1)

client.delete_registered_model(name="claro-rd-demand-model")  # elimina TODAS las versiones
```

## Patrón completo: gate automático de promoción

```python
def promover_si_supera_gate(nombre_modelo, nueva_version, mae_challenger, mae_gate=15.0):
    client = MlflowClient()
    if mae_challenger < mae_gate:
        client.set_registered_model_alias(nombre_modelo, "champion", nueva_version)
        client.set_model_version_tag(nombre_modelo, nueva_version, "promoted_reason", "beat_gate")
        return True
    client.set_model_version_tag(nombre_modelo, nueva_version, "rejected_reason", "failed_gate")
    return False
```

Esto formaliza lo que antes se hacía con archivos `.bak` y JSON de estado manual: historial completo, auditable y con rollback de un solo llamado.

## Ver también

- [[06 - Model Format y Flavors]]
- [[09 - Model Serving y Despliegue]]
- [[14 - Integraciones con el Ecosistema]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]] (en `Machine Learning/`)
