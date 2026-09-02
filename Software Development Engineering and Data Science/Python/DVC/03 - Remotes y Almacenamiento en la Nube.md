---
tags: [dvc, mlops, remotes, s3, azure, cheat-sheet]
---

# 03 — Remotes y Almacenamiento en la Nube

> Continúa de [[02 - Versionado de Datos - Comandos Fundamentales]].

Un **remote** en DVC es el almacenamiento compartido donde vive el contenido real de los datos versionados — el equivalente de `origin` en Git, pero para el cache de datos en vez de para el historial de código.

## Configurar un remote

```bash
dvc remote add -d storage s3://mi-bucket/dvc-storage
```

- `-d` (o `--default`): marca este remote como el default, usado automáticamente por `dvc push`/`pull` sin especificar `-r`.
- El nombre `storage` es arbitrario — se puede tener múltiples remotes con nombres distintos.

```bash
dvc remote list   # ver todos los remotes configurados
```

Esto se guarda en `.dvc/config` — versionado normalmente en Git, para que todo el equipo comparta la misma configuración de remote al clonar el repo.

## Amazon S3

```bash
dvc remote add -d storage s3://mi-bucket/dvc-storage
dvc remote modify storage region us-east-1

# Credenciales — vía variables de entorno estándar de AWS (recomendado, no hardcodear):
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."

# O explícitamente en la config (NO recomendado para credenciales sensibles en un repo compartido):
dvc remote modify storage access_key_id 'AKIA...' --local   # --local: va a .dvc/config.local, NO se versiona en Git
```

> `--local` es la forma correcta de guardar credenciales — escribe en `.dvc/config.local`, un archivo que DVC excluye automáticamente de Git por convención. Nunca poner credenciales directamente en `.dvc/config` sin `--local`.

## Azure Blob Storage

```bash
pip install "dvc[azure]"

dvc remote add -d storage azure://contenedor-ml/dvc-storage
dvc remote modify storage connection_string "DefaultEndpointsProtocol=https;AccountName=...;AccountKey=...;" --local

# Alternativa con account key por separado:
dvc remote modify storage account_name "miaccount" --local
dvc remote modify storage account_key "..." --local
```

## Google Cloud Storage

```bash
pip install "dvc[gs]"

dvc remote add -d storage gs://mi-bucket/dvc-storage
dvc remote modify storage credentialpath "/ruta/al/service-account.json" --local
```

## SSH — servidor propio, sin depender de un proveedor cloud

```bash
pip install "dvc[ssh]"

dvc remote add -d storage ssh://usuario@servidor.interno:/ruta/al/almacenamiento
dvc remote modify storage keyfile ~/.ssh/id_rsa
```

Útil en organizaciones con infraestructura on-premise donde no hay (o no se quiere depender de) un proveedor cloud — cualquier servidor accesible por SSH puede actuar como remote.

## Almacenamiento local / red compartida

```bash
dvc remote add -d storage /mnt/almacenamiento_compartido/dvc-storage
# o una ruta de red (SMB/NFS ya montada):
dvc remote add -d storage //servidor/carpeta_compartida/dvc-storage
```

La opción más simple para equipos pequeños con una carpeta de red ya existente — sin necesidad de configurar credenciales de un proveedor cloud.

## `dvc push` / `dvc pull` / `dvc fetch`

```bash
dvc push        # sube al remote TODO lo que está en el cache local pero no en el remote
dvc push -r storage_backup   # sube a un remote específico (si hay varios configurados)

dvc pull         # descarga del remote lo necesario para reconstruir el workspace actual

dvc fetch        # descarga al CACHE local, sin tocar el workspace (útil antes de un checkout específico)
dvc checkout     # aplica al workspace lo que ya está en cache (tras un fetch)
```

`dvc pull` es efectivamente `dvc fetch` + `dvc checkout` combinados — la forma más común de "traer los datos" tras clonar un repo o cambiar de branch.

```bash
git clone https://repo-url.git
cd repo
dvc pull   # trae todos los datos versionados, según el estado actual del branch
```

## Múltiples remotes — patrones de uso

```bash
dvc remote add s3_principal s3://bucket-principal/dvc
dvc remote add s3_backup s3://bucket-backup/dvc

dvc push -r s3_principal
dvc push -r s3_backup   # respaldo redundante en un segundo remote
```

Útil para estrategias de respaldo (push a dos ubicaciones) o para separar remotes por entorno (`s3_dev`, `s3_prod`) sin cambiar la configuración del proyecto entre ambientes, solo el remote objetivo del comando.

## `dvc gc` — limpieza del cache

```bash
dvc gc -w   # elimina del cache todo lo que NO esté referenciado en el workspace actual

dvc gc -a   # elimina del cache todo lo que NO esté referenciado en NINGÚN branch/tag local

dvc gc -c   # aplica la misma limpieza también al REMOTE, no solo al cache local — usar con precaución
```

> **Precaución con `dvc gc -c`**: elimina permanentemente del almacenamiento remoto cualquier versión de datos que ya no esté referenciada por ningún commit alcanzable — si algún compañero tiene un branch local sin pushear que referencia una versión de datos ahora "huérfana", esa versión se vuelve irrecuperable tras el `gc`. Ejecutar solo cuando se está seguro de que todo el historial relevante ya está reflejado en el repositorio remoto de Git.

## Verificar el estado de sincronización con el remote

```bash
dvc status -c   # ¿hay algo en el workspace/cache que falte subir al remote (cloud)?
```

## Ver también

- [[02 - Versionado de Datos - Comandos Fundamentales]]
- [[07 - Data Registries, Import y Compartición entre Proyectos]]
- `MLflow/03 - Tracking - Servidor, Backend Store y Artifact Store.md` (patrón similar de backend/artifact store)
