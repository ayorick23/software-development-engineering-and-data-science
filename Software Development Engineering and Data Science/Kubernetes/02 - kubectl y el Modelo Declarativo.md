---
tags: [kubernetes, k8s, mlops, orquestacion, cheat-sheet]
---

# 02 — kubectl y el Modelo Declarativo

`kubectl` es la CLI oficial para interactuar con el `kube-apiserver` de cualquier clúster (ver [[01 - Introducción y Arquitectura del Clúster]]). Todo lo que se puede hacer con Kubernetes pasa por esta herramienta (o por llamadas directas a la misma API REST que ella usa).

## Instalación y contexto

```bash
# instalación (varía por SO — ver docs oficiales)
brew install kubectl        # macOS
choco install kubernetes-cli  # Windows

kubectl version --client
```

Un mismo `kubectl` puede hablar con **varios clústeres distintos** (local, staging, producción). Esa configuración vive en `~/.kube/config`, organizada en **contexts**:

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <nombre-del-contexto>

# ver a qué clúster/namespace apunta el contexto actual
kubectl config view --minify
```

> **Precaución operativa:** confundir el contexto activo es la causa número uno de "apliqué un cambio al clúster equivocado". Antes de correr algo destructivo en un clúster real, siempre `kubectl config current-context` primero.

## Namespaces — dividir un clúster en particiones lógicas

Un **namespace** aísla lógicamente un grupo de recursos dentro del mismo clúster físico (equipos, entornos, proyectos).

```bash
kubectl get namespaces
kubectl create namespace ml-staging
kubectl config set-context --current --namespace=ml-staging   # cambiar el namespace por defecto del contexto actual
```

La mayoría de los comandos aceptan `-n <namespace>` explícitamente; sin él, se usa el namespace por defecto del contexto (`default` si no se configuró otro).

## El modelo declarativo — manifiestos YAML

En vez de comandos imperativos (`kubectl run ...`), el patrón estándar en producción es escribir el **estado deseado** en un archivo YAML y aplicarlo:

```yaml
# pod-ejemplo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-pod
  labels:
    app: demo
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
```

Todo manifiesto de Kubernetes comparte esta forma: `apiVersion` (qué versión de la API define este tipo de recurso), `kind` (el tipo de recurso — Pod, Deployment, Service, etc.), `metadata` (nombre, namespace, labels, annotations) y `spec` (el estado deseado específico del recurso).

```bash
kubectl apply -f pod-ejemplo.yaml
```

## `apply` vs `create` vs `run` — por qué `apply` es el estándar

| Comando | Comportamiento |
|---|---|
| `kubectl create -f archivo.yaml` | Crea el recurso; **falla** si ya existe |
| `kubectl apply -f archivo.yaml` | Crea el recurso si no existe, o **actualiza** los campos que cambiaron si ya existe (calcula un diff) |
| `kubectl run nombre --image=x` | Atajo imperativo para crear un Pod rápido, sin YAML — útil solo para pruebas rápidas |

`apply` es idempotente: correrlo dos veces con el mismo archivo no tiene efecto la segunda vez, y es seguro incluirlo en pipelines de CI/CD (ver [[13 - Managed Kubernetes, CI-CD y Buenas Prácticas]]) porque siempre converge al mismo estado sin importar el estado previo.

## Comandos esenciales de lectura

```bash
# listar recursos
kubectl get pods
kubectl get pods -o wide          # con más columnas (IP, nodo)
kubectl get pods --all-namespaces # -A también funciona
kubectl get deployments,services  # varios tipos a la vez

# detalle completo de un recurso, incluyendo eventos recientes
kubectl describe pod mi-pod

# ver el YAML real que Kubernetes tiene almacenado (incluye campos generados automáticamente)
kubectl get pod mi-pod -o yaml

# logs del/de los contenedor(es) de un Pod
kubectl logs mi-pod
kubectl logs mi-pod -c nombre-contenedor   # si el Pod tiene varios contenedores
kubectl logs -f mi-pod                     # seguir en tiempo real (follow)
kubectl logs --previous mi-pod             # logs del contenedor anterior, si reinició

# ejecutar un comando dentro de un contenedor ya corriendo
kubectl exec -it mi-pod -- /bin/bash

# reenviar un puerto local al Pod, para debug sin exponerlo públicamente
kubectl port-forward mi-pod 8080:80
```

## Modificar y eliminar recursos

```bash
kubectl delete -f pod-ejemplo.yaml
kubectl delete pod mi-pod

# editar un recurso en vivo con el editor por defecto (no recomendado para cambios permanentes — usar el YAML como fuente de verdad)
kubectl edit deployment mi-app

# parchear un campo específico sin reescribir todo el YAML
kubectl patch deployment mi-app -p '{"spec":{"replicas":5}}'

# forzar el reinicio de todos los Pods de un Deployment (útil tras cambiar un ConfigMap sin cambiar la imagen)
kubectl rollout restart deployment mi-app
```

## `kubectl explain` — documentación embebida

```bash
kubectl explain pod.spec.containers
kubectl explain deployment.spec.strategy --recursive
```

Cuando no se recuerda un campo exacto del YAML, `explain` documenta cada campo del recurso directamente desde el clúster (respeta la versión de API real que corre ese clúster), sin salir a buscar en la documentación web.

## Ver también

- [[01 - Introducción y Arquitectura del Clúster]]
- [[03 - Pods]]
- [[13 - Managed Kubernetes, CI-CD y Buenas Prácticas]]
