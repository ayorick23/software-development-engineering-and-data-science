---
tags: [mlflow, mlops, projects, cheat-sheet]
---

# 08 — Projects: Empaquetado Reproducible

> Relacionado con [[01 - Introducción y Arquitectura General]].

**MLflow Projects** es una convención para empaquetar código de ML de forma que cualquiera (o cualquier pipeline de CI/CD) pueda ejecutarlo con el entorno correcto, sin depender de instrucciones manuales tipo "primero instala esto, luego corre aquello".

Es el componente de MLflow menos usado en la práctica moderna (Docker + un orquestador como Airflow suelen cubrir el mismo problema), pero sigue siendo relevante para reproducibilidad estricta y para `mlflow run` en pipelines simples.

## El archivo `MLproject`

Es un YAML en la raíz del proyecto que define nombre, entorno y puntos de entrada:

```yaml
# MLproject
name: claro-rd-demand-forecast

python_env: python_env.yaml
# alternativas: conda_env: conda.yaml   |   docker_env: {image: mi-imagen:latest}

entry_points:
  main:
    parameters:
      dias_atras: {type: int, default: 90}
      n_estimators: {type: int, default: 300}
      learning_rate: {type: float, default: 0.05}
    command: "python train.py --dias-atras {dias_atras} --n-estimators {n_estimators} --learning-rate {learning_rate}"

  evaluate:
    parameters:
      model_uri: {type: string}
    command: "python evaluate.py --model-uri {model_uri}"
```

### `python_env.yaml` — entorno reproducible

```yaml
python: "3.10.12"
build_dependencies:
  - pip
  - setuptools
dependencies:
  - -r requirements.txt
```

## Ejecutar un Project

### Desde línea de comandos

```bash
# Proyecto local
mlflow run . -P dias_atras=60 -P n_estimators=500

# Proyecto en un repo de Git (rama o commit específico)
mlflow run https://github.com/empresa/claro-rd-forecast.git -v main -P dias_atras=90

# Especificar un entry point distinto al default "main"
mlflow run . -e evaluate -P model_uri=models:/claro-rd-demand-model@champion
```

### Desde Python

```python
import mlflow

result = mlflow.projects.run(
    uri=".",
    entry_point="main",
    parameters={"dias_atras": 90, "n_estimators": 300},
    env_manager="virtualenv",   # o "conda", "local"
)
print(result.run_id)
```

## `env_manager` — cómo se resuelve el entorno

| Valor | Comportamiento |
|---|---|
| `local` | Usa el entorno Python actual tal cual, sin aislar dependencias |
| `virtualenv` | Crea un venv nuevo a partir de `python_env.yaml` |
| `conda` | Crea un entorno conda nuevo a partir de `conda.yaml` |

```bash
mlflow run . --env-manager conda
```

## Ejecución en Docker

Para reproducibilidad total (incluyendo dependencias de sistema operativo, no solo Python), el entorno puede ser una imagen Docker completa — ver también `Docker/Introduction to Docker.md`:

```yaml
# MLproject
name: claro-rd-demand-forecast
docker_env:
  image: mi-registro.azurecr.io/claro-rd-training:1.4.0
  volumes: ["/data/claro-rd:/data"]
  environment: ["AWS_ACCESS_KEY_ID", "AWS_SECRET_ACCESS_KEY"]

entry_points:
  main:
    command: "python train.py"
```

```bash
mlflow run . --docker-args gpus=all   # ej. para pasar acceso a GPU al contenedor
```

MLflow ejecuta el contenedor, monta el proyecto dentro, corre el comando del entry point y automáticamente conecta el `MLFLOW_TRACKING_URI` del entorno host hacia dentro del contenedor, de forma que el tracking sigue funcionando igual.

## Cada ejecución de un Project genera un Run

Un aspecto clave: `mlflow run` **automáticamente crea un run de Tracking**, capturando los parámetros pasados por CLI, el commit de git del código ejecutado, y cualquier logging que el script `train.py` haga internamente con `mlflow.log_*`. Esto conecta Projects y Tracking sin configuración adicional.

## Multi-step workflows — encadenar entry points

Patrón común: un entry point orquesta otros, ejecutándolos como sub-runs:

```python
# workflow.py
import mlflow

with mlflow.start_run(run_name="pipeline-completo") as parent_run:
    prep_run = mlflow.projects.run(".", entry_point="preprocess", parameters={"input": "raw.csv"})
    train_run = mlflow.projects.run(".", entry_point="main", parameters={"data": prep_run.info.run_id})
    eval_run = mlflow.projects.run(".", entry_point="evaluate", parameters={"model_uri": f"runs:/{train_run.info.run_id}/model"})
```

Para pipelines más complejos con dependencias, reintentos y schedules, esto normalmente se delega a un orquestador dedicado (Airflow, Prefect) que invoca cada paso de MLflow como una tarea — ver [[14 - Integraciones con el Ecosistema]] y `Machine Learning/50-Orquestacion-Prefect-y-Airflow.md`.

## Cuándo usar Projects vs. cuándo no

**Sí tiene sentido cuando:**
- Necesitas que terceros (u otro equipo) ejecuten tu código de entrenamiento sin conocer los detalles internos de instalación.
- Quieres reproducibilidad estricta ligada a un commit de git específico.

**Probablemente no lo necesitas si:**
- Ya tienes Docker + un orquestador (Airflow/Prefect/GitLab CI) manejando la ejecución — en ese caso el "entry point reproducible" ya lo resuelve el Dockerfile + el pipeline CI/CD, y añadir Projects encima es una capa redundante.

## Ver también

- [[01 - Introducción y Arquitectura General]]
- [[14 - Integraciones con el Ecosistema]]
- `Docker/Docker Compose.md`
- `Machine Learning/50-Orquestacion-Prefect-y-Airflow.md`
