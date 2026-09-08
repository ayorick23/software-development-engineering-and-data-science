---
tags: [docker, redes, cheat-sheet]
---

# 06 — Redes en Docker

## Por qué Docker necesita su propia capa de red

Por defecto, cada contenedor tiene su propia interfaz de red aislada (namespace de red, ver [[01 - Introducción y Arquitectura#Los mecanismos del kernel que hacen posible el aislamiento|namespaces]]) — dos contenedores no se ven entre sí a menos que se conecten explícitamente a la misma red, y no son accesibles desde fuera del host a menos que se publique un puerto.

## Los drivers de red

```bash
docker network ls   # listar redes disponibles
```

| Driver | Uso |
|---|---|
| `bridge` (por defecto) | Red privada aislada en el host; contenedores en la misma red bridge se ven entre sí por nombre |
| `host` | El contenedor comparte directamente la red del host, sin aislamiento — máximo rendimiento, sin aislamiento de puertos |
| `none` | Sin red — aislamiento total, para tareas que no necesitan conectividad |
| `overlay` | Red entre múltiples hosts/nodos — necesaria para [[11 - Orquestación y Escalabilidad|Swarm o Kubernetes]] |

## Publicar puertos: exponer un contenedor al host

```bash
docker run -p 8080:80 mi-imagen        # puerto 80 del contenedor accesible en localhost:8080 del host
docker run -p 127.0.0.1:8080:80 mi-imagen   # publicar solo en localhost, no en todas las interfaces
docker run -P mi-imagen                 # -P mayúscula: publica todos los puertos EXPOSE en puertos aleatorios del host
```

## Crear una red personalizada y comunicación por nombre

```bash
docker network create mi-red

docker run -d --name base-datos --network mi-red postgres
docker run -d --name backend --network mi-red mi-app
```

Dentro de `mi-red`, el contenedor `backend` puede conectarse a la base de datos usando el nombre `base-datos` como si fuera un hostname — Docker resuelve automáticamente el nombre del contenedor a su IP interna mediante un DNS embebido. Esto es lo que hace posible que un archivo de configuración diga `DATABASE_HOST=base-datos` en vez de una IP fija.

```python
# dentro del contenedor "backend"
conexion = f"postgresql://usuario:clave@base-datos:5432/mi_db"   # "base-datos" resuelve por DNS interno de Docker
```

## Inspeccionar una red

```bash
docker network inspect mi-red   # qué contenedores están conectados, sus IPs internas
docker inspect mi-contenedor --format '{{.NetworkSettings.IPAddress}}'   # IP de un contenedor específico
```

## Conectar un contenedor ya corriendo a otra red

```bash
docker network connect mi-red mi-contenedor
docker network disconnect mi-red mi-contenedor
```

## Ver también

- [[05 - Ciclo de Vida de Contenedores]]
- [[08 - Docker Compose en Profundidad]] — Compose crea automáticamente una red por proyecto
- [[11 - Orquestación y Escalabilidad]]
