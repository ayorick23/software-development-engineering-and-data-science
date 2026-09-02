---
tags: [fastapi, apis, despliegue, docker, python, cheat-sheet]
---

# 15 — Despliegue y Producción

> Continúa de [[14 - Routers y Estructura de Proyectos Grandes]]. El Dockerfile básico ya se vio en [[49-APIs-con-FastAPI-para-Servir-Modelos]]; aquí se profundiza en configuración de producción más completa.

## Uvicorn con múltiples workers

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

Cada worker es un proceso Python independiente (importante: no comparten memoria — una variable global cacheada en un worker no existe en los otros). El número de workers se ajusta típicamente a `2 × núcleos_de_CPU + 1` como punto de partida.

## Gunicorn + Uvicorn workers — el estándar en producción

```bash
pip install gunicorn
gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

Gunicorn añade gestión de procesos más robusta que Uvicorn solo: reinicio automático de workers caídos, graceful shutdown, y mejor manejo de señales del sistema operativo — Uvicorn sigue siendo el que ejecuta el código ASGI, Gunicorn solo orquesta los procesos.

## Configuración con `pydantic-settings` — variables de entorno tipadas

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    debug: bool = False
    workers: int = 4

    model_config = {"env_file": ".env"}

settings = Settings()   # lee de variables de entorno o del .env, con validación de tipos
```

```python
@app.get("/config-actual")
def ver_config(settings: Annotated[Settings, Depends(lambda: settings)]):
    return {"debug": settings.debug}
```

Igual que con `BaseModel` (ver [[03 - Request Body y Modelos Pydantic]]), `BaseSettings` valida tipos automáticamente — si `DATABASE_URL` no está seteada, la app falla al iniciar con un error claro, en vez de fallar silenciosamente más tarde en producción.

## Dockerfile multi-stage para producción

```dockerfile
# --- Etapa de build ---
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# --- Etapa final, más liviana ---
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

EXPOSE 8000
CMD ["gunicorn", "main:app", "--workers", "4", "--worker-class", "uvicorn.workers.UvicornWorker", "--bind", "0.0.0.0:8000"]
```

El multi-stage build separa la instalación de dependencias (que necesita herramientas de compilación) de la imagen final que efectivamente corre — imagen más pequeña y con menos superficie de ataque, siguiendo el mismo principio que ya se vería en cualquier `Dockerfile` de producción bien escrito.

## Healthcheck en Docker/Kubernetes

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:8000/health || exit 1
```

Complementa el endpoint `/health` (o `/health/liveness` y `/health/readiness`) ya cubierto en [[49-APIs-con-FastAPI-para-Servir-Modelos]] — el orquestador (Docker, Kubernetes, un balanceador de carga) usa esto para decidir si reiniciar el contenedor o dejar de enviarle tráfico.

## Reverse proxy — Nginx delante de Uvicorn/Gunicorn

```nginx
server {
    listen 443 ssl;
    server_name api.miapp.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

En producción, casi nunca se expone Uvicorn/Gunicorn directamente a internet — un reverse proxy (Nginx, Traefik, o el load balancer de la nube) maneja TLS/HTTPS, compresión, y actúa como capa adicional de seguridad delante de la aplicación Python.

## Logs estructurados en producción

```python
import logging
import sys

logging.basicConfig(
    level=logging.INFO,
    format='{"timestamp": "%(asctime)s", "level": "%(levelname)s", "message": "%(message)s"}',
    stream=sys.stdout,
)
```

Igual que en [[49-APIs-con-FastAPI-para-Servir-Modelos]], loguear en formato JSON a `stdout` (en vez de a un archivo) es la convención esperada por la mayoría de orquestadores modernos (Docker, Kubernetes) — el sistema de logs de la plataforma se encarga de recolectar y centralizar desde ahí.

## Ver también

- [[14 - Routers y Estructura de Proyectos Grandes]]
- [[49-APIs-con-FastAPI-para-Servir-Modelos]]
- [[Python/FastAPI/16 - Integración con el Ecosistema]]
