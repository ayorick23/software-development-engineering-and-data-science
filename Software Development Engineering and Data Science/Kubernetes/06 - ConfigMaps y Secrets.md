---
tags: [kubernetes, k8s, mlops, orquestacion, cheat-sheet]
---

# 06 — ConfigMaps y Secrets

Empaquetar configuración (URLs, flags, umbrales de un modelo) o credenciales directamente dentro de una imagen de contenedor obliga a reconstruir y republicar la imagen cada vez que ese valor cambia — y, peor, en el caso de credenciales, las deja versionadas en el registro de imágenes. Kubernetes separa la **configuración** del **código empaquetado** con dos objetos: `ConfigMap` (datos no sensibles) y `Secret` (datos sensibles).

## ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-modelo-config
data:
  LOG_LEVEL: "INFO"
  MODEL_THRESHOLD: "0.75"
  config.yaml: |
    features:
      - edad
      - ingresos
      - historial_crediticio
    version: "2.1"
```

```bash
kubectl create configmap api-modelo-config --from-literal=LOG_LEVEL=INFO
kubectl create configmap api-modelo-config --from-file=config.yaml
kubectl get configmaps
kubectl describe configmap api-modelo-config
```

### Consumir un ConfigMap como variables de entorno

```yaml
containers:
  - name: api-modelo
    image: mi-registro/api-modelo:1.0
    envFrom:
      - configMapRef:
          name: api-modelo-config
    # o, campo por campo:
    env:
      - name: LOG_LEVEL
        valueFrom:
          configMapKeyRef:
            name: api-modelo-config
            key: LOG_LEVEL
```

### Consumir un ConfigMap como archivo montado (Volume)

```yaml
containers:
  - name: api-modelo
    image: mi-registro/api-modelo:1.0
    volumeMounts:
      - name: config-volume
        mountPath: /app/config
volumes:
  - name: config-volume
    configMap:
      name: api-modelo-config
```

Montar como Volume tiene una ventaja clave sobre las variables de entorno: si el ConfigMap se actualiza (`kubectl apply` con nuevos valores), el archivo montado **se actualiza automáticamente** dentro del Pod en corto tiempo (sin reiniciar el contenedor) — útil para reconfigurar un servicio en caliente. Las variables de entorno, en cambio, quedan fijas al valor que tenían cuando el contenedor arrancó.

## Secret

Estructuralmente igual a un ConfigMap, pero pensado para datos sensibles: contraseñas de base de datos, tokens de API, claves de un bucket de S3 donde vive un modelo (ver [[Machine Learning/46-Reproducibilidad-con-DVC|DVC]] para el mismo concepto aplicado a datos versionados).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  usuario: YWRtaW4=          # valores en base64, NO encriptados
  password: c3VwZXJzZWNyZXRv
```

```bash
# forma recomendada — kubectl codifica en base64 automáticamente
kubectl create secret generic db-credentials \
  --from-literal=usuario=admin \
  --from-literal=password=supersecreto

kubectl get secrets
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d   # decodificar para verificar
```

> **El malentendido más común: base64 no es encriptación.** Cualquiera con acceso de lectura a `etcd` o con permiso para hacer `kubectl get secret -o yaml` puede decodificar el valor trivialmente. Un `Secret` de Kubernetes por defecto solo aporta una capa de *separación* del resto de la configuración, no confidencialidad real. Para seguridad real:
> - Habilitar **encryption at rest** de `etcd` a nivel de clúster (configuración del control plane, no del Secret en sí).
> - Restringir con [[10 - Seguridad y RBAC|RBAC]] quién puede leer Secrets.
> - Usar un gestor de secretos externo (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault) integrado vía un operador como **External Secrets Operator**, que sincroniza el secreto real desde el servicio externo hacia un `Secret` de Kubernetes (o lo inyecta directamente), evitando que la credencial viva permanentemente en el YAML del clúster.

### Consumir un Secret

```yaml
containers:
  - name: api-modelo
    image: mi-registro/api-modelo:1.0
    env:
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: password
    volumeMounts:
      - name: creds
        mountPath: /etc/secrets
        readOnly: true
volumes:
  - name: creds
    secret:
      secretName: db-credentials
```

### `imagePullSecrets` — un tipo especial de Secret

Para que un nodo pueda descargar imágenes de un registro **privado** (no público como Docker Hub), el Pod necesita credenciales de ese registro:

```bash
kubectl create secret docker-registry mi-registro-secret \
  --docker-server=mi-registro.azurecr.io \
  --docker-username=usuario \
  --docker-password=password
```

```yaml
spec:
  imagePullSecrets:
    - name: mi-registro-secret
  containers:
    - name: api-modelo
      image: mi-registro.azurecr.io/api-modelo:1.0
```

## ConfigMaps y Secrets en un pipeline de ML

Un caso frecuente: la imagen del contenedor de servicio de modelo (ver [[11 - Kubernetes para Servir Modelos de ML]]) es genérica — no cambia entre entornos — pero necesita apuntar a distintas rutas de artefactos según el entorno (`staging` vs. `producción`), y necesita credenciales para el almacenamiento remoto de MLflow (ver `MLflow/03 - Tracking - Servidor, Backend Store y Artifact Store.md`) o de DVC. Separar esto en un ConfigMap (rutas, nombres) y un Secret (credenciales del artifact store) permite reusar exactamente la misma imagen en ambos entornos, cambiando solo la configuración externa.

## Ver también

- [[03 - Pods]]
- [[10 - Seguridad y RBAC]]
- [[11 - Kubernetes para Servir Modelos de ML]]
- `MLflow/03 - Tracking - Servidor, Backend Store y Artifact Store.md`
