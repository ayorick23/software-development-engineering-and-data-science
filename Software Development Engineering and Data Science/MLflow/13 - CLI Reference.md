---
tags: [mlflow, mlops, cli, referencia, cheat-sheet]
---

# 13 — CLI Reference

> Consolida comandos usados a lo largo de todo el cheat-sheet: [[03 - Tracking - Servidor, Backend Store y Artifact Store]], [[08 - Projects - Empaquetado Reproducible]], [[09 - Model Serving y Despliegue]].

Referencia rápida de todos los subcomandos de la CLI `mlflow`.

## `mlflow server` — levantar el servidor de Tracking

```bash
mlflow server \
  --backend-store-uri postgresql://user:pass@host:5432/mlflow \
  --default-artifact-root s3://bucket/artifacts \
  --host 0.0.0.0 \
  --port 5000 \
  --workers 4
```

| Flag | Propósito |
|---|---|
| `--backend-store-uri` | Dónde guardar metadatos (filesystem, SQLite, Postgres, MySQL) |
| `--default-artifact-root` | Dónde guardar artefactos por defecto |
| `--serve-artifacts` | El servidor actúa como proxy de artefactos (el cliente no necesita credenciales de la nube) |
| `--artifacts-destination` | Ruta real de artefactos cuando se usa `--serve-artifacts` |
| `--host` / `--port` | Interfaz de red donde escucha |
| `--workers` | Número de procesos worker (Gunicorn) |
| `--app-name` | Módulo de app custom (ej. `basic-auth`) |

## `mlflow ui` — UI local sin modo servidor completo

```bash
mlflow ui --backend-store-uri ./mlruns --port 5000
```

Diferencia con `mlflow server`: `ui` es para uso local/individual (menos configuración, sin workers múltiples); `server` es el modo pensado para exponerse a un equipo.

## `mlflow run` — ejecutar un Project

```bash
mlflow run . -P param1=valor -e entry_point_name --env-manager conda
mlflow run https://github.com/org/repo.git -v main
```

Ver [[08 - Projects - Empaquetado Reproducible]] para el detalle completo.

## `mlflow models` — servir y probar modelos

```bash
mlflow models serve -m "models:/mi-modelo@champion" --port 5001
mlflow models predict -m runs:/abc123/model -i input.csv -o output.csv
mlflow models build-docker -m "models:/mi-modelo@champion" -n mi-imagen:v1
mlflow models generate-dockerfile -m "models:/mi-modelo@champion" -d ./salida
```

Ver [[09 - Model Serving y Despliegue]].

## `mlflow experiments` — gestión de experiments

```bash
mlflow experiments create --experiment-name "nuevo-proyecto"
mlflow experiments search                                    # listar todos
mlflow experiments rename --experiment-id 1 --new-name "renombrado"
mlflow experiments delete --experiment-id 1
mlflow experiments restore --experiment-id 1
mlflow experiments csv --experiment-id 1 --filename export.csv
```

## `mlflow runs` — gestión de runs individuales

```bash
mlflow runs list --experiment-id 1
mlflow runs describe --run-id abc123
mlflow runs delete --run-id abc123
mlflow runs restore --run-id abc123
```

## `mlflow artifacts` — descargar/listar artefactos sin cargar el modelo completo

```bash
mlflow artifacts list --run-id abc123
mlflow artifacts download --run-id abc123 --artifact-path model -d ./destino
```

## `mlflow db` — administración del backend store

```bash
mlflow db upgrade postgresql://user:pass@host:5432/mlflow   # aplicar migraciones de schema tras actualizar MLflow
```

Es **obligatorio** correr esto después de actualizar la versión de MLflow en un servidor con backend store en base de datos — el schema de tracking evoluciona entre versiones.

## `mlflow gc` — purgar registros soft-deleted

```bash
mlflow gc --backend-store-uri postgresql://... --older-than 30d
```

Elimina **permanentemente** runs/experiments marcados como eliminados (soft delete) hace más del tiempo indicado. Libera espacio real en el backend store y en el artifact store.

## `mlflow deployments` — despliegue en plataformas cloud

```bash
mlflow deployments create -t sagemaker -m "models:/mi-modelo@champion" --name mi-endpoint
mlflow deployments list -t sagemaker
mlflow deployments delete -t sagemaker --name mi-endpoint
```

Ver [[09 - Model Serving y Despliegue]].

## `mlflow gateway` — AI Gateway / Deployments Server para LLMs

```bash
mlflow gateway start --config-path config.yaml --port 7000
```

Ver [[12 - MLflow para LLMs y GenAI]].

## Variables de entorno más usadas

| Variable | Propósito |
|---|---|
| `MLFLOW_TRACKING_URI` | Tracking URI por defecto, sin llamar `set_tracking_uri` en código |
| `MLFLOW_REGISTRY_URI` | Registry URI, si es distinto del Tracking URI |
| `MLFLOW_TRACKING_USERNAME` / `MLFLOW_TRACKING_PASSWORD` | Credenciales de HTTP Basic Auth |
| `MLFLOW_EXPERIMENT_NAME` / `MLFLOW_EXPERIMENT_ID` | Experiment por defecto |
| `MLFLOW_S3_ENDPOINT_URL` | Endpoint custom de S3 (útil con MinIO on-premise) |
| `MLFLOW_ENABLE_ARTIFACTS_PROGRESS_BAR` | Barra de progreso al subir/bajar artefactos grandes |
| `MLFLOW_HTTP_REQUEST_TIMEOUT` | Timeout de llamadas HTTP al servidor de Tracking |

## Ver también

- [[03 - Tracking - Servidor, Backend Store y Artifact Store]]
- [[08 - Projects - Empaquetado Reproducible]]
- [[09 - Model Serving y Despliegue]]
