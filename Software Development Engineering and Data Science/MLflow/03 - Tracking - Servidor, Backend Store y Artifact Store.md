---
tags: [mlflow, mlops, tracking, infraestructura, cheat-sheet]
---

# 03 — Tracking: Servidor, Backend Store y Artifact Store

> Continúa de [[02 - Tracking - Fundamentos y API de Logging]].

Cuando un equipo completo necesita ver los mismos experimentos, MLflow deja de ser "una carpeta local" y pasa a ser un **servicio cliente-servidor**. Entender esta arquitectura es clave para no confundir dónde vive cada cosa.

## Los dos almacenes que usa MLflow Tracking

MLflow **separa** dónde guarda metadatos de dónde guarda archivos pesados:

| Almacén | Qué guarda | Tecnologías típicas |
|---|---|---|
| **Backend Store** | Metadatos: params, metrics, tags, experiment/run info | Filesystem local, SQLite, Postgres, MySQL |
| **Artifact Store** | Archivos: modelos serializados, gráficas, datasets | Filesystem local, S3, Azure Blob Storage, GCS, DBFS |

Esta separación importa porque una base de datos relacional no está diseñada para guardar archivos de varios GB (un modelo entrenado), y un sistema de archivos plano no es bueno para hacer queries rápidas tipo "dame todos los runs con `rmse < 20`".

```mermaid
flowchart LR
    Client[Cliente Python\nmlflow.log_*] -->|HTTP REST| Server[mlflow server]
    Server -->|metadatos| DB[(Backend Store\nPostgres/MySQL)]
    Server -->|archivos| Store[(Artifact Store\nS3 / Azure Blob)]
```

## Levantar un servidor de Tracking

### Opción mínima (solo filesystem, sin base de datos)

```bash
mlflow server \
  --backend-store-uri ./mlflow-data \
  --default-artifact-root ./mlflow-artifacts \
  --host 0.0.0.0 \
  --port 5000
```

Válido para pruebas, pero **no soporta consultas concurrentes robustas** ni el Model Registry en modo completo (el Registry requiere backend store con base de datos relacional, no filesystem).

### Opción de producción (Postgres + S3)

```bash
mlflow server \
  --backend-store-uri postgresql://mlflow_user:password@db-host:5432/mlflow_db \
  --default-artifact-root s3://mi-bucket/mlflow-artifacts \
  --host 0.0.0.0 \
  --port 5000 \
  --workers 4
```

Requisitos:

```bash
pip install psycopg2-binary boto3
```

Credenciales de S3 se toman de las variables de entorno estándar de AWS (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) o de un rol IAM si corre en EC2/ECS.

### Con Azure Blob Storage como artifact store

```bash
pip install azure-storage-blob

export AZURE_STORAGE_CONNECTION_STRING="..."

mlflow server \
  --backend-store-uri postgresql://... \
  --default-artifact-root wasbs://mlflow-artifacts@miaccount.blob.core.windows.net/ \
  --host 0.0.0.0 --port 5000
```

## Desplegar el servidor con Docker Compose

Patrón típico para un equipo pequeño-mediano — ver también [[14 - Integraciones con el Ecosistema]] y `Docker/Docker Compose.md`:

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: mlflow
      POSTGRES_PASSWORD: mlflow
      POSTGRES_DB: mlflow
    volumes:
      - pg_data:/var/lib/postgresql/data

  mlflow:
    image: ghcr.io/mlflow/mlflow:latest
    command: >
      mlflow server
      --backend-store-uri postgresql://mlflow:mlflow@postgres:5432/mlflow
      --default-artifact-root s3://mi-bucket/mlflow-artifacts
      --host 0.0.0.0
    ports:
      - "5000:5000"
    environment:
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
    depends_on:
      - postgres

volumes:
  pg_data:
```

```bash
docker compose up -d
```

## Configurar el cliente para apuntar al servidor

```python
import mlflow

mlflow.set_tracking_uri("http://mlflow-server.internal:5000")
```

Equivalente por variable de entorno (útil en contenedores/CI donde no quieres tocar código):

```bash
export MLFLOW_TRACKING_URI="http://mlflow-server.internal:5000"
```

## Tracking URI — formatos soportados

```python
mlflow.set_tracking_uri("file:./mlruns")                       # filesystem local
mlflow.set_tracking_uri("sqlite:///mlflow.db")                 # SQLite local (soporta Registry)
mlflow.set_tracking_uri("postgresql://user:pass@host:5432/db") # Postgres directo (sin servidor HTTP)
mlflow.set_tracking_uri("http://mlflow-server:5000")           # servidor HTTP remoto
mlflow.set_tracking_uri("databricks")                          # Databricks Managed MLflow
```

> **Nota importante**: apuntar directo a `postgresql://...` desde el cliente (sin pasar por `mlflow server`) funciona para Tracking, pero normalmente **no es lo recomendado en equipo** — expone credenciales de la base de datos a cada cliente y salta cualquier capa de autenticación HTTP que hayas puesto delante del servidor.

## Autenticación básica del servidor

MLflow incluye un módulo de auth opcional (usuario/contraseña vía HTTP Basic Auth):

```bash
pip install mlflow[auth]

mlflow server --app-name basic-auth \
  --backend-store-uri postgresql://... \
  --default-artifact-root s3://...
```

Desde el cliente:

```python
import os
os.environ["MLFLOW_TRACKING_USERNAME"] = "admin"
os.environ["MLFLOW_TRACKING_PASSWORD"] = "secreto"
```

Para producción real, lo típico es poner el servidor MLflow **detrás de un reverse proxy** (nginx, un API Gateway) que maneje TLS y autenticación corporativa (SSO/OAuth), en vez de depender solo del auth nativo.

## Artifact Store — acceso directo vs. proxy

Desde MLflow 2.x, el servidor puede actuar como **proxy** para los artefactos (el cliente nunca ve las credenciales de S3 directamente, todo pasa por el servidor):

```bash
mlflow server \
  --backend-store-uri postgresql://... \
  --artifacts-destination s3://mi-bucket/mlflow-artifacts \
  --serve-artifacts \
  --host 0.0.0.0
```

Esto es más seguro (las credenciales de la nube solo las tiene el servidor) pero añade carga de red al servidor, ya que todo artefacto pasa por él.

## Backup y migración

Como el backend store es una base de datos relacional estándar, se respalda igual que cualquier otra:

```bash
pg_dump mlflow_db > mlflow_backup.sql
```

Para migrar experimentos entre servidores, MLflow no tiene un comando nativo de "export/import" completo — se suele usar el `MlflowClient` para leer runs de un servidor origen y re-loguearlos en el destino, o replicar la base de datos y el bucket de artefactos directamente.

## Ver también

- [[02 - Tracking - Fundamentos y API de Logging]]
- [[07 - Model Registry]] (requiere backend store con base de datos)
- [[14 - Integraciones con el Ecosistema]]
- [[15 - Buenas Prácticas, Seguridad y Comparativa]]
