# 61 - Empaquetado y Distribución

> Continúa de [[60 - CLIs Profesionales|60]]. [[20 - Package Managers|20]], [[21 - pip Commands|21]] y [[22 - Virtual Environments|22]] cubrieron cómo **instalar y gestionar** paquetes; aquí se cubre el otro lado: cómo convertir tu propio código en un paquete instalable y, opcionalmente, publicarlo.

## De Script a Paquete Instalable: la Estructura Mínima

```text
mi_paquete/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── mi_paquete/
│       ├── __init__.py
│       └── core.py
└── tests/
    └── test_core.py
```

> **`src/` layout vs layout plano:** poner el código dentro de `src/mi_paquete/` (en vez de `mi_paquete/` directamente en la raíz) es la convención moderna recomendada — evita que las pruebas accidentalmente importen el código desde el directorio de trabajo en vez de desde la versión realmente **instalada** del paquete, detectando antes problemas de empaquetado.

## `pyproject.toml`: el Único Archivo de Configuración que Necesitas Hoy

Históricamente, empaquetar en Python requería `setup.py`, `setup.cfg` y `MANIFEST.in` por separado. **`pyproject.toml` (PEP 621)** unificó todo esto en un solo archivo estándar.

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "mi-paquete"
version = "0.1.0"
description = "Una herramienta para procesar datos"
readme = "README.md"
requires-python = ">=3.11"
license = "MIT"
authors = [{ name = "Tu Nombre", email = "tu@email.com" }]
dependencies = [
    "requests>=2.31.0",
    "pydantic>=2.0.0",
]

[project.optional-dependencies]
dev = ["pytest>=8.0", "ruff>=0.6", "mypy>=1.11"]

[project.scripts]
mi-comando = "mi_paquete.cli:main"   # instala un ejecutable de terminal llamado "mi-comando"

[tool.ruff]
line-length = 100

[tool.mypy]
strict = true
```

`[project.scripts]` es lo que convierte una función de tu paquete (`mi_paquete.cli:main`, ver [[60 - CLIs Profesionales|60]]) en un **comando de terminal real** disponible tras `pip install`, sin que el usuario final necesite saber que está escrito en Python.

## Build Backends: Setuptools, Hatchling, PDM, Flit

`pyproject.toml` requiere declarar un **build backend** — la herramienta que realmente empaqueta el código en un archivo distribuible.

| Backend | Características |
|---|---|
| `setuptools` | El más antiguo y con más compatibilidad legacy; sigue siendo válido |
| `hatchling` (Hatch) | Moderno, rápido, configuración mínima; ganando adopción |
| `flit-core` (Flit) | Extremadamente simple, ideal para paquetes puros de Python sin extensiones |
| `pdm-backend` (PDM) | Integrado con el gestor de dependencias PDM |

## Poetry: Gestión de Dependencias + Empaquetado en Uno

Poetry combina lo que `pip` + `venv` + `pyproject.toml` hacen por separado en una sola herramienta, con **resolución de dependencias determinista** mediante un archivo de lock.

```bash
pip install poetry

poetry new mi-paquete          # crea la estructura de un proyecto nuevo
poetry add requests             # agrega una dependencia (y actualiza pyproject.toml + poetry.lock)
poetry add --group dev pytest   # dependencia solo para desarrollo
poetry install                  # instala todo según poetry.lock (reproducible)
poetry run pytest               # ejecuta un comando dentro del entorno virtual gestionado
poetry build                    # genera los artefactos distribuibles (wheel + sdist)
poetry publish                  # publica en PyPI
```

```toml
# pyproject.toml generado por Poetry
[tool.poetry]
name = "mi-paquete"
version = "0.1.0"
description = "Una herramienta para procesar datos"

[tool.poetry.dependencies]
python = "^3.11"
requests = "^2.31.0"

[tool.poetry.group.dev.dependencies]
pytest = "^8.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

`poetry.lock` fija las versiones **exactas** de todas las dependencias (directas y transitivas), garantizando que `poetry install` produzca el mismo entorno en cualquier máquina — resolviendo el problema clásico de "en mi máquina funciona" causado por resoluciones de dependencias distintas.

## Hatch: la Alternativa Más Ligera y Moderna

```bash
pip install hatch

hatch new mi-paquete
hatch env create             # crea el entorno virtual del proyecto
hatch run pytest              # ejecuta comandos dentro del entorno
hatch build                    # genera wheel + sdist
hatch publish                  # publica en PyPI
```

## `pip` vs Poetry vs Hatch vs PDM: Panorama General

| | `pip` + `venv` (manual) | Poetry | Hatch | PDM |
|---|---|---|---|---|
| Archivo de lock reproducible | No (requiere `pip freeze` manual) | Sí (`poetry.lock`) | Sí (`hatch.lock`, más reciente) | Sí (`pdm.lock`, sigue el estándar PEP 751) |
| Gestión de entornos virtuales | Manual (`python -m venv`) | Automática | Automática | Automática |
| Build + publicación integrados | No (requiere `build` + `twine` por separado) | Sí | Sí | Sí |
| Curva de aprendizaje | Baja (herramientas separadas y conocidas) | Media | Baja-media | Media |
| Popularidad en proyectos nuevos (2025+) | Base universal | Muy alta | Creciendo rápido | Creciendo, especialmente en librerías |

## Wheels vs Source Distributions (sdist)

```bash
python -m build   # genera ambos formatos en dist/
```

```text
dist/
├── mi_paquete-0.1.0-py3-none-any.whl   # wheel: pre-construido, instalación rápida
└── mi_paquete-0.1.0.tar.gz              # sdist: código fuente, se construye al instalar
```

| | Wheel (`.whl`) | Source Distribution (`.tar.gz`) |
|---|---|---|
| Instalación | Rápida (solo copiar archivos) | Más lenta (ejecuta el build en la máquina destino) |
| Requiere compilador en la máquina destino (extensiones C) | No (ya viene compilado para esa plataforma) | Sí, si el paquete tiene extensiones nativas |
| Formato preferido por `pip` cuando está disponible | Sí | Solo como respaldo |

## Publicando en PyPI

```bash
pip install twine

python -m build                              # genera dist/
twine upload --repository testpypi dist/*    # primero probar en TestPyPI
twine upload dist/*                          # publicación real en PyPI
```

> Antes de publicar por primera vez, verifica que el nombre del paquete esté disponible en [pypi.org](https://pypi.org), y considera usar **TestPyPI** (`test.pypi.org`) para ensayar el proceso completo sin arriesgar el nombre real.

## Versionado Semántico (SemVer)

```text
MAYOR.MENOR.PARCHE   ->   2.4.1
```

| Componente | Se incrementa cuando... |
|---|---|
| MAYOR | Hay cambios incompatibles con versiones anteriores (_breaking changes_) |
| MENOR | Se agrega funcionalidad nueva, compatible con versiones anteriores |
| PARCHE | Se corrigen bugs, sin cambios de API |

Seguir SemVer permite que quienes dependen de tu paquete usen restricciones de versión con confianza (`mi-paquete = "^2.4.0"` significa "acepto cualquier 2.x.x igual o posterior a 2.4.0, pero no 3.0.0").

## Ver también

- [[20 - Package Managers]] y [[21 - pip Commands]] — la base de instalación que este archivo complementa desde el lado de creación de paquetes.
- [[22 - Virtual Environments]] — los entornos que Poetry/Hatch/PDM gestionan automáticamente.
- [[60 - CLIs Profesionales]] — `[project.scripts]` como puente entre tu código y un comando instalable.
- [[44 - Linters y Formateadores]] — dónde vive la configuración de `ruff`/`mypy` dentro del mismo `pyproject.toml`.
