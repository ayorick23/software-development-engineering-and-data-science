---
tags: [airflow, instalacion, docker-compose, astro, configuracion, cheat-sheet]
---

# 02 — Instalación y Entorno Local

> Continúa de [[01 - Introducción y Arquitectura]]. Hay cuatro formas habituales de tener Airflow en tu máquina: **pip con constraints**, **`airflow standalone`**, el **Docker Compose oficial** y la **Astro CLI**. Esta nota cubre las cuatro, la estructura de un proyecto y cómo funciona la configuración (`airflow.cfg` y variables `AIRFLOW__`).

## ¿Qué opción elegir?

| Opción | Ideal para | Pros | Contras |
|---|---|---|---|
| `pip` + `airflow standalone` | Aprender, probar rápido | Un comando, sin Docker | SQLite y LocalExecutor; en Windows requiere WSL2 |
| **Docker Compose oficial** | Desarrollo realista | Postgres, Celery y Redis como en producción | Pesado (~4 GB de RAM), más piezas |
| **Astro CLI** | Desarrollo diario cómodo | `astro dev start`, tests y deploy integrados | Herramienta de un proveedor (Astronomer), aunque es gratuita |
| Helm en Kubernetes | Producción | Escalable | Complejo (ver [[16 - Executors, Escalado y Despliegue]]) |

> **Windows:** Airflow **no corre nativamente en Windows** (depende de funciones POSIX). Usa **WSL2** (Ubuntu) o Docker Desktop. Ver [[Docker/01 - Introducción y Arquitectura|Docker/01]].

---

## Opción 1: pip con constraints

Airflow tiene cientos de dependencias transitivas. Para que la instalación sea reproducible, el proyecto publica **archivos de constraints** por versión de Airflow y de Python. **Instala siempre con constraints.**

```bash
python -m venv .venv && source .venv/bin/activate      # ver Python/22 - Virtual Environments

AIRFLOW_VERSION=3.1.0
PYTHON_VERSION="$(python -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

# Con extras (providers) en la misma instalación
pip install "apache-airflow[postgres,amazon,cncf.kubernetes]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

# Providers sueltos después (sin romper la base)
pip install apache-airflow-providers-slack --constraint "${CONSTRAINT_URL}"
```

Ajusta `AIRFLOW_VERSION` a la versión 3.x vigente. Sobre gestores de paquetes, ver [[20 - Package Managers]] y [[22 - Virtual Environments]]. Con **uv**: `uv pip install "apache-airflow==3.1.0" --constraint "$CONSTRAINT_URL"`.

### `airflow standalone`

```bash
export AIRFLOW_HOME=~/airflow          # por defecto ~/airflow
airflow standalone
```

Este comando inicializa la base (SQLite), crea un usuario administrador y levanta **en un solo proceso** el api-server, el scheduler, el dag-processor y el triggerer. La UI queda en `http://localhost:8080`. La contraseña generada se imprime en consola y se guarda en `$AIRFLOW_HOME/simple_auth_manager_passwords.json.generated`.

> `standalone` es **solo para desarrollo**: SQLite no soporta concurrencia real.

### Levantar componentes por separado

```bash
airflow db migrate                 # crea o actualiza el esquema de la metadata DB
airflow api-server --port 8080
airflow scheduler
airflow dag-processor
airflow triggerer
```

---

## Opción 2: Docker Compose oficial

```bash
mkdir airflow-local && cd airflow-local
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.1.0/docker-compose.yaml'

mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env       # en Linux/WSL: evita problemas de permisos

docker compose up airflow-init               # migra la DB y crea el usuario airflow/airflow
docker compose up -d                         # levanta todo
docker compose ps
docker compose logs -f airflow-scheduler
docker compose down -v                       # apagar y borrar volúmenes (DB incluida)
```

Servicios que levanta: `postgres`, `redis`, `airflow-apiserver`, `airflow-scheduler`, `airflow-dag-processor`, `airflow-worker` (Celery), `airflow-triggerer` y, opcionalmente, `flower`.

### Añadir dependencias de Python

**No** uses `_PIP_ADDITIONAL_REQUIREMENTS` más que para pruebas, porque reinstala en cada arranque. Construye una imagen propia:

```dockerfile
# Dockerfile
FROM apache/airflow:3.1.0-python3.12
COPY requirements.txt /
RUN pip install --no-cache-dir -r /requirements.txt
```

```yaml
# docker-compose.yaml (dentro de x-airflow-common)
  # image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:3.1.0}
  build: .
```

```text
# requirements.txt (sin apache-airflow: ya viene en la imagen)
apache-airflow-providers-postgres
duckdb==1.4.*
dbt-duckdb
astronomer-cosmos
```

Ver [[Docker/08 - Docker Compose en Profundidad|Docker/08]] y [[Docker/02 - Dockerfile en Profundidad|Docker/02]].

---

## Opción 3: Astro CLI

La **Astro CLI** de Astronomer es la forma más cómoda de desarrollar en local. Usa Docker por debajo, pero oculta el compose.

```bash
# Instalar: winget install -e --id Astronomer.Astro   |   brew install astro   |   curl -sSL install.astronomer.io | sudo bash -s
astro dev init            # crea la estructura del proyecto
astro dev start           # levanta Airflow en Docker (UI en localhost:8080, admin/admin)
astro dev restart         # tras cambiar requirements.txt o el Dockerfile
astro dev parse           # verifica que los DAGs parsean sin errores
astro dev pytest          # corre tests/ dentro del contenedor
astro dev run dags list   # cualquier comando de la CLI de airflow
astro dev logs -s         # logs del scheduler
astro dev stop / kill
```

Estructura que genera:

```
mi-proyecto/
├── dags/                  # tus DAGs
│   └── exampledag.py
├── include/               # SQL, scripts y archivos auxiliares de los DAGs
├── plugins/               # plugins de Airflow
├── tests/dags/test_dag_example.py
├── Dockerfile             # FROM astrocrpublic.azurecr.io/runtime:3.x
├── requirements.txt       # paquetes pip
├── packages.txt           # paquetes apt del sistema
├── airflow_settings.yaml  # connections, variables y pools para local
└── .env                   # variables de entorno locales
```

---

## Estructura recomendada de un proyecto

```
airflow-proyecto/
├── dags/
│   ├── ventas/
│   │   ├── ventas_diarias.py
│   │   └── sql/
│   │       └── agregar_ventas.sql
│   └── ml/
│       └── reentreno_churn.py
├── include/                # lógica de negocio reutilizable (paquete Python normal)
│   └── mi_empresa/
│       ├── __init__.py
│       ├── extract.py
│       └── transform.py
├── plugins/
├── tests/
│   ├── test_dag_integrity.py
│   └── test_transform.py
├── requirements.txt
└── Dockerfile
```

> **Principio clave:** los archivos de DAGs deben ser **delgados**. Solo definen tareas y dependencias. La lógica de negocio va en módulos importables (`include/`) que puedes testear sin Airflow. Ver [[17 - Testing, Observabilidad y Buenas Prácticas]].

---

## Configuración

### `AIRFLOW_HOME` y `airflow.cfg`

- `AIRFLOW_HOME` (por defecto `~/airflow`) contiene `airflow.cfg`, la carpeta `dags/`, `logs/` y, en desarrollo, la base SQLite.
- `airflow.cfg` es un INI con secciones: `[core]`, `[database]`, `[scheduler]`, `[api]`, `[logging]`, `[celery]`, `[kubernetes_executor]`, `[secrets]`, etc.

### Variables de entorno `AIRFLOW__SECCION__CLAVE`

Cualquier opción se puede sobreescribir con una variable de entorno, con **doble guion bajo** como separador. En contenedores es la forma preferida:

```bash
export AIRFLOW__CORE__DAGS_FOLDER=/opt/airflow/dags
export AIRFLOW__CORE__LOAD_EXAMPLES=False
export AIRFLOW__CORE__EXECUTOR=LocalExecutor
export AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres/airflow
export AIRFLOW__CORE__PARALLELISM=32
export AIRFLOW__CORE__DEFAULT_TIMEZONE=America/Lima
export AIRFLOW__SCHEDULER__CATCHUP_BY_DEFAULT=False
export AIRFLOW__DAG_PROCESSOR__REFRESH_INTERVAL=60       # cada cuánto busca DAGs nuevos
export AIRFLOW__LOGGING__LOGGING_LEVEL=INFO

# Sufijo _CMD / _SECRET: leer el valor de un comando o de un secrets backend
export AIRFLOW__DATABASE__SQL_ALCHEMY_CONN_CMD="cat /run/secrets/sql_conn"
```

Orden de precedencia: **variable de entorno > `airflow.cfg` > valor por defecto**.

```bash
airflow config list                                 # toda la configuración efectiva
airflow config get-value core executor
airflow info                                        # versiones, paths, providers, executor
```

### Opciones más importantes

| Opción | Default | Nota |
|---|---|---|
| `core.executor` | `LocalExecutor` | Ver [[16 - Executors, Escalado y Despliegue]] |
| `core.parallelism` | 32 | Máximo de tareas corriendo a la vez **en toda la instalación** |
| `core.max_active_tasks_per_dag` | 16 | Por DAG |
| `core.max_active_runs_per_dag` | 16 | DAG Runs simultáneos por DAG |
| `core.load_examples` | True | Desactívalo para no ver decenas de DAGs de ejemplo |
| `core.default_timezone` | `utc` | La zona por defecto de los `start_date` sin tz |
| `scheduler.catchup_by_default` | False (3.x) | Ver [[10 - Scheduling, Timetables y Backfills]] |
| `core.dagbag_import_timeout` | 30 s | Un DAG que tarda más en importarse falla |
| `database.sql_alchemy_conn` | SQLite | **PostgreSQL** en cualquier entorno serio |

### Zona horaria

Airflow guarda todo en **UTC** internamente. Usa siempre `start_date` con zona horaria explícita con **pendulum**:

```python
import pendulum
start_date = pendulum.datetime(2026, 1, 1, tz="America/Lima")
```

Ver [[53 - Fechas y Zonas Horarias]].

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[03 - CLI de Airflow]]
- [[16 - Executors, Escalado y Despliegue]]: Helm y servicios gestionados
- [[Docker/08 - Docker Compose en Profundidad|Docker/08]]
- [[24-Configuracion-Profesional-de-Proyectos]] (en `Machine Learning/`): configuración por variables de entorno
