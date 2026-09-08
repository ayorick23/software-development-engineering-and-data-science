---
tags: [docker, dockerfile, cheat-sheet]
---

# 02 — Dockerfile en Profundidad

> Para las instrucciones específicas de multi-stage builds, ver [[03 - Multi-stage Builds y Optimización de Imágenes]]. Para el contexto de por qué el orden de las instrucciones importa en un pipeline de ML, ver [[Machine Learning/47-Docker-en-Profundidad-para-ML#7. Capas de Docker y caché de build — por qué el ORDEN de las instrucciones importa|Machine Learning/47]].

## Un Dockerfile mínimo, instrucción por instrucción

```dockerfile
FROM python:3.11-slim          # imagen base de partida
WORKDIR /app                   # directorio de trabajo dentro del contenedor
COPY requirements.txt .        # copiar solo este archivo primero (ver capas, más abajo)
RUN pip install --no-cache-dir -r requirements.txt
COPY . .                       # copiar el resto del código
EXPOSE 8000                    # documenta el puerto que la app usará (no lo publica por sí solo)
CMD ["python", "app.py"]       # comando por defecto al arrancar el contenedor
```

## Las instrucciones principales

| Instrucción | Qué hace |
|---|---|
| `FROM` | Define la imagen base. Siempre la primera instrucción (salvo `ARG` antes de `FROM`). |
| `WORKDIR` | Establece el directorio de trabajo; crea el directorio si no existe. |
| `COPY` | Copia archivos del host a la imagen. |
| `ADD` | Como `COPY`, pero además descomprime `.tar` automáticamente y acepta URLs — usar `COPY` salvo que se necesite explícitamente esto. |
| `RUN` | Ejecuta un comando **durante el build** (instalar paquetes, compilar) — crea una nueva capa. |
| `CMD` | Comando por defecto al iniciar el contenedor; se puede sobreescribir con `docker run <imagen> <otro comando>`. |
| `ENTRYPOINT` | Comando fijo que siempre se ejecuta; `CMD` pasa a ser sus argumentos por defecto (no se sobreescribe tan fácilmente). |
| `EXPOSE` | Documentación de qué puerto usa la app — no publica el puerto por sí solo (eso lo hace `docker run -p`). |
| `ENV` | Define variables de entorno disponibles dentro del contenedor. |
| `ARG` | Variable disponible solo durante el *build*, no en tiempo de ejecución. |
| `USER` | Cambia el usuario con el que corren las siguientes instrucciones y el contenedor final (ver [[09 - Seguridad]]). |
| `VOLUME` | Declara un punto de montaje pensado para datos persistentes (ver [[07 - Volúmenes y Persistencia de Datos]]). |

## `CMD` vs `ENTRYPOINT` — la diferencia que confunde a casi todos

```dockerfile
# Con CMD: se sobreescribe fácilmente
CMD ["python", "app.py"]
# docker run mi-imagen echo "hola"  -> ejecuta "echo hola" en vez de "python app.py"

# Con ENTRYPOINT: el comando es fijo, los argumentos de `docker run` se le añaden
ENTRYPOINT ["python", "app.py"]
# docker run mi-imagen --puerto 9000  -> ejecuta "python app.py --puerto 9000"

# Combinados: ENTRYPOINT fijo el binario, CMD da argumentos por defecto sobreescribibles
ENTRYPOINT ["python", "app.py"]
CMD ["--puerto", "8000"]
```

## Capas de Docker y caché de build — por qué el orden importa

Cada instrucción (`RUN`, `COPY`, `ADD`) crea una **capa** inmutable en la imagen. Docker cachea cada capa: si una instrucción y todo lo anterior a ella no cambiaron, reutiliza la capa cacheada en vez de reejecutarla.

```dockerfile
# MAL orden: copiar el código ANTES de instalar dependencias
COPY . .
RUN pip install -r requirements.txt
# cada cambio de código invalida la caché de "pip install" -- reinstala TODO en cada build

# BUEN orden: dependencias primero, código después
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
# cambiar el código NO invalida la caché de "pip install" -- solo reconstruye la última capa
```

Regla práctica: ordenar las instrucciones de la que cambia **menos** frecuentemente a la que cambia **más** frecuentemente. Las dependencias del proyecto cambian rara vez; el código fuente cambia en cada commit.

## `.dockerignore` — no meter basura a la imagen

Análogo a `.gitignore`: excluye archivos del contexto de build antes de que lleguen a la imagen.

```
# .dockerignore
__pycache__/
*.pyc
.venv/
.git/
.env
tests/
*.md
```

Sin `.dockerignore`, `COPY . .` puede arrastrar archivos pesados o sensibles (entornos virtuales, credenciales locales, historial de git) a la imagen final — inflando su tamaño y potencialmente filtrando secretos.

## Variables de entorno y argumentos de build

```dockerfile
ARG VERSION_PYTHON=3.11
FROM python:${VERSION_PYTHON}-slim

ENV APP_ENV=production
```

```bash
docker build --build-arg VERSION_PYTHON=3.12 -t mi-imagen .
```

## Ver también

- [[03 - Multi-stage Builds y Optimización de Imágenes]]
- [[04 - Gestión de Imágenes]]
- [[09 - Seguridad]]
