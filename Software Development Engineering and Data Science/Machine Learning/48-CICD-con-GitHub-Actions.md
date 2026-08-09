---
tags: [cicd, github-actions, devops]
---

# 48 — CI/CD con GitHub Actions (Complemento a GitLab)

> Nota del mentor: en Claro RD usas GitLab CI, y ahí es donde debes ser fuerte primero — pero GitHub Actions es tan común en la industria (especialmente en proyectos open source, librerías propias, y muchas empresas que no usan GitLab) que dominarlo también te da flexibilidad real. Los conceptos son 90% los mismos; esta nota se enfoca en las diferencias que realmente importan.

## 1. Anatomía de un workflow de GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: "0 6 * * *"  # cada día a las 6am UTC — equivalente a un Pipeline Schedule de GitLab

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: "pip"

      - name: Instalar dependencias
        run: pip install -e ".[dev]"

      - name: Lint
        run: ruff check src/

      - name: Tests
        run: pytest tests/ --cov=src --cov-report=xml

      - name: Subir cobertura
        uses: codecov/codecov-action@v4

  entrenar-y-evaluar:
    needs: lint-and-test
    runs-on: ubuntu-latest
    if: github.event_name == 'schedule'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - run: pip install -e .
      - run: python scripts/train_challenger.py
      - run: python scripts/evaluate_gate.py
```

## 2. Diferencias clave frente a GitLab CI — vocabulario y estructura

| Concepto | GitLab CI | GitHub Actions |
|---|---|---|
| Archivo de configuración | `.gitlab-ci.yml` | `.github/workflows/*.yml` (puede haber varios) |
| Unidad de ejecución | `job` dentro de `stages` | `job` dentro de `workflow`, con `steps` |
| Runner | GitLab Runner (compartido o propio) | GitHub-hosted runner o self-hosted |
| Piezas reutilizables | `include`, plantillas | **Actions** (`uses: actions/checkout@v4`) — componentes empaquetados y versionados |
| Trigger por cadencia | Pipeline Schedules (configurado en la UI) | `on: schedule` con sintaxis `cron` directamente en el YAML |
| Variables/secretos | CI/CD Variables (UI del proyecto) | GitHub Secrets (Settings → Secrets) |

La diferencia conceptual más importante es el ecosistema de **Actions reutilizables** (`uses: ...`): en vez de escribir manualmente el `script` para hacer checkout, configurar Python, o subir cobertura (como harías en GitLab con comandos explícitos), GitHub Actions tiene un marketplace enorme de acciones pre-construidas y versionadas por la comunidad — reduce la cantidad de YAML que escribes, a cambio de depender de terceros para pasos comunes.

## 3. Secrets — mismo principio de seguridad, sintaxis distinta

```yaml
steps:
  - name: Conectar a base de datos
    env:
      QFLOW_DB_CONNECTION_STRING: ${{ secrets.QFLOW_DB_CONNECTION_STRING }}
    run: python scripts/procesar.py
```

Exactamente el mismo principio que las CI/CD variables protegidas de GitLab de [[14-CICD-para-ML-con-GitLab]]: los secretos se configuran en la interfaz del repositorio (`Settings → Secrets and variables → Actions`), nunca en el YAML en texto plano, y se inyectan como variables de entorno solo durante la ejecución del job.

## 4. Matrix builds — probar múltiples versiones en paralelo

```yaml
jobs:
  test:
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pytest tests/
```

Esto ejecuta el job completo **una vez por cada combinación** de la matriz (aquí, tres versiones de Python en paralelo) — muy útil si mantienes una librería propia que debe funcionar en múltiples versiones de Python o de dependencias clave, algo especialmente relevante si ACF Technologies llegara a publicar una librería interna compartida entre proyectos de distintos clientes.

## 5. Model Registry y despliegue — el mismo patrón, ecosistema distinto

```yaml
jobs:
  publish-docker:
    needs: entrenar-y-evaluar
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/acf-technologies/claro-forecasting:${{ github.sha }}
```

`GITHUB_TOKEN` es un secreto generado automáticamente por GitHub en cada ejecución, con permisos limitados al repositorio actual — no necesitas crear ni gestionar esa credencial manualmente, a diferencia de GitLab donde normalmente configuras tus propias credenciales de acceso al Container Registry.

## 6. Cuándo vas a encontrarte con GitHub Actions en la práctica

- **Contribuir o depender de proyectos open source** (la inmensa mayoría de librerías de Python del ecosistema ML — scikit-learn, XGBoost, MLflow mismo — usan GitHub Actions para su propio CI).
- **Si ACF Technologies o un cliente futuro migra de GitLab a GitHub**, o si trabajas en un proyecto paralelo/personal alojado en GitHub.
- **Empresas más pequeñas o startups**, donde GitHub (por su plan gratuito generoso y su ecosistema) es más común que GitLab self-hosted o Enterprise.

## 7. La habilidad transferible real

Con lo que ya sabes de [[14-CICD-para-ML-con-GitLab]], aprender GitHub Actions no es aprender un concepto nuevo — es aprender un **dialecto distinto de los mismos conceptos**: stages/jobs, triggers, variables protegidas, artefactos, y gates de calidad antes de merge. La inversión real de tiempo es baja una vez que el concepto de CI/CD está sólido, que es exactamente lo que ya construiste con GitLab.

## Ver también
- [[14-CICD-para-ML-con-GitLab]]
- [[47-Docker-en-Profundidad-para-ML]]
- [[13-Testing-en-Machine-Learning]]
