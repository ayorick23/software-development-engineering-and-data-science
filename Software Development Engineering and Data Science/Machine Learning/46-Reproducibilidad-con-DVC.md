---
tags: [mlops, dvc, reproducibilidad, versionado]
---

# 46 — Reproducibilidad con DVC (Data Version Control)

> Nota del mentor: Git versiona tu código perfectamente, pero Git es terrible versionando datos y modelos — archivos binarios grandes rompen el rendimiento de git, y no hay forma nativa de decir "este modelo se entrenó exactamente con esta versión de estos datos". DVC resuelve exactamente ese hueco, y es la pieza que le da sentido completo a la trazabilidad que ya empezaste con MLflow.

## 1. El problema que resuelve — Git no fue diseñado para datos

```bash
git add demanda_historica.csv  # 500 MB — git se vuelve lentísimo, el repo crece sin control
git commit -m "actualizo datos"
```

Git guarda el historial completo de cada versión de cada archivo — con un CSV de 500MB que cambia semanalmente, en un año tienes un repositorio de gigabytes solo de datos históricos, haciendo `git clone` y `git log` insoportablemente lentos para todo el equipo, incluso para quien solo necesita el código.

## 2. Cómo funciona DVC — punteros ligeros en Git, datos reales en otro lado

```bash
dvc init
dvc add data/demanda_historica.parquet
git add data/demanda_historica.parquet.dvc data/.gitignore
git commit -m "Versiono el dataset de entrenamiento v1"
```

`dvc add` no mete el archivo pesado a Git — genera un archivo pequeño `.dvc` con un hash del contenido (similar a cómo Git ya versiona código, pero para datos), y el archivo real se guarda en un **almacenamiento remoto** configurado aparte (Azure Blob Storage, S3, o incluso una carpeta de red compartida).

```bash
dvc remote add -d storage azure://contenedor-ml/dvc-storage
dvc push   # sube los datos reales al remoto
dvc pull   # descarga los datos reales — cualquier compañero con el repo clonado puede hacerlo
```

El resultado: Git se mantiene ligero y rápido (solo versiona los punteros `.dvc`), mientras el histórico completo de versiones de tus datos vive en almacenamiento optimizado para archivos grandes.

## 3. Versionar el dataset exacto de cada modelo entrenado

```bash
git checkout v1.4.2  # vuelve al commit exacto de una versión del código
dvc checkout          # descarga EXACTAMENTE los datos que existían en ese commit
```

Esto es lo que resuelve la pregunta que probablemente ya te ha hecho tu jefa alguna vez: "¿con qué datos exactos se entrenó el modelo que está en producción ahora mismo?". Sin DVC, la respuesta depende de la memoria de alguien o de archivos sueltos con nombres como `datos_final_v2_REAL.csv`. Con DVC, es un `git checkout` + `dvc checkout` reproducible, sin ambigüedad.

## 4. DVC Pipelines — reproducibilidad del proceso completo, no solo de los datos

```yaml
# dvc.yaml
stages:
  preparar_datos:
    cmd: python scripts/preparar_datos.py
    deps:
      - scripts/preparar_datos.py
      - data/raw/demanda_historica.csv
    outs:
      - data/processed/features.parquet

  entrenar:
    cmd: python scripts/entrenar.py
    deps:
      - scripts/entrenar.py
      - data/processed/features.parquet
    params:
      - entrenamiento.dias_atras
      - entrenamiento.max_depth
    outs:
      - models/modelo.pkl
    metrics:
      - metrics/resultados.json:
          cache: false
```

```bash
dvc repro   # re-ejecuta SOLO las etapas cuyas dependencias cambiaron desde la última corrida
```

Esto es el equivalente de Make/Makefile aplicado a ML: DVC rastrea las dependencias entre etapas (`preparar_datos` → `entrenar`) y sus hashes de contenido — si `data/raw/demanda_historica.csv` no cambió desde la última ejecución, `dvc repro` **salta** la etapa de preparación de datos y solo reentrena si algo relevante realmente cambió (el código, los datos, o los parámetros declarados en `params`), ahorrando tiempo de cómputo real en iteraciones de desarrollo.

## 5. DVC + Git + MLflow — cómo encajan las tres piezas sin solaparse

| Herramienta | Qué versiona | Pregunta que responde |
|---|---|---|
| Git | Código fuente | "¿Qué lógica generó este resultado?" |
| DVC | Datos y pipelines | "¿Con qué datos exactos se entrenó esto?" |
| MLflow (ver [[15-MLflow-en-Profundidad]]) | Experimentos, métricas, modelos entrenados | "¿Qué parámetros e hiperparámetros se usaron, y con qué resultado?" |

No compiten — se complementan. Un flujo maduro: Git versiona `entrenar.py`, DVC versiona el dataset exacto que consumió ese script, y MLflow registra la corrida resultante (parámetros, métricas, y el artefacto del modelo) — las tres piezas juntas dan trazabilidad completa de extremo a extremo, algo que solo tu `estado_autoaprendizaje.json` actual cubre parcialmente y de forma manual.

## 6. ¿Vale la pena para Claro RD hoy?

Con criterio honesto, igual que con el Feature Store de la nota 43: DVC agrega valor real cuando **el dataset de entrenamiento cambia de forma versionable** (snapshots periódicos, distintas versiones curadas) y necesitas poder volver exactamente a una versión anterior de los datos, no solo del código. Si tu pipeline siempre lee "los últimos N días desde SQL Server" de forma dinámica (como probablemente haces hoy), el concepto de "versión fija de datos" es menos directo — DVC brilla más cuando trabajas con snapshots explícitos y curados, o cuando necesitas reproducir exactamente un entrenamiento histórico para debugging o auditoría.

## Ver también
- Cheat-sheet técnico completo de DVC (sintaxis, API, ejemplos): `DVC/01 - Introducción y Arquitectura Interna.md`
- [[15-MLflow-en-Profundidad]]
- [[12-Gestion-Moderna-de-Proyectos-Python]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]
