---
tags: [kubernetes, k8s, mlops, orquestacion, cheat-sheet]
---

# 07 — Volumes y Almacenamiento Persistente

Por diseño, el sistema de archivos de un contenedor es **efímero**: si el contenedor se reinicia o el Pod se reemplaza, todo lo escrito en su filesystem local se pierde. Para datos que deben sobrevivir a un contenedor individual — o compartirse entre los contenedores de un mismo [[03 - Pods|Pod]] — Kubernetes ofrece **Volumes**.

## `emptyDir` — almacenamiento efímero compartido dentro de un Pod

El tipo de Volume más simple: se crea vacío cuando el Pod arranca, vive mientras el Pod exista (sobrevive a reinicios de contenedores individuales dentro del Pod, **no** a la eliminación del Pod), y se usa típicamente para compartir archivos entre los contenedores de un mismo Pod (ver el patrón sidecar en [[03 - Pods#Patrón multi-contenedor: sidecar]]).

```yaml
volumes:
  - name: cache-temporal
    emptyDir: {}
    # emptyDir: { medium: Memory }   # variante: respaldado en RAM (tmpfs), muy rápido pero limitado por la memoria del nodo
```

## `hostPath` — montar un directorio del nodo (usar con cautela)

```yaml
volumes:
  - name: datos-nodo
    hostPath:
      path: /datos/compartidos
      type: Directory
```

Monta una ruta del **sistema de archivos del nodo** directamente en el Pod. Es frágil en producción: ata el Pod a los datos de un nodo físico específico (si el Pod se reprograma en otro nodo, esos datos no están ahí) y es un riesgo de seguridad si no se controla con cuidado. Se usa principalmente para casos de infraestructura (agentes de monitoreo que necesitan leer `/var/log` del nodo — ver [[12 - Observabilidad en Kubernetes]]), no para datos de aplicación.

## PersistentVolume (PV) y PersistentVolumeClaim (PVC) — el patrón real para datos que deben persistir

Para almacenamiento que debe sobrevivir a la eliminación del Pod **y** ser independiente del nodo físico, Kubernetes separa el "qué almacenamiento existe" (administrado por un operador de infraestructura) de "qué necesita esta aplicación" (declarado por quien despliega la app):

```mermaid
flowchart LR
    PVC["PersistentVolumeClaim\n(\"necesito 10Gi, ReadWriteOnce\")"] -->|se enlaza a| PV["PersistentVolume\n(almacenamiento real: disco EBS, Azure Disk, NFS...)"]
    SC["StorageClass\n(plantilla de aprovisionamiento)"] -->|provisiona dinámicamente| PV
    Pod --> PVC
```

- **PersistentVolume (PV):** representa una porción de almacenamiento real (un disco de EBS, un share de NFS, un disco de Azure) — normalmente creado automáticamente, no a mano.
- **PersistentVolumeClaim (PVC):** una **solicitud** de almacenamiento hecha por una aplicación ("necesito 10Gi con acceso de lectura-escritura desde un solo nodo"), sin conocer los detalles de infraestructura por debajo.
- **StorageClass:** define **cómo** aprovisionar un PV dinámicamente cuando llega un PVC — qué tipo de disco, qué proveedor, qué política de reclamo.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: modelos-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: gp3   # ej. en EKS; varía según el proveedor/clúster
```

```yaml
containers:
  - name: api-modelo
    image: mi-registro/api-modelo:1.0
    volumeMounts:
      - name: almacen-modelos
        mountPath: /modelos
volumes:
  - name: almacen-modelos
    persistentVolumeClaim:
      claimName: modelos-pvc
```

```bash
kubectl get pv
kubectl get pvc
kubectl describe pvc modelos-pvc
```

Cuando se crea el PVC, Kubernetes (vía el **provisioner** de la `StorageClass`) crea automáticamente el PV que lo satisface — esto se llama **aprovisionamiento dinámico**, y es el flujo estándar en clústeres gestionados (EKS/GKE/AKS ya traen StorageClasses preconfiguradas).

## Access Modes

| Modo | Significado |
|---|---|
| **ReadWriteOnce (RWO)** | Un solo nodo puede montarlo en lectura-escritura simultáneamente (el más común — discos de bloque tipo EBS/Azure Disk) |
| **ReadOnlyMany (ROX)** | Muchos nodos pueden montarlo, todos en solo lectura |
| **ReadWriteMany (RWX)** | Muchos nodos pueden montarlo en lectura-escritura simultáneamente (requiere un backend tipo NFS/EFS/Azure Files — no todos los discos de bloque lo soportan) |

Esta distinción importa directamente para servir modelos de ML a escala (ver [[11 - Kubernetes para Servir Modelos de ML]]): si varias réplicas de un Pod de inferencia en **distintos nodos** necesitan leer el mismo artefacto de modelo desde un volumen compartido, se necesita `ReadWriteMany` (ej. EFS en AWS), no `ReadWriteOnce` — de lo contrario, solo el Pod en el nodo "dueño" del volumen podrá montarlo.

## Dónde vive el peso de un modelo en producción — el patrón recomendado

En la mayoría de los despliegues de ML no se monta el modelo como un PVC persistente separado; los dos patrones más comunes son:

1. **Hornear el modelo dentro de la imagen** en tiempo de build (simple, inmutable, pero requiere reconstruir/republicar la imagen en cada actualización de modelo — encaja con el flujo de [[15 - Despliegue y Producción|FastAPI/Despliegue]]).
2. **Descargar el modelo al arrancar** desde un artifact store externo ([[07 - Model Registry|MLflow Model Registry]], S3, un bucket de GCS) usando un **init container** — un contenedor que corre y termina *antes* que el contenedor principal, dejando el modelo ya descargado en un `emptyDir` compartido:

```yaml
spec:
  initContainers:
    - name: descargar-modelo
      image: mi-registro/mlflow-downloader:1.0
      command: ["python", "download_model.py", "--stage=Production"]
      volumeMounts:
        - name: modelo-volume
          mountPath: /modelos
  containers:
    - name: api-modelo
      image: mi-registro/api-modelo:1.0
      volumeMounts:
        - name: modelo-volume
          mountPath: /modelos
  volumes:
    - name: modelo-volume
      emptyDir: {}
```

Este patrón desacopla la imagen de la aplicación (que rara vez cambia) del artefacto del modelo (que cambia frecuentemente), sin necesitar un PVC persistente: el `initContainer` descarga la versión correcta del modelo en cada arranque del Pod.

## Ver también

- [[03 - Pods]]
- [[06 - ConfigMaps y Secrets]]
- [[11 - Kubernetes para Servir Modelos de ML]]
- `MLflow/07 - Model Registry.md`
