---
tags: [orquestacion, prefect, airflow, mlops]
---

# 50 — Orquestación de Pipelines: Prefect y Airflow

> Nota del mentor: hoy tu pipeline probablemente corre vía un scheduler simple (Task Scheduler de Windows, un cron, o un Pipeline Schedule de GitLab CI). Eso funciona para un pipeline con pocas dependencias, pero en cuanto tienes múltiples pasos con dependencias complejas entre sí, reintentos condicionales, y necesitas visibilidad de qué falló y por qué, necesitas un orquestador real. Aquí es donde entran Prefect y Airflow.

## 1. Qué resuelve un orquestador que un cron simple no resuelve

Un cron ejecuta un script en un horario fijo, punto — no sabe si el paso anterior falló, no reintenta inteligentemente, no te da visibilidad de qué está corriendo ahora ni un historial navegable de ejecuciones pasadas, y no expresa dependencias entre pasos (¿debe correr `entrenar_modelo` solo si `validar_datos` tuvo éxito?). Un orquestador moderno resuelve las cuatro cosas.

## 2. Prefect — el orquestador moderno, "Pythonic" por diseño

```python
from prefect import flow, task
from prefect.tasks import task_input_hash
from datetime import timedelta

@task(retries=3, retry_delay_seconds=30)
def obtener_datos_historicos(office_id: int):
    # decorador declarativo: reintentos automáticos sin código manual de try/except
    return repositorio.obtener_historico(office_id)

@task
def validar_datos(datos):
    validate_input(datos)  # ver nota 13 — Pandera/Great Expectations
    return datos

@task
def entrenar_modelo(datos):
    modelo = XGBRegressor().fit(datos[features], datos[target])
    return modelo

@task
def evaluar_y_promover(modelo, datos_validacion):
    metricas = evaluar(modelo, datos_validacion)
    if supera_gate(metricas):
        promover_a_produccion(modelo)
    return metricas

@flow(name="pipeline-forecast-claro-rd")
def pipeline_forecast(office_id: int):
    datos = obtener_datos_historicos(office_id)
    datos_validados = validar_datos(datos)
    modelo = entrenar_modelo(datos_validados)
    metricas = evaluar_y_promover(modelo, datos_validados)
    return metricas

if __name__ == "__main__":
    pipeline_forecast(office_id=145)
```

La diferencia filosófica clave frente a Airflow (siguiente sección): un flow de Prefect es **código Python normal con decoradores**, no una configuración declarativa separada — puedes usar `if`/`for`/lógica condicional directamente dentro del flow, algo mucho menos natural en Airflow tradicional.

```python
from prefect.deployments import Deployment
from prefect.server.schemas.schedules import CronSchedule

deployment = Deployment.build_from_flow(
    flow=pipeline_forecast,
    name="produccion-claro-rd",
    schedule=CronSchedule(cron="*/30 * * * *"),  # cada 30 minutos
)
deployment.apply()
```

La UI de Prefect te da visibilidad de cada ejecución, cuánto tardó cada `task`, cuáles fallaron y por qué (con el traceback completo), y permite reintentar manualmente desde el punto de fallo — exactamente el tipo de observabilidad operativa que probablemente te falta hoy con un scheduler simple.

## 3. Airflow — el estándar histórico de la industria, basado en DAGs explícitos

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    "retries": 3,
    "retry_delay": timedelta(minutes=1),
}

with DAG(
    dag_id="pipeline_forecast_claro_rd",
    schedule_interval="*/30 * * * *",
    start_date=datetime(2026, 1, 1),
    default_args=default_args,
    catchup=False,
) as dag:

    obtener_datos = PythonOperator(
        task_id="obtener_datos_historicos",
        python_callable=obtener_datos_historicos,
        op_kwargs={"office_id": 145},
    )

    validar = PythonOperator(task_id="validar_datos", python_callable=validar_datos)
    entrenar = PythonOperator(task_id="entrenar_modelo", python_callable=entrenar_modelo)
    evaluar = PythonOperator(task_id="evaluar_y_promover", python_callable=evaluar_y_promover)

    obtener_datos >> validar >> entrenar >> evaluar  # define el DAG explícitamente con >>
```

El operador `>>` define explícitamente las dependencias entre tasks — Airflow construye un **DAG (grafo acíclico dirigido)** visible en su UI, mostrando exactamente qué depende de qué, y puede paralelizar automáticamente tasks que no tienen dependencia entre sí.

### Conceptos específicos de Airflow que debes conocer

- **`catchup=False`**: sin esto, si el scheduler estuvo apagado y se reactiva, Airflow intenta ejecutar **todas** las corridas que se "perdieron" retroactivamente desde `start_date` — casi nunca es lo que quieres en un pipeline de forecasting donde solo te importa el dato más reciente.
- **XComs**: el mecanismo de Airflow para pasar pequeños datos entre tasks (no está pensado para pasar DataFrames completos — para eso, cada task debería escribir a un almacenamiento intermedio como Parquet o una tabla de staging, y el siguiente task leer de ahí).
- **Operators**: Airflow tiene operators pre-construidos para decenas de integraciones (`SqlOperator`, `DockerOperator`, `KubernetesPodOperator`) — mucho del ecosistema de Airflow es exactamente ese catálogo de integraciones listas para usar.

## 4. Prefect vs. Airflow — cómo elegir con criterio

| | Prefect | Airflow |
|---|---|---|
| Filosofía | Código Python nativo con decoradores | DAG declarativo explícito |
| Curva de aprendizaje | Más baja si ya sabes Python | Más alta, vocabulario propio (DAG, Operator, XCom) |
| Lógica condicional dentro del flow | Natural (`if`/`for` normales) | Requiere patrones específicos (branching operators) |
| Madurez y ecosistema | Más joven, creciendo rápido | Extremadamente maduro, enorme catálogo de integraciones |
| Adopción en la industria | Creciendo, popular en equipos nuevos de ML | El estándar histórico, muy común en empresas grandes ya establecidas |

**Criterio práctico**: si empiezas un proyecto de orquestación desde cero hoy y el equipo es principalmente de ML/Python (como el tuyo), Prefect suele ser más rápido de adoptar por su sintaxis nativa de Python. Si te integras a una organización que ya tiene infraestructura de Airflow establecida (muy común en empresas con equipos de datos maduros), aprender Airflow es la elección pragmática — el conocimiento de "cómo pensar en pipelines orquestados con dependencias" se transfiere igual entre ambos.

## 5. Conexión con lo que ya tienes

Ninguna de estas herramientas reemplaza lo que ya construiste (tu sistema de autoaprendizaje, el gate de MAE/RMSE) — orquestan **cuándo y en qué orden** se ejecutan esos pasos, con reintentos, visibilidad y manejo de dependencias formales, en vez de que todo viva implícito en un solo script largo ejecutado por un scheduler simple. Es el siguiente nivel de madurez operativa sobre la base sólida de logging ([[11-Logging-en-Python-para-ML]]), manejo de errores ([[23-Manejo-Profesional-de-Errores]]) y testing ([[13-Testing-en-Machine-Learning]]) que ya construiste.

## Ver también
- [[14-CICD-para-ML-con-GitLab]]
- [[11-Logging-en-Python-para-ML]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]
- [[44-Ingesta-Incremental-CDC-e-Idempotencia]]
