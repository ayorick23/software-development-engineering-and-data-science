---
tags: [streamlit, dashboards, despliegue, docker, secrets, cheat-sheet]
---

# 12 — Despliegue: Community Cloud, Docker y Secrets

> Continúa de [[10 - Multipage Apps y Navegación]].

## `st.secrets` — manejo de credenciales

```toml
# .streamlit/secrets.toml — NUNCA versionar este archivo en Git
[mlflow]
tracking_uri = "http://mlflow-server:5000"

[database]
host = "db.internal"
user = "app_user"
password = "contraseña_secreta"
```

```python
import streamlit as st

tracking_uri = st.secrets["mlflow"]["tracking_uri"]
password = st.secrets["database"]["password"]
```

`st.secrets` lee automáticamente `.streamlit/secrets.toml` en desarrollo local, y en Streamlit Community Cloud se configura a través de la interfaz web del servicio (sin necesitar el archivo físico en el repositorio) — el mecanismo estándar de Streamlit para evitar hardcodear credenciales en el código fuente.

```bash
# .gitignore — imprescindible
.streamlit/secrets.toml
```

## Streamlit Community Cloud — despliegue gratuito directo desde GitHub

```
1. Push del código a un repositorio de GitHub (público o privado)
2. share.streamlit.io → "New app" → seleccionar repo, branch, y archivo principal (app.py)
3. Configurar secrets desde la UI web (Settings → Secrets)
4. Deploy — la app queda disponible en una URL tipo usuario-app.streamlit.app
```

La opción más simple para prototipos, demos internas o herramientas de equipo pequeño — sin gestionar infraestructura propia. Redeploys automáticos en cada push al branch configurado.

### `requirements.txt` — dependencias explícitas

```txt
streamlit==1.40.0
pandas==2.2.0
scikit-learn==1.5.0
mlflow==2.16.0
```

Community Cloud instala automáticamente desde `requirements.txt` en la raíz del repo — fijar versiones exactas (no rangos) evita que un cambio de versión de una dependencia rompa la app silenciosamente entre deploys (mismo principio que `Scikit-learn/13 - Persistencia de Modelos.md` para reproducibilidad de modelos).

## `.streamlit/config.toml` — configuración de tema y servidor

```toml
[theme]
primaryColor = "#FF4B4B"
backgroundColor = "#FFFFFF"
secondaryBackgroundColor = "#F0F2F6"
textColor = "#262730"
font = "sans serif"

[server]
maxUploadSize = 200   # límite de tamaño de archivo subido, en MB
enableXsrfProtection = true

[browser]
gatherUsageStats = false
```

## Despliegue con Docker — control total de infraestructura

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8501

HEALTHCHECK CMD curl --fail http://localhost:8501/_stcore/health || exit 1

ENTRYPOINT ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

```bash
docker build -t mi-app-streamlit .
docker run -p 8501:8501 --env-file .env mi-app-streamlit
```

`--server.address=0.0.0.0` es imprescindible dentro de un contenedor — sin esto, Streamlit solo escucha en `localhost` del contenedor, inaccesible desde fuera. Ver `Docker/Introduction to Docker.md`.

### Docker Compose — Streamlit + backend + base de datos

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8501:8501"
    environment:
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    depends_on:
      - mlflow

  mlflow:
    image: ghcr.io/mlflow/mlflow:latest
    command: mlflow server --host 0.0.0.0 --backend-store-uri sqlite:///mlflow.db
    ports:
      - "5000:5000"
```

Ver `Docker/Docker Compose.md` y `MLflow/03 - Tracking - Servidor, Backend Store y Artifact Store.md` — patrón común cuando el dashboard de Streamlit consume directamente un servidor de MLflow que corre en el mismo entorno.

## Despliegue detrás de un reverse proxy (nginx)

```nginx
server {
    listen 80;
    server_name mi-dashboard.empresa.com;

    location / {
        proxy_pass http://localhost:8501;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";   # CRÍTICO — Streamlit usa WebSockets
        proxy_set_header Host $host;
    }
}
```

> **Los headers de WebSocket (`Upgrade`/`Connection`) son obligatorios**: Streamlit usa WebSockets para mantener la conexión en vivo entre el navegador y el servidor (necesaria para que la interactividad funcione) — un reverse proxy mal configurado sin estos headers específicos hace que la app cargue pero deje de reaccionar a las interacciones del usuario, un problema de despliegue muy común y confuso de diagnosticar si no se sabe qué buscar.

## Variables de entorno como alternativa a `secrets.toml`

```python
import os

tracking_uri = os.environ.get("MLFLOW_TRACKING_URI", "http://localhost:5000")
```

En despliegues con Docker/Kubernetes, suele preferirse inyectar configuración vía variables de entorno del sistema en vez de `st.secrets` — más consistente con cómo se gestionan secretos en el resto de la infraestructura (Kubernetes Secrets, `docker run --env-file`), sin depender del mecanismo específico de Streamlit.

## Autenticación — Streamlit no incluye login nativo completo

```python
import streamlit_authenticator as stauth   # librería de terceros, no parte del core de Streamlit
```

Streamlit no incluye un sistema de autenticación de usuarios "listo para producción" en su núcleo — para necesidades reales de login/roles, se recurre a librerías de terceros (`streamlit-authenticator`) o se delega la autenticación al reverse proxy/gateway que está delante de la app (patrón más robusto para entornos corporativos con SSO existente).

## Ver también

- `Docker/Introduction to Docker.md`
- `Docker/Docker Compose.md`
- `MLflow/03 - Tracking - Servidor, Backend Store y Artifact Store.md`
- [[14 - Buenas Prácticas, Rendimiento y Testing]]
