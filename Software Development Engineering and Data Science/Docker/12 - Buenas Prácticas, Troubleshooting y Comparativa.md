---
tags: [docker, buenas-practicas, troubleshooting, cheat-sheet]
---

# 12 — Buenas Prácticas, Troubleshooting y Comparativa

## Checklist de buenas prácticas

- Ordenar el Dockerfile de lo que cambia menos a lo que cambia más ([[02 - Dockerfile en Profundidad#Capas de Docker y caché de build — por qué el orden importa|caché de capas]]).
- Usar multi-stage builds para que herramientas de compilación no lleguen a la imagen final ([[03 - Multi-stage Builds y Optimización de Imágenes]]).
- Correr como usuario no-root ([[09 - Seguridad#Usuario no-root dentro del contenedor|Seguridad]]).
- Nunca usar `latest` como único tag en producción ([[04 - Gestión de Imágenes#Etiquetado (tags)|Gestión de Imágenes]]).
- `.dockerignore` siempre presente, incluso en proyectos pequeños.
- Establecer límites de recursos (`--memory`, `--cpus`) explícitamente en producción.
- Un proceso principal por contenedor — no meter una aplicación completa multi-servicio en un solo contenedor; usar Compose/Kubernetes para orquestar varios contenedores especializados.
- Los datos van en volúmenes, nunca copiados dentro de la imagen ([[07 - Volúmenes y Persistencia de Datos]]).

## Troubleshooting: problemas comunes y su diagnóstico

| Síntoma | Causa probable | Cómo verificar |
|---|---|---|
| El contenedor se detiene inmediatamente después de `docker run` | El proceso principal (`CMD`/`ENTRYPOINT`) terminó (posiblemente con error) | `docker logs <contenedor>` |
| `docker build` no refleja un cambio de código reciente | Caché de capas usando una versión antigua | `docker build --no-cache` para forzar reconstrucción completa |
| No se puede conectar a un servicio por su nombre desde otro contenedor | Los contenedores no están en la misma red | `docker network inspect <red>` — verificar que ambos aparezcan conectados |
| "Port already in use" al hacer `docker run -p` | Otro proceso (o contenedor) ya usa ese puerto del host | `docker ps` para ver contenedores activos; en Linux, `lsof -i :<puerto>` para procesos del host |
| El contenedor consume toda la memoria disponible | Sin límites de recursos establecidos | Agregar `--memory` y revisar con `docker stats` |
| Cambios de código en desarrollo no se reflejan dentro del contenedor | Falta un bind mount, o la imagen fue construida con `COPY` en vez de montarse como volumen | Verificar `-v $(pwd):/app` en el `docker run`/Compose |
| Permisos denegados al escribir en un volumen | El usuario dentro del contenedor no coincide con el dueño del volumen en el host | `docker exec` y revisar `id` dentro del contenedor vs. permisos del volumen |

## Comandos de diagnóstico esenciales

```bash
docker logs --tail 100 -f mi-contenedor    # últimas 100 líneas, siguiendo en tiempo real
docker exec -it mi-contenedor sh           # entrar a inspeccionar (sh si la imagen no tiene bash, como alpine)
docker inspect mi-contenedor               # toda la configuración: red, volúmenes, variables, restart policy
docker events                              # stream de eventos del daemon en tiempo real (útil para depurar arranques/caídas)
docker system df                           # espacio ocupado por imágenes/contenedores/volúmenes
```

## Docker vs Podman

| | Docker | Podman |
|---|---|---|
| Arquitectura | Daemon centralizado (`dockerd`) corriendo como root | Sin daemon (*daemonless*), cada contenedor es un proceso hijo directo |
| Rootless por defecto | Requiere configuración adicional | Diseñado para correr sin privilegios de root desde el inicio |
| Compatibilidad de CLI | — | Prácticamente un reemplazo directo (`alias docker=podman` funciona en la mayoría de casos) |
| Adopción | Estándar de facto, mayor ecosistema y documentación | Creciendo, preferido en contextos donde el modelo daemon-root es una preocupación de seguridad |

## Docker vs Máquinas Virtuales — resumen de cuándo usar cada uno

Retomando la comparación de [[01 - Introducción y Arquitectura#Contenedores vs. Máquinas Virtuales (VMs)|el archivo introductorio]]: usar contenedores cuando se necesita aislar aplicaciones del mismo sistema operativo con máximo rendimiento y densidad; usar VMs cuando se necesita aislamiento total a nivel de kernel (multi-tenancy con clientes no confiables, o ejecutar un sistema operativo distinto al del host).

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[09 - Seguridad]]
- [[Kubernetes/13 - Managed Kubernetes, CI-CD y Buenas Prácticas]]
