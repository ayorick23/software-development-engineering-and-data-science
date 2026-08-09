---
tags: [python, packaging, buenas-practicas, mlops]
---

# 12 — pyproject.toml y Gestión Moderna de Proyectos Python

> Nota del mentor: cuando heredaste el proyecto de Adrián, seguramente encontraste un `requirements.txt` con versiones sin fijar, o peor, sin ningún archivo de dependencias — "funciona en mi máquina". `pyproject.toml` es el estándar que resuelve ese caos, y entenderlo bien es lo que separa a alguien que "sabe Python" de alguien que "sabe construir software en Python".

## 1. Breve historia — por qué existe `pyproject.toml`

Python packaging fue durante años un desastre de archivos dispersos: `setup.py`, `setup.cfg`, `requirements.txt`, `MANIFEST.in`, cada herramienta con su propio formato. **PEP 518** (2017) introdujo `pyproject.toml` como un único archivo estándar, en formato TOML, para declarar cómo se construye un proyecto Python — de forma análoga a como `package.json` centraliza todo en Node.js o `Cargo.toml` en Rust.

No nació por moda: nació porque instalar un paquete requería ejecutar código arbitrario (`setup.py`) antes de siquiera saber qué dependencias necesitabas para construirlo — un problema real de seguridad y reproducibilidad.

## 2. Anatomía de un `pyproject.toml`

```toml
[project]
name = "claro-forecasting-pipeline"
version = "2.1.0"
description = "Pipeline de forecasting de demanda y tiempo de servicio para Claro RD"
requires-python = ">=3.11"
dependencies = [
    "pandas>=2.0,<3.0",
    "xgboost>=2.0",
    "scikit-learn>=1.4",
    "mlflow>=2.10",
    "pyodbc>=5.0",
]

[project.optional-dependencies]
dev = ["pytest>=8.0", "pytest-cov", "ruff", "black"]

[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.build_meta"

[tool.ruff]
line-length = 100

[tool.pytest.ini_options]
testpaths = ["tests"]
```

Tres secciones clave que debes entender de memoria:

- **`[project]`**: metadata del paquete y, lo más importante para tu día a día, la lista de **dependencias fijadas con rangos de versión**. Esto es lo que evita el "en mi máquina funciona": todos en el equipo instalan versiones compatibles, no "lo último que había disponible ese día".
- **`[build-system]`**: le dice a herramientas como `pip` con qué motor construir el paquete (`setuptools`, `hatchling`, `poetry-core`, etc.). Es lo que permite que `pip install .` funcione sin ambigüedad.
- **Secciones `[tool.*]`**: cada herramienta (linter, formateador, test runner) puede vivir configurada en el mismo archivo, en vez de tener `.flake8`, `pytest.ini`, `.blackrc` regados por el repo.

## 3. `requirements.txt` vs `pyproject.toml` — no son excluyentes

Un error común de junior es pensar que hay que elegir uno. La realidad:

| | `requirements.txt` | `pyproject.toml` |
|---|---|---|
| Propósito | Congelar el *entorno exacto* (pip freeze) | Declarar *qué necesita el paquete para funcionar* |
| Versionado | Exacto (`==2.1.3`) | Rangos semánticos (`>=2.0,<3.0`) |
| Reproducibilidad de CI | Alta (mismas versiones siempre) | Depende del resolutor |
| Ideal para | Entornos de despliegue, contenedores Docker | Librerías y paquetes instalables |

En proyectos maduros verás **ambos**: `pyproject.toml` para declarar el paquete, y un lockfile (`requirements.lock`, `poetry.lock` o `uv.lock`) generado a partir de él para fijar el entorno exacto de producción.

## 4. Gestores de proyecto: pip, Poetry, uv

- **`pip` + `venv`**: la base de todo, siempre disponible, pero manual — tú gestionas el entorno virtual, el lockfile, todo.
- **Poetry**: gestor de dependencias y empaquetado todo-en-uno, muy popular en 2020-2023. Resuelve dependencias de forma determinista y genera `poetry.lock`.
- **uv** (Astral, los creadores de `ruff`): el gestor más nuevo y hoy el más recomendado para proyectos de ML — está escrito en Rust, es entre 10 y 100 veces más rápido que `pip` resolviendo dependencias, y es compatible con `pyproject.toml` estándar sin necesitar su propio formato propietario.

```bash
# Con uv, el flujo típico en un proyecto de ML
uv init mi-proyecto-ml
uv add pandas scikit-learn mlflow
uv add --dev pytest ruff
uv run python train.py
```

En un entorno como Claro RD, donde probablemente trabajas en Windows con PowerShell, `uv` también simplifica mucho el manejo de entornos virtuales entre máquinas de distintos compañeros del equipo.

## 5. Estructura de un repositorio profesional (`src` layout)

```
claro-forecasting-pipeline/
├── pyproject.toml
├── README.md
├── .gitlab-ci.yml
├── src/
│   └── forecasting_pipeline/
│       ├── __init__.py
│       ├── config.py
│       ├── database.py
│       ├── forecasting.py
│       ├── feature_engineering.py
│       ├── staffing.py
│       └── logging_config.py
├── tests/
│   ├── test_forecasting.py
│   └── test_feature_engineering.py
└── notebooks/
    └── 01_eda.ipynb
```

El **`src` layout** (código dentro de `src/nombre_paquete/` en vez de en la raíz) no es capricho: fuerza a que las pruebas se ejecuten contra el paquete *instalado*, no contra archivos sueltos en el path — esto detecta errores de empaquetado antes de que lleguen a producción, algo que un layout plano puede ocultar silenciosamente.

## 6. Versionado semántico (SemVer)

`MAJOR.MINOR.PATCH` (ej. `2.1.0`):

- **MAJOR**: cambios incompatibles (rompes la firma de una función que otros módulos usan).
- **MINOR**: nueva funcionalidad compatible hacia atrás (agregas soporte para un nuevo `office_id` sin romper nada existente).
- **PATCH**: corrección de bugs sin cambiar comportamiento esperado.

Esto se conecta directo con [[15-MLflow-en-Profundidad]]: cuando registras un modelo, su versión en el Model Registry debería estar alineada con la versión de tu código que lo generó — si no, pierdes trazabilidad de "qué código produjo qué modelo".

## Ver también

- [[09-MLOps-en-Profundidad]]
- [[13-Testing-en-Machine-Learning]]
- [[14-CICD-para-ML-con-GitLab]]
