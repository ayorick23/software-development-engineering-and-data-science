---
tags: [docker, multi-stage, optimizacion, cheat-sheet]
---

# 03 — Multi-stage Builds y Optimización de Imágenes

## El problema: herramientas de build no deberían viajar a producción

Compilar una aplicación a menudo requiere herramientas (compiladores, dependencias de desarrollo) que **no se necesitan en tiempo de ejecución** — incluirlas en la imagen final la infla innecesariamente y amplía la superficie de ataque de seguridad.

## Multi-stage build: dos (o más) etapas en un solo Dockerfile

```dockerfile
# Etapa 1: build -- instala dependencias con las herramientas de compilación necesarias
FROM python:3.11 AS build
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Etapa 2: runtime -- imagen final, sin herramientas de compilación
FROM python:3.11-slim
WORKDIR /app
COPY --from=build /root/.local /root/.local   # copia SOLO el resultado, no las herramientas de build
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

`COPY --from=build` trae exclusivamente los artefactos ya compilados de la primera etapa; todo el resto de la etapa `build` (compiladores, caché de pip, archivos temporales) se descarta al finalizar el build — nunca llega a la imagen final.

## Por qué la imagen final debe ser mínima

| Razón | Consecuencia de una imagen inflada |
|---|---|
| Velocidad de despliegue | Imágenes grandes tardan más en subir/bajar de un registro, lentificando CI/CD y escalado |
| Superficie de ataque | Cada paquete adicional (compiladores, herramientas de sistema) es una vulnerabilidad potencial más |
| Costo de almacenamiento | Registros privados cobran por espacio; cada capa extra se acumula |

## Elegir la imagen base correcta

```dockerfile
FROM python:3.11              # ~1 GB -- incluye herramientas de compilación completas
FROM python:3.11-slim         # ~150 MB -- Debian mínimo, sin herramientas de compilación
FROM python:3.11-alpine       # ~50 MB -- Alpine Linux, aún más pequeña, pero usa musl libc en vez de glibc
```

`alpine` es la más pequeña, pero su libc distinta (`musl` en vez de `glibc`) puede causar incompatibilidades sutiles con paquetes de Python que incluyen extensiones compiladas en C (numpy, pandas) — `slim` suele ser el mejor balance para proyectos de ciencia de datos.

## Reducir capas combinando instrucciones `RUN`

```dockerfile
# Cada RUN crea una capa -- 3 capas, y la caché de apt queda "atrapada" en una capa intermedia
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# Una sola capa, y la limpieza SÍ reduce el tamaño porque ocurre en el mismo RUN
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```

Si la limpieza (`rm -rf`) ocurre en un `RUN` separado, los archivos "eliminados" siguen ocupando espacio en la capa anterior — Docker no puede recuperar espacio de capas ya creadas, solo de lo que nunca llegó a escribirse en una capa final.

## Verificar el tamaño y composición de una imagen

```bash
docker images mi-imagen              # tamaño total
docker history mi-imagen             # tamaño de cada capa individual
```

```bash
# dive: herramienta externa para explorar visualmente qué ocupa espacio en cada capa
dive mi-imagen
```

## Ver también

- [[02 - Dockerfile en Profundidad]]
- [[04 - Gestión de Imágenes]]
- [[10 - Docker para Ciencia de Datos y ML]]
