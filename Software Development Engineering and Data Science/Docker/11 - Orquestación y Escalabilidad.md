---
tags: [docker, orquestacion, swarm, cheat-sheet]
---

# 11 — Orquestación y Escalabilidad

> Para el sistema de orquestación que domina en producción real, ver [[Kubernetes/01 - Introducción y Arquitectura del Clúster|Kubernetes]] — este archivo cubre Docker Swarm principalmente como puente conceptual y para casos donde su simplicidad es una ventaja real.

## Por qué hace falta un orquestador

`docker run` y `docker compose` gestionan contenedores en **un solo host**. En producción real, la pregunta cambia: ¿en cuál de varios servidores corre cada contenedor? ¿qué pasa si un servidor se cae? ¿cómo se reparte el tráfico entre decenas de réplicas? ¿cómo se actualiza la aplicación sin downtime? Responder esto manualmente a escala es inviable — para eso existen los orquestadores.

## Docker Swarm — el orquestador nativo de Docker

Swarm convierte varios hosts Docker en un clúster único, con la misma sintaxis de Compose:

```bash
docker swarm init                    # convertir el host actual en el primer nodo manager
docker swarm join-token worker        # obtener el comando para que otros nodos se unan como workers
```

```bash
docker service create --name mi-app --replicas 3 -p 8080:80 mi-imagen
docker service scale mi-app=5         # escalar de 3 a 5 réplicas
docker service update --image mi-imagen:2.0 mi-app   # actualización sin downtime (rolling update)
```

```bash
docker stack deploy -c docker-compose.yml mi-stack   # desplegar un archivo Compose completo como stack en el clúster
```

## Swarm vs Kubernetes

| | Docker Swarm | Kubernetes |
|---|---|---|
| Curva de aprendizaje | Baja — misma sintaxis que Compose | Alta — conceptos propios (Pods, Deployments, Services) |
| Adopción en la industria | Minoritaria, en declive | Estándar dominante de facto |
| Flexibilidad/extensibilidad | Limitada | Muy alta (operadores, CRDs, ecosistema enorme) |
| Cuándo elegirlo | Equipos pequeños, clústers simples, se prioriza simplicidad sobre features | Casi cualquier despliegue serio en producción, especialmente con necesidades de ML (ver [[Kubernetes/11 - Kubernetes para Servir Modelos de ML]]) |

En la práctica actual, Kubernetes ha desplazado a Swarm como el estándar de la industria — Swarm sigue siendo válido para equipos que priorizan simplicidad sobre features avanzadas, pero la mayoría de nuevos proyectos en producción adoptan Kubernetes directamente.

## El puente conceptual hacia Kubernetes

| Concepto en Docker/Swarm | Equivalente en Kubernetes |
|---|---|
| Contenedor | Contenedor dentro de un [[Kubernetes/03 - Pods|Pod]] |
| `docker service` (réplicas) | [[Kubernetes/04 - Deployments y ReplicaSets|Deployment]] |
| Publicar un puerto | [[Kubernetes/05 - Services y Networking|Service]] |
| `docker network` | Red interna del clúster, gestionada por Kubernetes |
| `docker volume` | [[Kubernetes/07 - Volumes y Almacenamiento Persistente|PersistentVolume]] |

Quien ya domina Docker Compose/Swarm tiene la mayoría de los conceptos necesarios para entender Kubernetes — los nombres cambian, pero el problema que resuelven (múltiples contenedores, red compartida, almacenamiento persistente, escalado) es el mismo.

## Ver también

- [[08 - Docker Compose en Profundidad]]
- [[Kubernetes/01 - Introducción y Arquitectura del Clúster]]
