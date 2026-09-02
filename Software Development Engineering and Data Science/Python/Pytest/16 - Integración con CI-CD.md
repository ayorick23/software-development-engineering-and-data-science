---
tags: [pytest, python, testing, ci-cd, cheat-sheet]
---

# 16 — Integración con CI-CD

> Continúa de [[15 - Property-Based Testing con Hypothesis]].

## pytest dentro de GitHub Actions

```yaml
# .github/workflows/tests.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: pytest --cov=mi_paquete --cov-report=xml --cov-fail-under=80
```

Ver el detalle completo de configuración de workflows (matrices de versiones, caché de dependencias, triggers) en [[Machine Learning/48-CICD-con-GitHub-Actions|Machine Learning/48 - CI/CD con GitHub Actions]] — este archivo cubre específicamente las banderas y salidas de pytest relevantes dentro de ese contexto.

## Matriz de versiones de Python

```yaml
strategy:
  matrix:
    python-version: ["3.10", "3.11", "3.12"]
steps:
  - uses: actions/setup-python@v5
    with:
      python-version: ${{ matrix.python-version }}
  - run: pytest
```

Ejecutar la suite completa contra múltiples versiones de Python en paralelo detecta incompatibilidades antes de que un usuario con una versión distinta las encuentre en producción — el mismo objetivo que [[Testing#Tox El Entorno de Pruebas Aislado|Tox]] resuelve localmente, aplicado ahora a nivel de pipeline de CI.

## Reportes en formato JUnit XML

```bash
pytest --junitxml=resultados.xml
```

El formato JUnit XML es un estándar reconocido por prácticamente cualquier plataforma de CI (GitHub Actions, GitLab CI, Jenkins) para mostrar resultados de tests de forma visual e integrada (qué tests pasaron/fallaron, directamente en la interfaz del pipeline) — sin este formato, solo se tendría el log de texto plano de la terminal.

## Separar tests rápidos de tests lentos en el pipeline

```yaml
jobs:
  tests-rapidos:
    steps:
      - run: pytest -m "not lento"        # feedback rápido en cada push/PR

  tests-completos:
    if: github.event_name == 'schedule'     # solo en un job programado (ej. nocturno)
    steps:
      - run: pytest      # suite completa, incluyendo tests lentos/de integración
```

Este patrón (ver los marks personalizados en [[06 - Marks y Selección de Tests#Seleccionar tests por mark o por nombre|Marks]]) balancea velocidad de feedback en cada commit contra cobertura completa periódica — nadie quiere esperar 20 minutos de tests de integración lentos en cada push, pero sí correrlos regularmente.

## Fallar rápido en CI: `-x` y `--maxfail`

```bash
pytest -x                      # detiene en el PRIMER fallo — feedback más rápido en CI cuando un fallo probablemente indica un problema más amplio
pytest --maxfail=3               # detiene después de 3 fallos acumulados
```

## Cachear dependencias y resultados de tests entre corridas de CI

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
```

```bash
pytest --lf     # combinado con caché de pytest (.pytest_cache/) persistido entre corridas, re-ejecuta solo lo que falló antes
```

Cachear tanto las dependencias de Python como el propio caché interno de pytest (`.pytest_cache/`, usado por `--lf`/`--ff`, ver [[02 - Escritura y Descubrimiento de Tests#Opciones de línea de comandos más usadas|Escritura de Tests]]) acelera pipelines de CI que corren frecuentemente sobre la misma base de código.

## Ver también

- [[15 - Property-Based Testing con Hypothesis]]
- [[12 - Cobertura de Código]]
- [[Machine Learning/48-CICD-con-GitHub-Actions|Machine Learning/48 - CI/CD con GitHub Actions]]
- [[Testing#Tox El Entorno de Pruebas Aislado|Python/Testing]]
