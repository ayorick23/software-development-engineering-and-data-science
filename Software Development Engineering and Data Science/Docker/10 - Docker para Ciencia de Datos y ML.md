---
tags: [docker, ciencia-de-datos, machine-learning, cheat-sheet]
---

# 10 — Docker para Ciencia de Datos y ML

> Para prácticas específicas de Dockerfiles de ML (multi-stage, capas, `.dockerignore` aplicado a modelos), ver [[Machine Learning/47-Docker-en-Profundidad-para-ML]]. Este archivo cubre los patrones operativos más frecuentes: Jupyter en contenedor, GPUs, y el flujo de entrenamiento/serving.

## Los problemas de DS/ML que Docker resuelve

- **Reproducibilidad**: un modelo debe poder reentrenarse con exactamente el mismo entorno (versiones de NumPy, CUDA, scikit-learn) meses después, algo que "instalar de nuevo con pip" no garantiza si las versiones no están fijadas.
- **Entrenamiento vs. serving**: el entorno donde se entrena un modelo y el entorno donde se sirven predicciones deben ser idénticos — de lo contrario, diferencias sutiles de versión pueden cambiar el comportamiento del modelo en producción.
- **GPUs**: instalar drivers CUDA correctamente en cada máquina de desarrollo es frágil; empaquetarlos en la imagen evita ese problema para cada colaborador.

## Entorno de Jupyter en un contenedor

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt jupyter

COPY . .
EXPOSE 8888
CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--allow-root", "--no-browser"]
```

```bash
docker run -it -p 8888:8888 -v $(pwd):/app mi-imagen-ds
```

El volumen `-v $(pwd):/app` (ver [[07 - Volúmenes y Persistencia de Datos]]) monta el código y los notebooks del host dentro del contenedor — los cambios se guardan directamente en el host, sin necesidad de reconstruir la imagen en cada edición.

## Acceso a GPU: NVIDIA Container Toolkit

```bash
# Instalación del toolkit en el host (una sola vez, fuera de Docker)
# https://github.com/NVIDIA/nvidia-container-toolkit

docker run --gpus all mi-imagen-ml           # acceso a todas las GPUs del host
docker run --gpus '"device=0,1"' mi-imagen-ml  # solo GPUs específicas
```

```dockerfile
FROM nvidia/cuda:12.2.0-runtime-ubuntu22.04   # imagen base con drivers/runtime CUDA ya incluidos

RUN apt-get update && apt-get install -y python3-pip
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt   # torch/tensorflow con soporte CUDA
```

Verificar dentro del contenedor que la GPU es visible:

```python
import torch
torch.cuda.is_available()   # debe dar True si --gpus se pasó correctamente y la imagen tiene el driver adecuado
```

## Entrenamiento y serving como imágenes separadas

Un patrón común de multi-stage (ver [[03 - Multi-stage Builds y Optimización de Imágenes]]) aplicado específicamente a ML: la imagen de entrenamiento incluye herramientas pesadas (datasets, notebooks, librerías de experimentación); la imagen de serving contiene solo el modelo serializado y un servidor ligero.

```dockerfile
# Imagen de entrenamiento -- pesada, con todo el stack de experimentación
FROM python:3.11 AS entrenamiento
COPY requirements-train.txt .
RUN pip install -r requirements-train.txt
COPY entrenar.py .
RUN python entrenar.py   # produce modelo.pkl

# Imagen de serving -- ligera, solo lo necesario para predecir
FROM python:3.11-slim AS serving
COPY requirements-serve.txt .
RUN pip install -r requirements-serve.txt   # solo fastapi/flask + librería de inferencia, sin herramientas de EDA
COPY --from=entrenamiento /modelo.pkl .
COPY servir.py .
CMD ["uvicorn", "servir:app", "--host", "0.0.0.0"]
```

Ver [[Python/FastAPI/01 - Introducción y Arquitectura|FastAPI]] para el servidor de inferencia, y [[Python/MLflow/01 - Introducción y Arquitectura General|MLflow]] para versionar el artefacto `modelo.pkl` en vez de copiarlo directamente en la imagen.

## No incluir los datos en la imagen

```dockerfile
# MAL: el dataset queda embebido permanentemente en la imagen, inflándola y volviéndola difícil de actualizar
COPY dataset_entrenamiento.csv /app/data/

# BIEN: el dataset se monta como volumen en tiempo de ejecución, la imagen no lo conoce
```

```bash
docker run -v /ruta/al/dataset:/app/data mi-imagen-ml
```

## Ver también

- [[03 - Multi-stage Builds y Optimización de Imágenes]]
- [[07 - Volúmenes y Persistencia de Datos]]
- [[Machine Learning/47-Docker-en-Profundidad-para-ML]]
- [[Python/MLflow/01 - Introducción y Arquitectura General]]
