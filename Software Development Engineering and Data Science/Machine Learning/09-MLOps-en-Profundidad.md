---
tags: [mlops, devops, kubernetes, docker, cicd, monitoreo, git]
aliases: [MLOps, MLOps en Profundidad]
---
# Parte 9 — MLOps en Profundidad

> Continúa de [[08-Plataformas-de-Datos-y-ML]]. Esta es la nota que más se conecta directamente con tu trabajo diario como consultor de ingeniería. La idea central de MLOps ya se planteó en [[03-Arquitectura-Empresarial-de-Datos-y-ML]] (el ciclo despliegue→monitoreo→reentrenamiento) y en [[02-Roles-y-Carreras-en-DS-ML#4. MLOps Engineer]] (el rol); aquí bajamos a las herramientas concretas que hacen ese ciclo posible, herramienta por herramienta, y por qué cada una existe.

---

## La idea que conecta todo MLOps

MLOps es, en esencia, **aplicarle al ciclo de vida de un modelo de ML las mismas disciplinas que DevOps ya le aplicó al software tradicional**: control de versiones, automatización, pruebas, integración/despliegue continuo, observabilidad. La diferencia clave que hace a MLOps más difícil que DevOps puro: en software tradicional solo versionas y pruebas **código**; en ML tienes que versionar y probar **código + datos + el modelo resultante**, y el modelo puede "fallar en silencio" (degradarse sin lanzar un error) de una forma que el software tradicional no.

---

## Control de versiones

### Git

**Qué hace:** El sistema estándar de control de versiones de código — registra cada cambio, permite ramas (branches) para trabajar en paralelo sin pisarse, y es la base sobre la que se construye casi toda la automatización moderna (CI/CD).

**Rol específico en MLOps:** Versiona el código de tus pipelines, scripts de entrenamiento, definiciones de infraestructura — pero **no versiona bien datasets grandes ni modelos entrenados** (archivos binarios de GBs) de forma nativa y eficiente. De ahí nace la necesidad de herramientas complementarias (DVC — Data Version Control, o directamente el Model Registry de MLflow) específicamente para versionar datos y modelos, dejando a Git enfocado en lo que hace mejor: código.

---

## Contenedores y empaquetado

### Docker

**Qué hace:** Empaqueta una aplicación (o un modelo con todas sus dependencias exactas: versión de Python, librerías, sistema operativo base) en un **contenedor** — una unidad portable que corre exactamente igual sin importar en qué máquina se ejecute.

**Por qué es crítico específicamente para ML:** El problema de "en mi máquina funciona" es particularmente grave en ML porque los modelos dependen de versiones exactas de librerías (una versión distinta de scikit-learn o PyTorch puede cambiar sutilmente el comportamiento de un modelo). Docker elimina esa ambigüedad — el contenedor que entrenaste es, literalmente, el mismo que corre en producción.

---

## CI/CD (Integración Continua / Despliegue Continuo)

**Qué es:** Automatizar el proceso de probar y desplegar cambios de código (y en MLOps, también de modelos) cada vez que alguien hace un cambio — en vez de que un humano ejecute manualmente pruebas y despliegues paso a paso.

**Herramientas típicas:** GitHub Actions, GitLab CI, Jenkins, Azure DevOps Pipelines.

**Por qué existe (aplicado a ML — "CT", Continuous Training):** En MLOps, el concepto se extiende más allá del CI/CD tradicional de software a menudo llamado **CT (Continuous Training)** — no solo automatizas el despliegue del código que sirve al modelo, sino también el **reentrenamiento automático** cuando llegan datos nuevos o cuando el monitoreo detecta drift (cerrando el ciclo descrito en [[03-Arquitectura-Empresarial-de-Datos-y-ML#13. Reentrenamiento]]).

**Qué problema resuelve:** Sin CI/CD/CT, cada actualización de modelo o pipeline es un proceso manual, lento y propenso a errores humanos — y en un contexto como forecasting recurrente (tu caso con Claro RD), la falta de automatización significa literalmente alguien ejecutando el mismo proceso a mano cada vez que hay que actualizar el modelo.

---

## Versionado (de datos y modelos, más allá del código)

**Qué es:** Además de Git para código, en MLOps necesitas versionar **datasets** (¿con qué datos exactos se entrenó este modelo?) y **modelos** (¿cuál es la versión actual en producción, cuál la anterior?). Herramientas: DVC (Data Version Control, se integra con Git), y el **Model Registry** (ver siguiente sección y [[03-Arquitectura-Empresarial-de-Datos-y-ML#10. Registro del Modelo (Model Registry)]]).

**Por qué es un problema distinto al versionado de código:** Los datasets pueden pesar gigabytes/terabytes — Git no está diseñado para eso. Y a diferencia del código (donde "la versión 3" es inequívoca), un modelo depende de una combinación de código + datos + hiperparámetros + hardware/librerías, por lo que versionar "solo el código" no es suficiente para poder reproducir exactamente un modelo anterior.

---

## Model Registry

Ya cubierto en profundidad en [[03-Arquitectura-Empresarial-de-Datos-y-ML#10. Registro del Modelo (Model Registry)]] y [[07-Librerias-de-Data-Science-y-ML#MLflow]]. En el contexto de esta nota, vale la pena remarcar su rol como **la pieza que conecta versionado con despliegue**: el Model Registry es el punto de la cadena donde un modelo pasa de "experimento" a "candidato a producción", con estados formales (staging → producción → archivado) que un pipeline de CI/CD puede leer automáticamente para decidir qué desplegar.

---

## Monitoreo y Drift

Ya cubierto conceptualmente en [[03-Arquitectura-Empresarial-de-Datos-y-ML#12. Monitoreo]]. Aquí el detalle técnico de **qué tipos de drift existen** y con qué se detectan:

- **Data drift:** la distribución de las features de entrada cambió respecto a los datos de entrenamiento (ej. tu forecasting empezó a recibir un rango de valores de "volumen histórico" muy distinto al que vio en entrenamiento, por un cambio de negocio).
- **Concept drift:** la _relación_ entre las features y el target cambió, aunque las features en sí se vean parecidas (ej. el mismo nivel de "quejas del cliente" que antes predecía churn con 80% de confianza, ahora predice distinto porque cambió el contexto de mercado).
- **Model/performance drift:** la métrica de desempeño del modelo (accuracy, RMSE) medida contra resultados reales ya observados, cae por debajo de un umbral aceptable — la señal más directa, pero que requiere tener el "ground truth" disponible con cierta latencia.

**Herramientas:** Evidently (ver [[07-Librerias-de-Data-Science-y-ML#Evidently]]) para el análisis estadístico de drift; **Prometheus + Grafana** (ver abajo) para el monitoreo de infraestructura y métricas operacionales del servicio que expone el modelo.

---

## Reentrenamiento (Retraining)

Ya cubierto en [[03-Arquitectura-Empresarial-de-Datos-y-ML#13. Reentrenamiento]]. El detalle operativo que agrega esta nota: el reentrenamiento puede dispararse de tres formas — **programado** (cron/Airflow, ej. cada mes), **disparado por drift** (cuando Evidently u otra herramienta de monitoreo detecta que se cruzó un umbral), o **manual** (un humano decide que es momento, típicamente tras un evento de negocio conocido — ej. un cambio de política comercial que sabes de antemano que afectará los patrones históricos).

---

## Pipelines y Orquestación

**Apache Airflow** ya se cubrió en detalle en [[04-Ingenieria-de-Datos#Apache Airflow]] desde el ángulo de ingeniería de datos — en MLOps, el mismo Airflow (o alternativas como Kubeflow Pipelines, ver [[08-Plataformas-de-Datos-y-ML#Kubeflow]]) orquesta específicamente los pasos del ciclo de vida del modelo: extracción de datos frescos → cálculo de features → entrenamiento → validación → registro → despliegue, como un DAG con dependencias claras entre cada paso.

**Argo (Argo Workflows / Argo CD):** Un orquestador de flujos de trabajo **nativo de Kubernetes** — Argo Workflows ejecuta pipelines como una secuencia de contenedores Docker corriendo en Kubernetes, y Argo CD implementa **GitOps** (el estado deseado de la infraestructura/despliegue vive declarado en un repositorio Git, y Argo CD se encarga de que el clúster siempre converja a ese estado). Por qué existe en el contexto de MLOps sobre Kubernetes: cuando toda tu infraestructura de despliegue ya vive en Kubernetes (como suele pasar en organizaciones que adoptaron Kubeflow), Argo se integra de forma más nativa que Airflow, que fue diseñado originalmente pensando en pipelines de datos batch, no específicamente en despliegues continuos de contenedores.

---

## Testing (aplicado a ML — más allá del testing de software tradicional)

**Qué se prueba en un pipeline de ML, más allá del código:**

- **Tests de datos:** ¿los datos de entrada cumplen las expectativas de calidad? (Great Expectations, ver [[07-Librerias-de-Data-Science-y-ML#Great Expectations]]).
- **Tests del modelo:** ¿el modelo nuevo tiene un desempeño al menos igual de bueno que el modelo actual en producción, sobre un conjunto de datos de referencia fijo (_regression testing_ de modelos)?
- **Tests de comportamiento/sesgo:** ¿el modelo se comporta de forma consistente y justa entre distintos subgrupos de datos (ej. no discrimina sistemáticamente contra un segmento particular)?
- **Tests de infraestructura:** ¿el endpoint de inferencia responde dentro del límite de latencia aceptable bajo carga?

**Por qué esto es más complejo que testing de software tradicional:** El software tradicional tiene un comportamiento determinístico que puedes probar con asserts exactos ("la función devuelve 4"). Un modelo de ML es probabilístico — no puedes exigir "el modelo predice exactamente X", solo puedes exigir "el modelo predice dentro de un rango aceptable de error, sobre un conjunto de referencia conocido".

---

## Automatización y Orquestación (Síntesis)

Vale la pena remarcar la diferencia entre estos dos términos que se usan de forma intercambiable pero no son lo mismo:

- **Automatización:** eliminar un paso manual específico (ej. "el script de limpieza de datos corre solo cada noche" es automatización de un paso).
- **Orquestación:** coordinar **múltiples pasos automatizados entre sí**, respetando dependencias, reintentos, y manejo de fallos (ej. Airflow no solo ejecuta el script de limpieza, sino que sabe que el paso de entrenamiento no debe correr hasta que la limpieza termine bien, y qué hacer si algo falla a la mitad).

---

## Monitoreo de infraestructura: Prometheus y Grafana

### Prometheus

**Qué hace:** Un sistema de monitoreo y alertas que **recolecta métricas numéricas a lo largo del tiempo** (series de tiempo) de cualquier sistema que las exponga — CPU, memoria, latencia de peticiones, tasa de errores, número de peticiones por segundo a tu endpoint de inferencia.

**Por qué existe en el contexto de ML:** Un modelo desplegado como servicio (API) es, ante todo, un servicio de software — necesita el mismo tipo de monitoreo operacional (¿está caído? ¿está lento? ¿está recibiendo tráfico anormal?) que cualquier otro microservicio, independientemente de si sus predicciones son buenas o malas (eso lo cubre Evidently, no Prometheus).

### Grafana

**Qué hace:** La capa de **visualización** que típicamente se conecta a Prometheus (y a muchas otras fuentes de datos) para construir dashboards en tiempo real de esas métricas.

**Distinción clave:** Prometheus + Grafana monitorean la **salud operacional del sistema** que sirve al modelo (infraestructura); Evidently monitorea la **salud estadística de las predicciones** del modelo (drift, calidad). Un equipo de MLOps maduro necesita ambos tipos de monitoreo, porque un modelo puede estar "sano" operacionalmente (responde rápido, sin errores) mientras sus predicciones se degradan silenciosamente, o viceversa.

---

## Kubernetes

**Qué hace:** Un orquestador de contenedores — automatiza el despliegue, escalado y gestión de aplicaciones empaquetadas en contenedores Docker a través de un cluster de máquinas, manejando automáticamente cosas como "si un contenedor se cae, levanta uno nuevo" o "si hay mucho tráfico, agrega más réplicas".

**Rol en MLOps:** Es la infraestructura subyacente sobre la que corren muchas plataformas de ML modernas (Kubeflow está construido explícitamente sobre Kubernetes) y sobre la que se despliegan los endpoints de inferencia en producción a escala — permite que un servicio de modelo escale automáticamente según la demanda real, sin que un humano tenga que aprovisionar servidores manualmente.

---

## El flujo de herramientas MLOps de punta a punta

Uniendo todo lo anterior en el flujo real:

```
1. Código y configuración → Git
2. Empaquetado reproducible → Docker
3. Orquestación del pipeline completo → Airflow / Kubeflow / Argo
4. Entrenamiento + tracking de experimentos → MLflow / W&B
5. Versionado y registro del modelo resultante → MLflow Model Registry
6. Validación de datos y del modelo (testing) → Great Expectations + tests de regresión
7. Automatización del despliegue → CI/CD (GitHub Actions/GitLab CI) + Argo CD
8. Infraestructura de servicio en producción → Kubernetes
9. Monitoreo operacional → Prometheus + Grafana
10. Monitoreo estadístico del modelo → Evidently
11. Disparo de reentrenamiento (drift detectado o programado) → vuelve al paso 1
```

Este es, en esencia, el mismo ciclo de la sección 13 de [[01-Fundamentos-y-Panorama-General]] y de [[03-Arquitectura-Empresarial-de-Datos-y-ML]], ahora con el nombre concreto de la herramienta que ejecuta cada paso en una instalación MLOps madura.

---

## Ver también

- [[03-Arquitectura-Empresarial-de-Datos-y-ML]]
- [[02-Roles-y-Carreras-en-DS-ML]]
- [[07-Librerias-de-Data-Science-y-ML]]
- [[08-Plataformas-de-Datos-y-ML]]