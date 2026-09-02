---
tags: [optuna, hiperparametros, cli, referencia, cheat-sheet]
---

# 10 — CLI Reference

> Consolida comandos usados en [[05 - Persistencia y Ejecución Distribuida]] y [[07 - Visualización y Análisis de Resultados]].

Optuna incluye una CLI (`optuna`) para gestionar estudios sin necesariamente escribir un script Python — útil para administración rápida o integración en shell scripts de CI/CD.

## `optuna create-study` — crear un estudio desde la terminal

```bash
optuna create-study \
  --study-name "claro-rd-demand-tuning" \
  --storage "postgresql://usuario:password@host:5432/optuna_db" \
  --direction minimize
```

## `optuna studies` — listar estudios existentes

```bash
optuna studies --storage "postgresql://usuario:password@host:5432/optuna_db"
```

Muestra nombre, número de trials y estado de cada estudio guardado en ese storage.

## `optuna study optimize` — ejecutar la optimización desde CLI

Requiere que la función objetivo esté definida en un archivo `.py` importable:

```python
# objective_module.py
def objective(trial):
    x = trial.suggest_float("x", -10, 10)
    return (x - 2) ** 2
```

```bash
optuna study optimize objective_module.py objective \
  --n-trials 100 \
  --study-name "claro-rd-demand-tuning" \
  --storage "postgresql://usuario:password@host:5432/optuna_db"
```

En la práctica, este comando se usa menos que `study.optimize()` en Python directamente — es más relevante para lanzar **workers distribuidos** desde un orquestador externo (un job de Kubernetes, un paso de CI/CD) sin necesitar un wrapper de Python custom por worker.

## `optuna best-trial` / `optuna best-trials` — consultar resultados sin abrir Python

```bash
optuna best-trial --study-name "claro-rd-demand-tuning" --storage "postgresql://..."
optuna best-trials --study-name "claro-rd-demand-tuning" --storage "postgresql://..."   # multi-objetivo
```

## `optuna trials` — listar todos los trials de un estudio

```bash
optuna trials --study-name "claro-rd-demand-tuning" --storage "postgresql://..."
```

## `optuna delete-study` — eliminar un estudio

```bash
optuna delete-study --study-name "estudio-viejo" --storage "postgresql://..."
```

## `optuna-dashboard` — UI web (paquete separado)

```bash
pip install optuna-dashboard
optuna-dashboard postgresql://usuario:password@host:5432/optuna_db --port 8080
```

Ver [[07 - Visualización y Análisis de Resultados]] para el detalle de qué gráficas expone.

## Patrón de uso en pipelines de CI/CD

```bash
# .gitlab-ci.yml — paso de reentrenamiento con tuning programado
tuning_semanal:
  stage: train
  script:
    - optuna create-study --study-name "tuning-$(date +%Y%m%d)" --storage "$OPTUNA_STORAGE_URI" --direction minimize || true
    - optuna study optimize objective_module.py objective --n-trials 200 --study-name "tuning-$(date +%Y%m%d)" --storage "$OPTUNA_STORAGE_URI"
    - python promover_mejor_modelo.py --study-name "tuning-$(date +%Y%m%d)"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
```

El `|| true` evita que el pipeline falle si el estudio ya existe de una corrida previa (equivalente a `load_if_exists=True` en la API de Python).

## Ver también

- [[05 - Persistencia y Ejecución Distribuida]]
- [[07 - Visualización y Análisis de Resultados]]
- `Machine Learning/14-CICD-para-ML-con-GitLab.md`
