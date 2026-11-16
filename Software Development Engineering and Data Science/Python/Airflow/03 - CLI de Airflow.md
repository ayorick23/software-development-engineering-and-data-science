---
tags: [airflow, cli, debugging, cheat-sheet]
---

# 03 — CLI de Airflow

> Continúa de [[02 - Instalación y Entorno Local]]. La CLI `airflow` sirve para administrar la instalación, **probar DAGs y tareas sin pasar por el scheduler**, gestionar connections, variables y pools, y depurar. Si usas Docker Compose, antepón `docker compose exec airflow-scheduler`; con Astro, `astro dev run`.

```bash
airflow --help
airflow dags --help
airflow cheat-sheet          # resumen de todos los comandos
airflow version
airflow info                 # versión, executor, paths, providers instalados
```

---

## DAGs

```bash
airflow dags list                                   # DAGs parseados
airflow dags list -o json                           # salida: table | json | yaml | plain
airflow dags list-import-errors                     # ⭐ DAGs que NO se pudieron importar y por qué
airflow dags show ventas_diarias                    # dependencias en texto (o --save grafo.png)
airflow dags details ventas_diarias

airflow dags pause ventas_diarias
airflow dags unpause ventas_diarias                 # los DAGs nuevos arrancan pausados por defecto

# Disparar un DAG Run manual (con configuración)
airflow dags trigger ventas_diarias
airflow dags trigger ventas_diarias --conf '{"region": "LATAM", "full_refresh": true}'
airflow dags trigger ventas_diarias --logical-date 2026-11-01T00:00:00

airflow dags list-runs ventas_diarias               # historial de ejecuciones
airflow dags state ventas_diarias 2026-11-14T05:00:00+00:00
airflow dags next-execution ventas_diarias

airflow dags reserialize                            # forzar re-parseo de todos los DAGs
airflow dags delete ventas_diarias                  # borra la metadata (no el archivo)
```

### ⭐ `airflow dags test`: ejecutar un DAG completo en local

Ejecuta **todas las tareas del DAG en un solo proceso**, sin scheduler ni executor, y muestra los logs en la terminal. Es la herramienta de depuración más útil.

```bash
airflow dags test ventas_diarias 2026-11-14
airflow dags test ventas_diarias 2026-11-14 --conf '{"region": "EMEA"}'
```

Lo mismo desde Python, ideal para depurar con breakpoints en VS Code:

```python
# al final de dags/ventas_diarias.py
dag_obj = ventas_diarias()

if __name__ == "__main__":
    dag_obj.test(logical_date=pendulum.datetime(2026, 11, 14, tz="UTC"))
```

```bash
python dags/ventas_diarias.py        # corre el DAG; puedes poner breakpoints
```

Ver [[12 - Debugging]] (en `Python/`).

---

## Tareas

```bash
airflow tasks list ventas_diarias
airflow tasks list ventas_diarias --tree

# ⭐ Probar UNA tarea aislada (no guarda estado ni respeta dependencias)
airflow tasks test ventas_diarias transformar 2026-11-14
airflow tasks test ventas_diarias transformar 2026-11-14 --task-params '{"x": 1}'

# Ver cómo quedan los templates Jinja renderizados
airflow tasks render ventas_diarias cargar_sql 2026-11-14

airflow tasks state ventas_diarias transformar 2026-11-14
airflow tasks states-for-dag-run ventas_diarias manual__2026-11-14T10:00:00+00:00

# Limpiar (clear) estados para volver a ejecutar
airflow tasks clear ventas_diarias -t "transformar|cargar" -s 2026-11-01 -e 2026-11-14
airflow tasks clear ventas_diarias --only-failed -s 2026-11-01 -e 2026-11-14 --yes
airflow tasks clear ventas_diarias -t transformar --downstream -s 2026-11-14 -e 2026-11-14
```

| Comando | Guarda estado en DB | Respeta dependencias | Uso |
|---|---|---|---|
| `airflow tasks test` | No | No | Probar la lógica de una tarea |
| `airflow dags test` | Sí (run local) | Sí | Probar el DAG completo |
| `airflow tasks clear` | Sí | Sí (las reencola el scheduler) | Reejecutar en el entorno real |

---

## Backfill

En Airflow 3 el backfill lo **gestiona el scheduler**: la CLI solo crea la petición. Ver [[10 - Scheduling, Timetables y Backfills#Backfills]].

```bash
airflow backfill create --dag-id ventas_diarias \
    --from-date 2026-10-01 --to-date 2026-10-31 \
    --max-active-runs 3 \
    --reprocess-behavior failed        # none | failed | completed
airflow backfill create --dag-id ventas_diarias --from-date 2026-10-01 --to-date 2026-10-31 --dry-run
```

---

## Connections, Variables y Pools

```bash
# Connections
airflow connections list
airflow connections get postgres_dwh
airflow connections add postgres_dwh \
    --conn-uri 'postgresql://user:pass@host:5432/dwh'
airflow connections add s3_lake --conn-type aws \
    --conn-extra '{"region_name": "us-east-1"}'
airflow connections add api_crm --conn-json '{"conn_type": "http", "host": "https://api.crm.com", "password": "token"}'
airflow connections test postgres_dwh        # requiere habilitar test_connection en la config
airflow connections export conns.json
airflow connections import conns.json
airflow connections delete postgres_dwh

# Variables
airflow variables set umbral_mae 12.5
airflow variables set config_ventas '{"regiones": ["LATAM","EMEA"]}'
airflow variables get config_ventas
airflow variables list
airflow variables import variables.json
airflow variables export variables.json
airflow variables delete umbral_mae

# Pools
airflow pools list
airflow pools set duckdb_writer 1 "Un solo escritor sobre analitica.duckdb"
airflow pools import pools.json
airflow pools delete duckdb_writer
```

Más detalle en [[13 - Connections, Variables, Hooks y Providers]].

---

## Base de datos y administración

```bash
airflow db migrate                    # crear o actualizar el esquema (antes: db upgrade / db init)
airflow db check                      # ¿hay conexión con la DB?
airflow db check-migrations
airflow db clean --clean-before-timestamp '2026-08-01' --yes   # purgar historial antiguo
airflow db downgrade --to-version 2.10.5                       # solo en emergencias
airflow db shell                      # consola SQL de la metadata DB

airflow jobs check --job-type SchedulerJob     # health check del scheduler (útil en K8s)
airflow providers list
airflow plugins
airflow config list
airflow rotate-fernet-key             # rotar la clave que cifra passwords de connections
```

> **`db clean`**: la metadata DB crece sin límite (task instances, logs de eventos, XComs). Prográmalo periódicamente; hay incluso DAGs de mantenimiento que lo ejecutan.

### Usuarios

En Airflow 3 la autenticación depende del **auth manager** configurado:

- **SimpleAuthManager** (por defecto en 3.x): usuarios definidos en la configuración (`[core] simple_auth_manager_users = admin:admin,viewer:viewer`). Pensado para desarrollo.
- **FAB auth manager** (provider `fab`): el modelo clásico de 2.x con usuarios y roles en la DB.

```bash
# Con FAB auth manager
airflow users create --username ana --firstname Ana --lastname P \
    --role Admin --email ana@empresa.com --password '...'
airflow users list
```

En producción se integra con SSO (OAuth/LDAP) o lo gestiona el servicio (MWAA, Composer, Astro).

---

## Procesos (servidores)

```bash
airflow api-server --port 8080 --workers 4
airflow scheduler
airflow dag-processor
airflow triggerer
airflow celery worker --queues default,ml
airflow celery flower                    # monitor de Celery
airflow standalone                       # todo junto (dev)
```

## API REST y `airflowctl`

Airflow 3 expone una **REST API pública estable** (`/api/v2/...`), documentada con OpenAPI en `http://localhost:8080/docs`:

```bash
# Obtener un token (SimpleAuthManager / FAB)
TOKEN=$(curl -s -X POST localhost:8080/auth/token \
  -H 'Content-Type: application/json' -d '{"username":"admin","password":"admin"}' | jq -r .access_token)

# Disparar un DAG Run desde otro sistema
curl -X POST localhost:8080/api/v2/dags/ventas_diarias/dagRuns \
  -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  -d '{"logical_date": null, "conf": {"region": "LATAM"}}'
```

**`airflowctl`** (paquete `apache-airflow-ctl`) es la nueva CLI **remota**: opera contra la API de un Airflow desplegado sin necesitar acceso a la metadata DB, a diferencia de la CLI `airflow`.

## Recetas de depuración

| Problema | Comando |
|---|---|
| "Mi DAG no aparece en la UI" | `airflow dags list-import-errors` y `airflow dags reserialize` |
| "¿Qué SQL se va a ejecutar?" | `airflow tasks render <dag> <task> <fecha>` |
| "Quiero probar una tarea rápido" | `airflow tasks test <dag> <task> <fecha>` |
| "Quiero correr el DAG entero con breakpoints" | `dag.test()` desde Python |
| "Fallaron 30 días: reintentar solo los fallidos" | `airflow tasks clear <dag> --only-failed -s ... -e ...` |
| "¿Está vivo el scheduler?" | `airflow jobs check --job-type SchedulerJob` |
| "¿Qué configuración está usando realmente?" | `airflow config get-value <sección> <clave>` |

## Ver también

- [[02 - Instalación y Entorno Local]]
- [[04 - DAGs - Definición y Parámetros]]
- [[10 - Scheduling, Timetables y Backfills]]
- [[13 - Connections, Variables, Hooks y Providers]]
- [[17 - Testing, Observabilidad y Buenas Prácticas]]
- [[Git/Remote|Git]]: la otra CLI que usarás a diario con tus DAGs
