---
tags: [dvc, mlops, data-registry, cheat-sheet]
---

# 07 — Data Registries, Import y Compartición entre Proyectos

> Continúa de [[06 - DVC Experiments - Iteración Rápida sin Comprometer Git]].

Hasta ahora, todo el cheat-sheet asumió un único repositorio con sus propios datos versionados. Este archivo cubre cómo **compartir** datos/modelos versionados con DVC entre proyectos distintos, o consumir un repositorio de datos como fuente externa.

## El patrón de "Data Registry" — un repo dedicado solo a versionar datos

Un **Data Registry** es simplemente un repositorio Git+DVC cuyo único propósito es versionar datasets compartidos por múltiples proyectos — no contiene código de entrenamiento, solo datos y su historial de versiones.

```
data-registry/           # repo dedicado, sin lógica de modelado
├── .dvc/
├── clientes/
│   └── historico_clientes.parquet.dvc
├── demanda/
│   └── demanda_nacional.parquet.dvc
└── README.md
```

Otros proyectos (el repo de entrenamiento, el repo de un dashboard) referencian versiones específicas de este registry sin necesitar clonarlo completo ni duplicar su historial.

## `dvc import` — traer un archivo versionado desde OTRO repo Git+DVC

```bash
dvc import https://github.com/empresa/data-registry.git demanda/demanda_nacional.parquet -o data/demanda_nacional.parquet
```

Esto crea un archivo `data/demanda_nacional.parquet.dvc` en el proyecto actual que **referencia** la versión específica del archivo en el repo origen — mantiene un vínculo vivo hacia esa fuente externa, a diferencia de simplemente copiar el archivo.

```bash
dvc import https://github.com/empresa/data-registry.git demanda/demanda_nacional.parquet \
  --rev v2.3.0 \
  -o data/demanda_nacional.parquet
```

`--rev` fija el import a un tag/commit/branch específico del repo origen — sin esto, apunta al branch por defecto, que puede cambiar con el tiempo.

## `dvc update` — actualizar un import a la versión más reciente del origen

```bash
dvc update data/demanda_nacional.parquet.dvc
```

Cuando el data registry publica una nueva versión del dataset, este comando actualiza el import local para apuntar a esa nueva versión — sin necesitar rehacer el `dvc import` desde cero.

## `dvc get` — descarga puntual, sin mantener un vínculo vivo

```bash
dvc get https://github.com/empresa/data-registry.git demanda/demanda_nacional.parquet -o data/demanda_nacional.parquet
```

Diferencia clave con `dvc import`: `dvc get` simplemente **copia** el archivo, sin crear un `.dvc` que mantenga la referencia al repo origen — apropiado para una descarga puntual de datos externos (por ejemplo, un dataset público de referencia) que no se espera actualizar posteriormente desde esa fuente.

```bash
dvc get https://github.com/empresa/data-registry.git demanda/ --rev v2.3.0 -o data/  # también funciona con carpetas
```

## `dvc.api` — leer datos versionados directamente desde Python, sin clonar el repo

```python
import dvc.api

with dvc.api.open(
    "demanda/demanda_nacional.parquet",
    repo="https://github.com/empresa/data-registry.git",
    rev="v2.3.0",
    mode="rb",
) as f:
    import pandas as pd
    df = pd.read_parquet(f)
```

```python
# Atajo directo para leer con pandas:
url = dvc.api.get_url(
    "demanda/demanda_nacional.parquet",
    repo="https://github.com/empresa/data-registry.git",
    rev="v2.3.0",
)
df = pd.read_parquet(url)
```

Útil dentro de notebooks de exploración o scripts de análisis que necesitan leer una versión específica de un dataset externo sin montar el flujo completo de `dvc import`/`dvc pull` en un proyecto que no necesita versionar ese dato localmente, solo consumirlo.

## Cuándo usar cada mecanismo — tabla de decisión

| Situación | Mecanismo |
|---|---|
| El proyecto necesita mantenerse sincronizado con actualizaciones futuras del dataset origen | `dvc import` + `dvc update` |
| Solo se necesita una copia puntual, sin seguimiento de actualizaciones | `dvc get` |
| Lectura exploratoria desde Python/notebook, sin versionar nada localmente | `dvc.api.open` / `dvc.api.get_url` |
| El dataset vive y se versiona en el mismo repo del proyecto | `dvc add` (ver [[02 - Versionado de Datos - Comandos Fundamentales]]) |

## Compartir un modelo entrenado como artefacto consumible

El mismo patrón de `dvc import`/`dvc get` aplica igual a modelos ya entrenados, no solo a datasets crudos — un repo de "model registry" basado en DVC puede coexistir con (o sustituir en organizaciones más simples) el Model Registry de MLflow (ver `MLflow/07 - Model Registry.md`):

```bash
dvc get https://github.com/empresa/model-registry.git modelos/demanda_v3.pkl -o models/modelo_actual.pkl
```

En la práctica, para modelos de producción con ciclo de vida formal (stages, aliases, trazabilidad de despliegue), el Model Registry de MLflow suele ser más apropiado que este patrón — `dvc import`/`get` de modelos es más común para compartir *checkpoints* intermedios entre proyectos de investigación, no para el ciclo de vida productivo completo.

## Ver también

- [[03 - Remotes y Almacenamiento en la Nube]]
- `MLflow/07 - Model Registry.md`
- [[09 - Integración con MLflow y el Ecosistema]]
