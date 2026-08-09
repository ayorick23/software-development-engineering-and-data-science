---
tags: [docker, mlops, contenedores, devops]
---

# 47 — Docker en Profundidad para Machine Learning

> Nota del mentor: en [[17-Arquitecturas-de-Despliegue-de-Modelos]] ya viste un Dockerfile básico. Aquí vamos a fondo — Docker es probablemente la herramienta individual con mayor retorno de inversión de aprendizaje en todo tu stack de MLOps, porque resuelve de raíz el "en mi máquina funciona" que seguramente ya viviste con las diferencias entre tu entorno de Windows/PowerShell y donde sea que corra el pipeline en producción.

## 1. Por qué contenedores y no solo entornos virtuales

Un entorno virtual de Python (`venv`, visto en [[12-Gestion-Moderna-de-Proyectos-Python]]) aísla las dependencias de Python, pero **no** aísla el sistema operativo, las librerías del sistema, la versión de Python misma, ni variables de entorno del SO. Un contenedor empaqueta **todo eso junto**: SO base, Python, librerías del sistema (como el driver ODBC que necesitas para conectar a SQL Server), y tu código — garantizando que lo que corre en tu laptop es *literalmente* el mismo entorno que corre en el runner de GitLab CI y en el servidor de producción.

## 2. Anatomía de un Dockerfile bien construido para ML

```dockerfile
# Etapa 1: build — instala dependencias con las herramientas de compilación necesarias
FROM python:3.11-slim AS builder

RUN apt-get update && apt-get install -y --no-install-recommends \
    unixodbc-dev \
    gcc \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY pyproject.toml .
RUN pip install --no-cache-dir --user .

# Etapa 2: runtime — imagen final, sin herramientas de compilación
FROM python:3.11-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    unixodbc \
    curl \
    && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

WORKDIR /app
COPY src/ src/
COPY config/ config/

RUN useradd --create-home appuser
USER appuser

CMD ["python", "-m", "forecasting_pipeline.main"]
```

## 3. Multi-stage builds — por qué la imagen final debe ser mínima

El Dockerfile de arriba usa **dos etapas** (`builder` y la etapa final): la primera instala `gcc` y herramientas de compilación necesarias para compilar dependencias con extensiones en C (comunes en el ecosistema de ML — numpy, scipy, xgboost), pero esas herramientas **no** se necesitan en producción, solo durante la instalación. La segunda etapa copia únicamente el resultado ya compilado, descartando `gcc` y todo lo demás del `builder`.

**Impacto real**: una imagen sin multi-stage puede pesar 1.5-2GB; con multi-stage, frecuentemente baja a 300-500MB. Esto importa porque cada despliegue transfiere esa imagen completa por la red, y una imagen más pequeña también reduce la superficie de ataque de seguridad (menos software instalado = menos vulnerabilidades potenciales).

## 4. Usuario no-root — un detalle de seguridad que casi todos los tutoriales omiten

```dockerfile
RUN useradd --create-home appuser
USER appuser
```

Sin esto, tu contenedor corre como `root` por defecto — si alguien compromete la aplicación dentro del contenedor (una vulnerabilidad de una dependencia, por ejemplo), tiene privilegios de root **dentro del contenedor**, lo cual en ciertas configuraciones puede facilitar escapar hacia el host. Correr como un usuario sin privilegios especiales es una práctica de seguridad básica que cuesta dos líneas y reduce significativamente el impacto potencial de una vulnerabilidad.

## 5. `.dockerignore` — no metas basura a la imagen

```
# .dockerignore
.git/
.venv/
__pycache__/
*.pyc
.env
tests/
notebooks/
data/raw/
```

Sin `.dockerignore`, `COPY . .` mete **todo** el directorio al contexto de build — incluyendo tu historial de git, notebooks pesados, y potencialmente el `.env` con secretos (un riesgo real de seguridad si ese `.env` termina embebido en una capa de la imagen). Además, un contexto de build más pequeño acelera significativamente el proceso de `docker build`.

## 6. Docker Compose — orquestar múltiples contenedores en desarrollo local

```yaml
# docker-compose.yml
services:
  pipeline:
    build: .
    environment:
      - QFLOW_DB_CONNECTION_STRING=${QFLOW_DB_CONNECTION_STRING}
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    depends_on:
      - mlflow

  mlflow:
    image: ghcr.io/mlflow/mlflow
    ports:
      - "5000:5000"
    command: mlflow server --host 0.0.0.0 --backend-store-uri sqlite:///mlflow.db

  sqlserver-dev:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=${SA_PASSWORD}
    ports:
      - "1433:1433"
```

```bash
docker compose up   # levanta pipeline + servidor MLflow + SQL Server de desarrollo, todo junto
```

Esto te permite replicar localmente una versión completa de tu stack (pipeline + MLflow + una base de datos de prueba) con un solo comando — extremadamente útil para que un compañero nuevo del equipo tenga un entorno funcional en minutos, en vez de instalar y configurar cada pieza manualmente en su máquina.

## 7. Capas de Docker y caché de build — por qué el ORDEN de las instrucciones importa

```dockerfile
# MAL orden: copiar el código ANTES de instalar dependencias
COPY . .
RUN pip install .
# cada cambio de código invalida el caché de la instalación de dependencias — reinstala TODO cada vez

# BUEN orden: dependencias primero, código después
COPY pyproject.toml .
RUN pip install .
COPY src/ src/
# cambiar el código NO invalida el caché de "pip install" — solo reconstruye la última capa
```

Docker construye la imagen en **capas**, y cada capa se cachea si su contenido (y el de las instrucciones anteriores) no cambió. Copiar los archivos que cambian con menos frecuencia (`pyproject.toml`) antes que los que cambian constantemente (`src/`) significa que la mayoría de tus builds solo reconstruyen la última capa — la diferencia entre un build de 10 segundos y uno de 3 minutos reinstalando todas las dependencias desde cero cada vez que editas una línea de código.

## 8. Conexión con el resto del stack

Este Dockerfile es exactamente lo que tu `.gitlab-ci.yml` de [[14-CICD-para-ML-con-GitLab]] construye y publica en cada pipeline exitoso, y es la unidad de despliegue que discutimos en [[17-Arquitecturas-de-Despliegue-de-Modelos]] — sin importar si el destino final es un servidor tradicional, Kubernetes, o Azure Container Instances, la imagen Docker es el artefacto portable que garantiza consistencia entre todos esos entornos.

## Ver también
- [[14-CICD-para-ML-con-GitLab]]
- [[17-Arquitecturas-de-Despliegue-de-Modelos]]
- [[12-Gestion-Moderna-de-Proyectos-Python]]
- [[24-Configuracion-Profesional-de-Proyectos]]
