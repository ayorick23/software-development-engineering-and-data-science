---
tags: [dvc, mlops, cicd, cml, github-actions, gitlab-ci, cheat-sheet]
---

# 08 — Integración con CI/CD y CML

> Continúa de [[06 - DVC Experiments - Iteración Rápida sin Comprometer Git]]. Ver también `Machine Learning/14-CICD-para-ML-con-GitLab.md` y `Machine Learning/48-CICD-con-GitHub-Actions.md`.

## `dvc pull` dentro de un pipeline de CI/CD — el paso básico

Cualquier job de CI/CD que necesite los datos reales (no solo el código) debe traerlos explícitamente, igual que en un clon local:

```yaml
# .gitlab-ci.yml
entrenar_y_validar:
  stage: train
  script:
    - pip install dvc[s3]
    - dvc pull   # trae los datos versionados, usando las credenciales del remote configuradas en el CI
    - dvc repro    # reproduce el pipeline completo, reentrenando solo lo que cambió
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
```

```yaml
# .github/workflows/train.yml
jobs:
  train:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install "dvc[s3]"
      - run: dvc pull
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      - run: dvc repro
```

Las credenciales del remote (ver [[03 - Remotes y Almacenamiento en la Nube]]) se inyectan como secretos del sistema de CI/CD, nunca hardcodeadas en el pipeline.

## CML (Continuous Machine Learning) — reportes automáticos en Pull/Merge Requests

**CML** es una herramienta complementaria (del mismo equipo que DVC) que publica automáticamente métricas, plots y comparaciones **directamente como comentario en un PR/MR** — convierte la revisión de un cambio de modelo en algo tan visual como revisar un diff de código.

```bash
npm install -g @dvcorg/cml
# o, en GitHub Actions, se usa la action oficial: iterative/setup-cml
```

```yaml
# .github/workflows/cml.yml
name: train-and-report
on: [pull_request]

jobs:
  train:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: iterative/setup-cml@v2
      - run: pip install "dvc[s3]"
      - run: dvc pull
      - run: dvc repro

      - name: Publicar reporte en el PR
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "## Resultados del experimento" >> reporte.md
          dvc metrics diff main --show-md >> reporte.md
          dvc params diff main --show-md >> reporte.md
          cml comment create reporte.md
```

El resultado: cada Pull Request que modifica código de entrenamiento o datos genera automáticamente un comentario con una tabla mostrando exactamente cómo cambiaron las métricas respecto a `main` — sin que nadie tenga que correr el pipeline manualmente ni copiar resultados a mano para reportarlos.

## Incluir gráficas en el reporte de CML

```yaml
      - name: Publicar reporte con gráficas
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          dvc plots diff main --show-vega > vega.json
          vl2png vega.json plot.png
          echo "## Curva de residuos" >> reporte.md
          echo '![](./plot.png)' >> reporte.md
          cml comment create reporte.md
```

## `dvc exp` dentro de CI/CD — automatizar búsquedas completas

```yaml
jobs:
  buscar_hiperparametros:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install "dvc[s3]"
      - run: dvc pull
      - run: |
          dvc exp run --queue --set-param entrenamiento.max_depth=4
          dvc exp run --queue --set-param entrenamiento.max_depth=6
          dvc exp run --queue --set-param entrenamiento.max_depth=8
          dvc exp run --run-all -j 3
      - run: dvc exp show --csv > resultados.csv
```

Ejecutar `dvc exp` dentro de CI/CD centraliza la búsqueda en infraestructura compartida (no en la máquina de un desarrollador individual) — relevante cuando se quiere que la búsqueda de hiperparámetros forme parte de un pipeline programado (ej. reentrenamiento semanal, ver `Machine Learning/14-CICD-para-ML-con-GitLab.md`), en vez de un proceso manual.

## Runners con acceso a GPU — CML self-hosted

```yaml
      - uses: iterative/setup-cml@v2
      - name: Lanzar runner con GPU en la nube (bajo demanda)
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        run: |
          cml runner launch \
            --cloud=aws --cloud-region=us-east-1 --cloud-type=g4dn.xlarge \
            --labels=cml-gpu
```

CML puede aprovisionar automáticamente una instancia cloud con GPU **solo para la duración del job** de entrenamiento, y destruirla al terminar — evita mantener infraestructura GPU corriendo permanentemente cuando el entrenamiento es esporádico (ej. reentrenamientos semanales), pagando únicamente por el tiempo de cómputo real usado.

## Gate de calidad automatizado — fallar el CI si el modelo empeora

```yaml
      - name: Verificar que el modelo no empeoró
        run: |
          python -c "
          import json, sys
          with open('metrics/resultados.json') as f:
              nuevo = json.load(f)
          if nuevo['mae'] > 15.0:
              print('MAE supera el umbral aceptable')
              sys.exit(1)
          "
```

Combinado con `dvc metrics diff` contra el branch principal, este patrón implementa un gate de calidad automatizado (conceptualmente equivalente a `validation_thresholds` de `mlflow.evaluate`, ver `MLflow/11 - Evaluación de Modelos (mlflow.evaluate).md`) — el Pull Request no se puede mergear si el modelo candidato no supera el umbral definido.

## Ver también

- [[06 - DVC Experiments - Iteración Rápida sin Comprometer Git]]
- `Machine Learning/14-CICD-para-ML-con-GitLab.md`
- `Machine Learning/48-CICD-con-GitHub-Actions.md`
- `MLflow/11 - Evaluación de Modelos (mlflow.evaluate).md`
