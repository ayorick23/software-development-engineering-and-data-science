---
tags: [dvc, mlops, pipelines, reproducibilidad, cheat-sheet]
---

# 04 — DVC Pipelines: dvc.yaml y Reproducibilidad

> Continúa de [[03 - Remotes y Almacenamiento en la Nube]]. Ver también `Machine Learning/50-Orquestacion-Prefect-y-Airflow.md` para orquestación a mayor escala.

**DVC Pipelines** extiende el versionado de archivos individuales a la reproducibilidad de un **proceso completo** — el equivalente de un Makefile aplicado a flujos de trabajo de ML, donde DVC rastrea automáticamente qué etapas necesitan re-ejecutarse según qué cambió.

## `dvc.yaml` — la definición del pipeline

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

  evaluar:
    cmd: python scripts/evaluar.py
    deps:
      - scripts/evaluar.py
      - models/modelo.pkl
      - data/processed/features.parquet
    metrics:
      - metrics/evaluacion.json:
          cache: false
    plots:
      - plots/curva_residuos.csv
```

### Anatomía de una etapa (`stage`)

| Clave | Propósito |
|---|---|
| `cmd` | El comando que se ejecuta (script, notebook convertido, cualquier ejecutable) |
| `deps` | Archivos/carpetas de los que depende — si cambian, la etapa se re-ejecuta |
| `outs` | Archivos que produce — se versionan automáticamente, como con `dvc add` |
| `params` | Claves específicas de `params.yaml` de las que depende (no el archivo completo) |
| `metrics` | Archivos de métricas — se muestran con `dvc metrics` (ver [[05 - Params, Metrics y Plots]]) |
| `plots` | Archivos de datos para graficar — se muestran con `dvc plots` |

## `dvc stage add` — generar `dvc.yaml` desde la línea de comandos

```bash
dvc stage add -n preparar_datos \
  -d scripts/preparar_datos.py -d data/raw/demanda_historica.csv \
  -o data/processed/features.parquet \
  python scripts/preparar_datos.py
```

Alternativa a editar `dvc.yaml` a mano — genera la misma entrada de forma programática, útil para scripts que configuran pipelines dinámicamente.

## `params.yaml` — hiperparámetros versionados como código

```yaml
# params.yaml
entrenamiento:
  dias_atras: 90
  max_depth: 6
  learning_rate: 0.05
```

Al declarar `params: [entrenamiento.dias_atras, entrenamiento.max_depth]` en una etapa, DVC vigila **específicamente esas claves** — cambiar `learning_rate` sin tocar `dias_atras`/`max_depth` no dispara un re-entrenamiento si esa etapa no depende de `learning_rate`, dando control fino sobre qué cambios realmente invalidan cada etapa.

## `dvc repro` — re-ejecutar solo lo necesario

```bash
dvc repro
```

Compara los hashes actuales de cada `deps`/`params` contra los registrados en `dvc.lock` (el archivo de estado, generado automáticamente) — si nada cambió desde la última ejecución, **la etapa se salta por completo**, ahorrando tiempo de cómputo real en iteraciones de desarrollo.

```bash
dvc repro entrenar          # re-ejecuta solo hasta la etapa "entrenar" (y sus dependencias previas si cambiaron)
dvc repro --force             # fuerza la re-ejecución de TODO, ignorando si algo cambió
dvc repro --dry              # muestra qué se ejecutaría, SIN ejecutar nada realmente
```

## `dvc.lock` — el estado exacto de la última ejecución reproducible

```yaml
# dvc.lock (generado automáticamente, NO se edita a mano)
stages:
  entrenar:
    cmd: python scripts/entrenar.py
    deps:
      - path: scripts/entrenar.py
        md5: a1b2c3...
      - path: data/processed/features.parquet
        md5: d4e5f6...
    params:
      params.yaml:
        entrenamiento.dias_atras: 90
        entrenamiento.max_depth: 6
    outs:
      - path: models/modelo.pkl
        md5: g7h8i9...
```

`dvc.lock` es análogo a un `package-lock.json`/`poetry.lock` (ver `Machine Learning/12-Gestion-Moderna-de-Proyectos-Python.md`), pero para todo el pipeline: fija los hashes exactos de cada dependencia, parámetro y output de la última ejecución exitosa — **se versiona en Git**, junto a `dvc.yaml`, para que cualquiera pueda reconstruir exactamente el mismo estado con `dvc repro`/`dvc checkout`.

## `dvc dag` — visualizar el grafo de dependencias

```bash
dvc dag
```

```
      +-----------------+
      | preparar_datos  |
      +-----------------+
               *
               *
         +-----------+
         | entrenar  |
         +-----------+
               *
               *
         +-----------+
         | evaluar   |
         +-----------+
```

```bash
dvc dag --md   # exporta el DAG como diagrama Mermaid, para incluir en documentación
```

## Etapas con múltiples outputs y foreach (matriz de configuraciones)

```yaml
stages:
  entrenar:
    foreach:
      - modelo_a
      - modelo_b
    do:
      cmd: python scripts/entrenar.py --config configs/${item}.yaml
      deps:
        - scripts/entrenar.py
        - configs/${item}.yaml
      outs:
        - models/${item}.pkl
```

`foreach`/`do` genera múltiples etapas a partir de una plantilla — útil para entrenar variantes de un mismo pipeline (distintos algoritmos, distintas configuraciones) sin duplicar la definición completa de la etapa para cada una.

## Reproducibilidad de extremo a extremo — el flujo completo

```bash
git checkout v1.4.2   # código exacto de esa versión
dvc checkout            # datos exactos de esa versión (inputs)
dvc repro                # reconstruye pipeline — si algo se corrompió, lo regenera; si no, usa cache
```

Esto responde de forma verificable a "¿puedo reproducir exactamente el modelo que está en producción?" — sin depender de que alguien recuerde manualmente los pasos, gracias a que `dvc.yaml` + `dvc.lock` capturan el proceso completo, no solo los datos de entrada.

## Ver también

- [[02 - Versionado de Datos - Comandos Fundamentales]]
- [[05 - Params, Metrics y Plots]]
- [[06 - DVC Experiments - Iteración Rápida sin Comprometer Git]]
- `Machine Learning/50-Orquestacion-Prefect-y-Airflow.md`
