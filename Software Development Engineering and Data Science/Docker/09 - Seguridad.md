---
tags: [docker, seguridad, cheat-sheet]
---

# 09 — Seguridad

> Para el detalle de por qué el usuario no-root es "un detalle de seguridad que casi todos los tutoriales omiten", ver también [[Machine Learning/47-Docker-en-Profundidad-para-ML#4. Usuario no-root — un detalle de seguridad que casi todos los tutoriales omiten|Machine Learning/47]].

## Usuario no-root dentro del contenedor

Por defecto, un contenedor corre como `root` — si un atacante logra ejecutar código dentro del contenedor (por una vulnerabilidad en la aplicación), tiene privilegios de root *dentro* de ese contenedor, lo que amplía el impacto de un posible escape de contenedor.

```dockerfile
FROM python:3.11-slim

RUN useradd --create-home appuser
WORKDIR /app
COPY --chown=appuser:appuser . .

USER appuser   # todas las instrucciones y el proceso final corren como este usuario, no como root

CMD ["python", "app.py"]
```

```bash
docker run --user 1000:1000 mi-imagen   # alternativa: forzar un UID/GID específico desde docker run
```

## `.dockerignore` como control de seguridad, no solo de tamaño

Además de reducir el tamaño de la imagen (ver [[02 - Dockerfile en Profundidad#`.dockerignore` — no meter basura a la imagen|Dockerfile en Profundidad]]), excluir `.env`, `.git/` y credenciales locales del contexto de build evita que terminen embebidas permanentemente en una capa de la imagen — una vez en una capa, esos archivos quedan recuperables aunque se borren en una instrucción posterior.

## Nunca hardcodear secretos en el Dockerfile

```dockerfile
# MAL: la contraseña queda visible en el historial de la imagen para siempre
ENV DATABASE_PASSWORD=clave123

# BIEN: se inyecta en tiempo de ejecución, no queda en ninguna capa de la imagen
# docker run -e DATABASE_PASSWORD=clave123 mi-imagen
```

```bash
docker history mi-imagen   # un secreto puesto con ENV o ARG sin cuidado puede ser visible aquí
```

Para builds que necesitan un secreto temporalmente (por ejemplo, un token para descargar un paquete privado) sin dejar rastro en ninguna capa, usar BuildKit secrets:

```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=github_token \
    TOKEN=$(cat /run/secrets/github_token) && pip install ...
```

```bash
docker build --secret id=github_token,src=./token.txt .
```

## Escanear imágenes en busca de vulnerabilidades conocidas

```bash
docker scout cves mi-imagen         # escaneo integrado de Docker
trivy image mi-imagen               # alternativa popular, ampliamente usada en pipelines de CI/CD
```

Integrar el escaneo como paso obligatorio en CI/CD (fallar el pipeline si se encuentran vulnerabilidades críticas) es la práctica estándar antes de publicar una imagen a un registro compartido.

## Limitar capacidades del kernel

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE mi-imagen
```

Por defecto, un contenedor recibe un conjunto amplio de capacidades del kernel de Linux (`--cap-drop=ALL` las retira todas; `--cap-add` reactiva solo las estrictamente necesarias) — el principio de menor privilegio aplicado a nivel de kernel.

## Imágenes base oficiales y verificadas

Preferir imágenes oficiales (`python`, `postgres`, `node` en Docker Hub, marcadas como "Official Image") o imágenes verificadas del propio proveedor cloud, en vez de imágenes de terceros no auditadas — una imagen base comprometida contamina automáticamente todo lo que se construye sobre ella.

## Ver también

- [[02 - Dockerfile en Profundidad]]
- [[04 - Gestión de Imágenes]]
- [[Kubernetes/10 - Seguridad y RBAC|Kubernetes — RBAC y seguridad]]
