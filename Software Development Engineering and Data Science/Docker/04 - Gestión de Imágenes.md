---
tags: [docker, imagenes, registries, cheat-sheet]
---

# 04 — Gestión de Imágenes

## Construir una imagen

```bash
docker build -t mi-app:1.0 .          # -t: nombre:tag; "." el contexto de build (directorio con el Dockerfile)
docker build -t mi-app:1.0 -f Dockerfile.prod .   # usar un Dockerfile con otro nombre
```

## Etiquetado (tags)

```bash
docker tag mi-app:1.0 mi-app:latest              # una misma imagen puede tener varios tags
docker tag mi-app:1.0 usuario/mi-app:1.0         # renombrar para subir a un registro
```

`latest` no significa "la versión más reciente automáticamente" — es solo un tag como cualquier otro, que hay que asignar explícitamente. Depender de `latest` en producción es una mala práctica: no hay forma de saber qué versión exacta se está ejecutando ni de revertir a una anterior con certeza.

## Registros: Docker Hub y alternativas privadas

```bash
docker login                          # autenticarse en Docker Hub (o -u/-p para otro registro)
docker push usuario/mi-app:1.0        # subir la imagen
docker pull usuario/mi-app:1.0        # descargar la imagen
```

| Registro | Cuándo usarlo |
|---|---|
| Docker Hub | Público, gratuito para imágenes públicas; el más usado para imágenes base oficiales |
| Amazon ECR / Google GCR / Azure ACR | Registros privados integrados con el proveedor cloud correspondiente |
| Harbor / GitLab Registry | Registros privados auto-hospedados, comunes en empresas con infraestructura propia |

## Inspeccionar una imagen

```bash
docker inspect mi-app:1.0             # metadatos completos en JSON: capas, variables de entorno, comando por defecto
docker history mi-app:1.0             # historial de capas y su tamaño individual
```

## Exportar e importar imágenes sin un registro

Útil para mover una imagen entre máquinas sin acceso a un registro compartido (por ejemplo, un entorno air-gapped):

```bash
docker save -o mi-app.tar mi-app:1.0   # exportar a un archivo .tar
docker load -i mi-app.tar              # importar desde el archivo
```

## Limpieza de imágenes

```bash
docker images -f "dangling=true"       # imágenes "huérfanas" (sin tag, resultado de builds intermedios)
docker image prune                     # eliminar imágenes huérfanas
docker image prune -a                  # eliminar TODAS las imágenes no usadas por un contenedor activo
docker system df                       # ver cuánto espacio ocupan imágenes, contenedores y volúmenes
```

## Escaneo de vulnerabilidades

```bash
docker scout cveports mi-app:1.0       # Docker Scout: escanea vulnerabilidades conocidas en las capas de la imagen
```

Ver [[09 - Seguridad]] para el flujo completo de escaneo como parte de un pipeline de CI/CD.

## Ver también

- [[02 - Dockerfile en Profundidad]]
- [[03 - Multi-stage Builds y Optimización de Imágenes]]
- [[09 - Seguridad]]
