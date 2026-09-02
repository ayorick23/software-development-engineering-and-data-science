---
tags: [dvc, mlops, versionado, cheat-sheet]
---

# 02 — Versionado de Datos: Comandos Fundamentales

> Continúa de [[01 - Introducción y Arquitectura Interna]].

## `dvc add` — empezar a versionar un archivo o carpeta

```bash
dvc add data/demanda_historica.parquet
```

Efectos de este comando:
1. Mueve el archivo real al **cache** de DVC (`.dvc/cache/`).
2. Crea `demanda_historica.parquet.dvc` (un archivo YAML pequeño con el hash).
3. Agrega `demanda_historica.parquet` a un `.gitignore` automático en esa carpeta (para que Git nunca intente versionar el archivo pesado directamente).
4. Deja en el workspace un link al archivo real en cache — visualmente parece el mismo archivo, pero es una referencia.

```bash
git add data/demanda_historica.parquet.dvc data/.gitignore
git commit -m "Versiono el dataset de entrenamiento v1"
```

El paso de `git add`/`commit` es **manual y necesario** — DVC no hace commits de Git automáticamente, solo prepara los archivos `.dvc` para que Git los versiones.

### Versionar una carpeta completa

```bash
dvc add data/raw/   # versiona TODOS los archivos dentro de la carpeta como una unidad
```

Genera un único `data/raw.dvc` que referencia todo el contenido de la carpeta — útil cuando el dataset está compuesto por muchos archivos pequeños (imágenes, por ejemplo) en vez de un solo archivo grande.

## `dvc status` — qué cambió desde la última versión versionada

```bash
dvc status
```

```
data/demanda_historica.parquet.dvc:
    changed outs:
        modified:           data/demanda_historica.parquet
```

Compara el hash actual del archivo en el workspace contra el hash registrado en el `.dvc` — reporta si el archivo fue modificado, eliminado, o si el cache está desincronizado, análogo a `git status` pero para datos.

```bash
dvc status -c   # compara contra el REMOTO, no solo el cache local — ¿hace falta un dvc push?
```

## `dvc diff` — comparar entre dos versiones (commits/tags/branches)

```bash
dvc diff HEAD~1 HEAD
```

```
Added:
    data/nuevas_features.parquet

Modified:
    data/demanda_historica.parquet
```

Muestra qué archivos versionados por DVC cambiaron entre dos referencias de Git — útil para responder rápidamente "¿qué datos cambiaron entre esta versión del modelo y la anterior?" sin tener que inspeccionar manualmente los hashes.

## `dvc checkout` — sincronizar el workspace con lo que Git tiene registrado

```bash
git checkout v1.4.2   # cambia el código a un commit/tag específico
dvc checkout            # trae los datos EXACTOS que existían en ese commit
```

`git checkout` por sí solo solo actualiza los archivos `.dvc` (los punteros); `dvc checkout` es el paso que efectivamente reemplaza el contenido del workspace con la versión de datos correspondiente a esos punteros, tomándola del cache local (o descargándola del remote si no está en cache, ver [[03 - Remotes y Almacenamiento en la Nube]]).

## `dvc remove` — dejar de versionar un archivo

```bash
dvc remove data/demanda_historica.parquet.dvc   # elimina el .dvc y el archivo del workspace
dvc remove data/demanda_historica.parquet.dvc --outs   # NO elimina el archivo del workspace, solo el tracking
```

## `dvc move` — renombrar/mover un archivo versionado correctamente

```bash
dvc move data/demanda_historica.parquet data/demanda_rd_historica.parquet
```

Renombrar un archivo versionado con `mv` del sistema operativo rompe la referencia en el `.dvc` — `dvc move` actualiza tanto el archivo físico como su archivo `.dvc` correspondiente de forma consistente.

## `dvc unprotect` — hacer un archivo editable directamente

```bash
dvc unprotect data/demanda_historica.parquet
```

Como el archivo en el workspace suele ser un link al cache (de solo lectura, para evitar modificaciones accidentales que corrompan el cache), `dvc unprotect` lo convierte en una copia física editable — necesario antes de modificar manualmente un archivo versionado sin pasar por el pipeline que lo genera.

## `.dvcignore` — excluir archivos del tracking de una carpeta

```
# .dvcignore, sintaxis similar a .gitignore
*.tmp
__pycache__/
*.log
```

Cuando se versiona una carpeta completa (`dvc add data/raw/`), `.dvcignore` excluye patrones específicos de esa carpeta del versionado — útil para evitar arrastrar archivos temporales o de caché de otras herramientas que conviven en la misma carpeta.

## El archivo `.dvc` — anatomía completa

```yaml
outs:
  - md5: a1b2c3d4e5f6789...
    size: 524288000
    hash: md5
    path: demanda_historica.parquet
```

- `md5`: hash de contenido — la clave real que identifica esta versión exacta de los datos.
- `size`: tamaño en bytes, usado para verificaciones rápidas de integridad.
- `path`: nombre del archivo en el workspace, relativo a la ubicación del `.dvc`.

Este archivo es lo único que Git versiona directamente — es intencionalmente pequeño y legible como texto plano, lo que permite ver en un `git diff` exactamente cuándo cambió el hash de un dataset, sin necesitar herramientas especiales.

## Ver también

- [[01 - Introducción y Arquitectura Interna]]
- [[03 - Remotes y Almacenamiento en la Nube]]
- [[04 - DVC Pipelines - dvc.yaml y Reproducibilidad]]
