---
tags: [cicd, gitlab, mlops, devops]
---

# 14 — CI/CD para Machine Learning con GitLab

> Nota del mentor: CI/CD para software tradicional ya es un tema maduro. CI/CD para *Machine Learning* (a veces llamado **CT — Continuous Training**) tiene una capa extra de complejidad: no solo despliegas código, despliegas **código + datos + modelo**, y los tres cambian a ritmos distintos. Vas a vivir esto directamente en Claro RD en cuanto el proyecto se suba a GitLab.

## 1. CI, CD y CT — las tres continuidades

- **CI (Continuous Integration)**: cada cambio de código se integra, se testea y se valida automáticamente (linting, `pytest`, validación de esquema de datos).
- **CD (Continuous Delivery/Deployment)**: el código validado se empaqueta y se despliega (a un entorno de staging, o directo a producción con aprobación).
- **CT (Continuous Training)**: exclusivo de ML — el **modelo** se reentrena automáticamente cuando hay suficiente drift o nuevos datos, con su propio gate de calidad antes de reemplazar al modelo en producción.

Tu sistema de autoaprendizaje (champion/challenger con gate de MAE/RMSE) es, en esencia, una implementación artesanal de CT. Formalizarlo dentro de un pipeline de GitLab CI es el siguiente paso natural de madurez.

## 2. Anatomía de `.gitlab-ci.yml`

```yaml
stages:
  - lint
  - test
  - validate-data
  - train
  - deploy

variables:
  PYTHON_VERSION: "3.11"

lint:
  stage: lint
  image: python:3.11-slim
  script:
    - pip install ruff
    - ruff check src/

unit-tests:
  stage: test
  image: python:3.11-slim
  script:
    - pip install -e ".[dev]"
    - pytest tests/ --cov=src --cov-report=term-missing
  coverage: '/TOTAL.*\s+(\d+%)$/'

data-validation:
  stage: validate-data
  image: python:3.11-slim
  script:
    - pip install -e .
    - python scripts/validate_input_schema.py
  only:
    - schedules

train-and-evaluate:
  stage: train
  image: python:3.11-slim
  script:
    - pip install -e .
    - python scripts/train_challenger.py
    - python scripts/evaluate_gate.py  # falla el job si no supera al champion
  artifacts:
    paths:
      - models/challenger_metrics.json
  only:
    - schedules

deploy-production:
  stage: deploy
  script:
    - python scripts/promote_model.py
  environment:
    name: production
  when: manual
  only:
    - main
```

Cosas a las que prestar atención línea por línea, porque son las que un junior suele pasar por alto:

- **`stages`** define el orden; cada stage solo arranca si el anterior pasó. Si `lint` falla, `test` ni siquiera corre — esto ahorra minutos de cómputo y feedback más rápido.
- **`only: schedules`** en el job de entrenamiento: no quieres reentrenar en cada `git push`, sino en una cadencia controlada (como ya haces con tu `estado_autoaprendizaje.json`), disparada por un **Pipeline Schedule** de GitLab.
- **`when: manual`** en `deploy-production`: en un cliente como Claro RD, donde un modelo malo afecta la dotación real de agentes, el paso a producción casi nunca debería ser 100% automático sin un humano dando el clic final — al menos hasta que el equipo confíe plenamente en los gates automáticos.
- **`artifacts`**: los archivos que un job genera (métricas, modelos, reportes) y que quieres conservar para inspección o para pasar al siguiente stage.

## 3. GitLab Runners — quién ejecuta todo esto

Un **Runner** es el agente (contenedor, VM o máquina física) que realmente ejecuta los jobs. En un contexto empresarial como ACF Technologies, normalmente hay runners compartidos (shared runners de GitLab.com o GitLab self-hosted) o runners dedicados dentro de la red de la empresa — esto último es común cuando el pipeline necesita conectarse a bases de datos internas del cliente (como `qf.hrAgentForecastResult` en tu caso), que no son accesibles desde internet público.

## 4. GitLab CI/CD específico para artefactos de ML

Dos piezas que en software tradicional no existen pero en ML son centrales:

- **Model Registry integrado**: GitLab tiene su propio *Model Registry* (o puedes usar el de MLflow, ver [[15-MLflow-en-Profundidad]]) para versionar modelos como artefactos de primera clase, no solo como archivos `.pkl` sueltos en una carpeta.
- **GitLab CI/CD variables protegidas**: credenciales de base de datos, API keys, connection strings — nunca hardcodeadas en `config.py`, siempre como *CI/CD variables* marcadas como *protected* y *masked* en la configuración del proyecto.

## 5. Merge Requests como puerta de calidad

El flujo real de trabajo en equipo, y el que deberías empezar a exigir en tu proyecto:

```
feature/fix-warmstart-bug → Merge Request → pipeline CI corre automático
                                                  ↓
                                    lint ✅  tests ✅  validate-data ✅
                                                  ↓
                                    Revisión de código por un compañero
                                                  ↓
                                    Merge a main → deploy pipeline
```

GitLab permite bloquear el merge si el pipeline falla (**"Pipelines must succeed"** en la configuración de protección de rama). Esto es lo que convierte tus tests de [[13-Testing-en-Machine-Learning]] de "buena intención" a "barrera real" — sin esta configuración, cualquiera puede mergear código roto y los tests se vuelven decorativos.

## 6. Errores comunes al implementar CI/CD para ML

- **Reentrenar en cada push de código.** El entrenamiento es costoso; se dispara por cadencia de datos, no por cadencia de código.
- **No versionar los datos de entrenamiento junto al código.** Sin herramientas como DVC o un snapshot versionado, no puedes reproducir exactamente qué generó un modelo en producción hace tres meses.
- **Pipelines de CI que tardan 40 minutos.** Cachea dependencias (`cache:` en GitLab CI), paraleliza tests, y separa validaciones rápidas (lint, unit tests) de las lentas (entrenamiento completo).
- **Secretos en el `.gitlab-ci.yml` en texto plano.** Siempre usar CI/CD variables protegidas, nunca commitear credenciales, ni siquiera "temporalmente".

## Ver también

- [[09-MLOps-en-Profundidad]]
- [[13-Testing-en-Machine-Learning]]
- [[15-MLflow-en-Profundidad]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]
