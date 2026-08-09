---
tags: [configuracion, python, buenas-practicas, seguridad]
---

# 24 — Configuración Profesional de Proyectos: `.env`, YAML, JSON y `config.py`

> Nota del mentor: probablemente ya tienes un `config.py` en tu proyecto de Claro RD. La pregunta que separa a un junior de un senior aquí no es "¿tengo configuración centralizada?" — es "¿mi configuración distingue correctamente entre secretos, parámetros de negocio y parámetros de infraestructura, y cada uno vive donde debe vivir?".

## 1. El principio central: nunca hardcodear, nunca commitear secretos

```python
# MAL — connection string hardcodeado en el código, va directo a git
CONNECTION_STRING = "DRIVER={SQL Server};SERVER=prod-db;UID=admin;PWD=SuperSecreto123"

# BIEN — el secreto vive fuera del código, el código solo sabe leerlo
import os
CONNECTION_STRING = os.environ["QFLOW_DB_CONNECTION_STRING"]
```

Un connection string con credenciales en el código fuente es, sin exagerar, uno de los incidentes de seguridad más comunes y más evitables en la industria — y una vez que algo llega a un commit de git, **sigue existiendo en el historial aunque lo borres después** (a menos que reescribas el historial completo).

## 2. `.env` — para desarrollo local

```bash
# .env (NUNCA se commitea — va en .gitignore)
QFLOW_DB_CONNECTION_STRING=DRIVER={SQL Server};SERVER=dev-db;UID=dev;PWD=dev123
MLFLOW_TRACKING_URI=http://localhost:5000
LOG_LEVEL=DEBUG
```

```python
from dotenv import load_dotenv
import os

load_dotenv()  # carga .env al entorno de variables de os.environ
db_conn = os.environ["QFLOW_DB_CONNECTION_STRING"]
```

Regla no negociable: **`.env` siempre en `.gitignore`**, y el repositorio debe incluir un `.env.example` con las claves necesarias pero sin valores reales, para que cualquier compañero nuevo sepa qué variables necesita configurar sin exponer ningún secreto real.

```bash
# .env.example (SÍ se commitea)
QFLOW_DB_CONNECTION_STRING=
MLFLOW_TRACKING_URI=
LOG_LEVEL=INFO
```

## 3. En producción: variables de entorno del sistema o CI/CD variables

En GitLab CI (ver [[14-CICD-para-ML-con-GitLab]]), los secretos viven como **CI/CD variables protegidas y enmascaradas**, nunca en un archivo `.env` dentro del repositorio ni en el `.gitlab-ci.yml`. En un servidor de producción tradicional, viven como variables de entorno del sistema operativo o en un gestor de secretos (Azure Key Vault, HashiCorp Vault) — la jerarquía de confianza siempre es: **gestor de secretos > variables de entorno > jamás en archivos versionados**.

## 4. YAML — para configuración legible de parámetros de negocio (no secretos)

```yaml
# config/entrenamiento.yaml
entrenamiento:
  dias_atras: 90
  dias_validacion: 14
  margen_mejora_minima: 0.02
  margen_tolerancia_rmse: 0.05

hiperparametros_xgboost:
  n_estimators: 300
  max_depth: 6
  learning_rate: 0.05

oficinas_excluidas:
  - 999  # oficina de pruebas, no incluir en producción
```

```python
import yaml
from pathlib import Path

def cargar_config_entrenamiento() -> dict:
    ruta = Path("config/entrenamiento.yaml")
    with ruta.open() as f:
        return yaml.safe_load(f)
```

YAML es ideal para configuración **jerárquica y legible por humanos** (parámetros de negocio, hiperparámetros, listas de exclusión) que un miembro del equipo no técnico (o menos técnico) puede revisar y ajustar sin tocar código Python. Nota: **siempre `yaml.safe_load`, nunca `yaml.load` a secas** — `yaml.load` sin especificar el loader puede ejecutar código arbitrario si el archivo YAML viene de una fuente no confiable, un vector de ataque real y documentado.

## 5. JSON — para configuración generada por máquina, no editada a mano

```json
{
  "ultima_fecha_reentrenamiento": "2026-07-15T08:00:00Z",
  "version_modelo_actual": "1.4.2",
  "metricas_ultimo_entrenamiento": {"mae": 12.4, "rmse": 18.7}
}
```

Este es exactamente el tipo de contenido de tu `estado_autoaprendizaje.json`: datos que el **pipeline mismo escribe y lee** como parte de su estado de ejecución, no configuración que un humano edita directamente. La distinción práctica: si un humano lo edita a mano regularmente → YAML (más legible, soporta comentarios). Si el programa lo genera y consume → JSON (más simple de serializar, sin comentarios porque no los necesita).

## 6. `pyproject.toml` — configuración de herramientas de desarrollo

Ya cubierto en profundidad en [[12-Gestion-Moderna-de-Proyectos-Python]]. Vale la pena recordar aquí la distinción de responsabilidad: `pyproject.toml` configura **cómo se construye y qué herramientas usa el proyecto** (dependencias, linter, pytest); YAML/JSON/`.env` configuran **cómo se comporta el proyecto en ejecución**. Son capas distintas que no deben mezclarse.

## 7. `config.py` — el punto de unión, con validación

```python
from dataclasses import dataclass
import os
import yaml

@dataclass(frozen=True)
class Configuracion:
    db_connection_string: str
    mlflow_tracking_uri: str
    dias_atras: int
    dias_validacion: int
    margen_mejora_minima: float

    @classmethod
    def cargar(cls) -> "Configuracion":
        params_negocio = yaml.safe_load(Path("config/entrenamiento.yaml").read_text())
        return cls(
            db_connection_string=os.environ["QFLOW_DB_CONNECTION_STRING"],
            mlflow_tracking_uri=os.environ.get("MLFLOW_TRACKING_URI", "http://localhost:5000"),
            dias_atras=params_negocio["entrenamiento"]["dias_atras"],
            dias_validacion=params_negocio["entrenamiento"]["dias_validacion"],
            margen_mejora_minima=params_negocio["entrenamiento"]["margen_mejora_minima"],
        )
```

`@dataclass(frozen=True)` (ver [[20-Python-Avanzado-Sistema-de-Tipos]]) hace que la configuración sea **inmutable** una vez cargada — evita el bug clásico de que algún módulo modifique `config.dias_atras` a mitad de ejecución y afecte silenciosamente a otro módulo que la lee después. `config.py` se convierte así en el único punto de la aplicación que sabe **de dónde** viene cada valor (variable de entorno vs. YAML), mientras el resto del código solo conoce el objeto `Configuracion` ya validado — exactamente el principio de bajo acoplamiento de [[22-Organizacion-del-Codigo-y-Principios-de-Diseno]].

## Ver también
- [[12-Gestion-Moderna-de-Proyectos-Python]]
- [[20-Python-Avanzado-Sistema-de-Tipos]]
- [[22-Organizacion-del-Codigo-y-Principios-de-Diseno]]
