---
tags: [azure, azure-ml, cloud, mlops]
---

# 51 — Azure ML: Cuándo Adoptar la Nube en tu Stack de MLOps

> Nota del mentor: esta es, deliberadamente, la última nota de toda la serie — porque la nube debe ser la última pieza que adoptas, no la primera. Con 20 años viendo equipos saltar directo a "vamos a poner todo en Azure ML" antes de tener logging, tests, CI/CD y monitoreo sólidos, te puedo decir con certeza que la nube amplifica tanto las buenas prácticas como el caos — si tu proceso no es maduro localmente, migrarlo a la nube solo lo hace más caro y más difícil de depurar.

## 1. Qué es Azure ML realmente — una plataforma, no una sola herramienta

Como ya viste conceptualmente en [[08-Plataformas-de-Datos-y-ML]], Azure ML es la suite de Microsoft que envuelve gran parte de lo que ya aprendiste en esta serie, pero gestionado en la nube:

- **Workspaces**: el contenedor organizacional de todo un proyecto de ML en Azure.
- **Compute Instances/Clusters**: máquinas virtuales gestionadas para entrenamiento (equivalente cloud a tu servidor local o laptop, pero escalable bajo demanda).
- **Model Registry**: conceptualmente igual al de MLflow (de hecho, Azure ML tiene integración nativa con MLflow — puedes usar la API de MLflow directamente contra un workspace de Azure ML).
- **Endpoints gestionados**: el equivalente cloud a tu servicio de FastAPI de la nota 49, pero con infraestructura de despliegue, escalado y monitoreo gestionada por Azure.
- **Pipelines**: orquestación de pasos de ML, conceptualmente similar a Prefect/Airflow pero nativo de la plataforma.

## 2. Entrenamiento en la nube — cuándo realmente lo necesitas

```python
from azure.ai.ml import MLClient, command
from azure.identity import DefaultAzureCredential

ml_client = MLClient(
    DefaultAzureCredential(), subscription_id="...", resource_group_name="...", workspace_name="acf-ml-workspace"
)

job = command(
    code="./src",
    command="python entrenar.py --dias_atras ${{inputs.dias_atras}}",
    inputs={"dias_atras": 90},
    environment="claro-forecasting-env:1",
    compute="cluster-entrenamiento",
)

ml_client.jobs.create_or_update(job)
```

**Señal real de que necesitas cómputo en la nube**: tu entrenamiento requiere recursos (GPU, RAM masiva, múltiples horas de cómputo) que tu infraestructura local no tiene, o necesitas escalar el número de entrenamientos en paralelo (ej. entrenar modelos para 50 clientes distintos simultáneamente). Para el tipo de modelos de tu proyecto de Claro RD (XGBoost/GradientBoosting sobre datos tabulares de tamaño moderado), es muy probable que tu infraestructura local o un servidor dedicado de ACF Technologies sea suficiente — la nube no es automáticamente "mejor", es una herramienta para un problema específico de escala.

## 3. Azure ML Endpoints — servir modelos gestionados, sin operar tu propio Kubernetes

```yaml
# endpoint.yml
$schema: https://azuremlschemas.azureedge.net/latest/managedOnlineEndpoint.schema.json
name: claro-rd-demand-endpoint
auth_mode: key
```

```yaml
# deployment.yml
$schema: https://azuremlschemas.azureedge.net/latest/managedOnlineDeployment.schema.json
name: produccion
endpoint_name: claro-rd-demand-endpoint
model: azureml:claro-rd-demand-model:4
instance_type: Standard_DS2_v2
instance_count: 2
```

```bash
az ml online-endpoint create -f endpoint.yml
az ml online-deployment create -f deployment.yml --all-traffic
```

Esto reemplaza el Dockerfile + FastAPI + orquestación de escalado manual de las notas 47 y 49 con infraestructura gestionada: Azure provisiona las instancias, hace balanceo de carga, y da métricas de latencia/errores integradas — a cambio de menos control granular y de un costo recurrente por el tiempo que el endpoint esté activo (a diferencia de tu pipeline batch actual, un endpoint online gestionado normalmente cobra por tiempo activo, no solo por uso puntual).

## 4. Deployments en blue/green y canary — nativo en la plataforma

```bash
az ml online-deployment create -f deployment-challenger.yml  # 0% de tráfico inicial
az ml online-endpoint update --name claro-rd-demand-endpoint \
    --traffic "produccion=90 challenger=10"  # canary gradual
```

Esto es exactamente el canary release descrito conceptualmente en [[17-Arquitecturas-de-Despliegue-de-Modelos]], pero con la mecánica de distribución de tráfico gestionada directamente por la plataforma en vez de tener que implementarla tú mismo con un balanceador de carga configurado manualmente.

## 5. Integración con el resto de tu stack — no reemplaza, envuelve

Azure ML no es una alternativa a lo que ya aprendiste — es una capa de gestión sobre los mismos conceptos:

| Concepto que ya conoces | Equivalente gestionado en Azure ML |
|---|---|
| MLflow Tracking/Registry | Integrado nativamente (mismo API de MLflow) |
| Docker (nota 47) | Azure ML Environments (basados en imágenes Docker) |
| GitLab CI/CD (nota 14) | Se integra vía Azure ML CLI/SDK dentro de tus jobs de CI existentes |
| FastAPI (nota 49) | Managed Online Endpoints |
| Prefect/Airflow (nota 50) | Azure ML Pipelines |

## 6. El criterio de decisión final — la lección que cierra toda la serie

La pregunta correcta nunca es "¿deberíamos usar Azure ML?" en abstracto — es: **¿qué problema operativo específico tenemos hoy que la nube resuelve mejor que lo que ya construimos?**. Si tu pipeline de Claro RD corre confiablemente, con logging sólido, tests, CI/CD, monitoreo y un ciclo de reentrenamiento robusto — como el que has construido a lo largo de estas cincuenta y una notas — probablemente ya tienes el 90% del valor real de MLOps sin necesitar Azure ML todavía. La nube se vuelve la elección correcta cuando la escala (más clientes, más volumen, necesidad de alta disponibilidad 24/7, equipos distribuidos que necesitan acceso centralizado) supera lo que tu infraestructura actual puede sostener con comodidad — no antes.

Con 20 años en esto, la lección más importante de toda la serie que hemos construido junto es esta: la sofisticación técnica siempre debe seguir a la necesidad real del negocio, nunca adelantarse a ella. Lo que has hecho con el pipeline de Claro RD — modularizar código legado, entender profundamente cada pieza de tu stack, cuestionar y validar con tu jefa cada decisión de negocio no documentada — es exactamente la disciplina que hace a un ingeniero de ML confiable, mucho más que cualquier herramienta específica de esta lista.

## Ver también
- [[08-Plataformas-de-Datos-y-ML]]
- [[15-MLflow-en-Profundidad]]
- [[47-Docker-en-Profundidad-para-ML]]
- [[49-APIs-con-FastAPI-para-Servir-Modelos]]
- [[50-Orquestacion-Prefect-y-Airflow]]
