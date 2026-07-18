---
tags: [plataformas, mlops, cloud, azure, aws, databricks]
aliases: [Plataformas de ML, Plataformas Empresariales de Datos]
---
# Parte 8 — Plataformas: Por Qué Existen, Qué Hacen, Cuándo Valen la Pena

> Continúa de [[07-Librerias-de-Data-Science-y-ML]]. La diferencia clave entre una **librería** (parte 7) y una **plataforma** (esta parte): una librería es una pieza que tú integras en tu propio código/infraestructura; una plataforma es un **producto completo** que empaqueta infraestructura, interfaz, gobernanza y (a menudo) varias librerías por debajo, para que un equipo no tenga que ensamblar todo desde cero.

---

## Tabla de Contenidos

1. [[#La pregunta central de esta nota|La pregunta central de esta nota]]
2. [[#Plataformas de ML en la Nube (Cloud-Native)|Plataformas de ML en la Nube (Cloud-Native)]]
	1. [[#Plataformas de ML en la Nube (Cloud-Native)#Azure ML (Azure Machine Learning)|Azure ML (Azure Machine Learning)]]
	2. [[#Plataformas de ML en la Nube (Cloud-Native)#AWS SageMaker|AWS SageMaker]]
	3. [[#Plataformas de ML en la Nube (Cloud-Native)#Vertex AI|Vertex AI]]
3. [[#Plataformas Unificadas de Datos + ML|Plataformas Unificadas de Datos + ML]]
	1. [[#Plataformas Unificadas de Datos + ML#Databricks|Databricks]]
	2. [[#Plataformas Unificadas de Datos + ML#Microsoft Fabric|Microsoft Fabric]]
	3. [[#Plataformas Unificadas de Datos + ML#Snowflake|Snowflake]]
4. [[#Plataformas Especializadas en Ciencia de Datos Colaborativa|Plataformas Especializadas en Ciencia de Datos Colaborativa]]
	1. [[#Plataformas Especializadas en Ciencia de Datos Colaborativa#Domino Data Lab|Domino Data Lab]]
	2. [[#Plataformas Especializadas en Ciencia de Datos Colaborativa#Dataiku|Dataiku]]
	3. [[#Plataformas Especializadas en Ciencia de Datos Colaborativa#H2O (H2O.ai)|H2O (H2O.ai)]]
	4. [[#Plataformas Especializadas en Ciencia de Datos Colaborativa#KNIME|KNIME]]
	5. [[#Plataformas Especializadas en Ciencia de Datos Colaborativa#RapidMiner|RapidMiner]]
5. [[#Plataformas de Orquestación de ML (open-source, "hazlo tú mismo")|Plataformas de Orquestación de ML (open-source, "hazlo tú mismo")]]
	1. [[#Plataformas de Orquestación de ML (open-source, "hazlo tú mismo")#MLflow|MLflow]]
	2. [[#Plataformas de Orquestación de ML (open-source, "hazlo tú mismo")#Kubeflow|Kubeflow]]
6. [[#Cómo Decidir entre "plataforma gestionada" vs. "ensamblar tú mismo" (open-source)|Cómo Decidir entre "plataforma gestionada" vs. "ensamblar tú mismo" (open-source)]]
7. [[#Ver también|Ver también]]

---
## La pregunta central de esta nota

Para cada plataforma, la pregunta no es solo "¿qué hace?" sino: **¿qué le ahorra a la empresa construir por sí misma, y a qué costo (dinero, flexibilidad, dependencia del proveedor)?** Toda plataforma es, en el fondo, un trade-off entre velocidad/comodidad y control/costo a largo plazo.

---

## Plataformas de ML en la Nube (Cloud-Native)

### Azure ML (Azure Machine Learning)

**Qué hace:** La plataforma de Microsoft Azure para todo el ciclo de vida de ML — notebooks gestionados, entrenamiento distribuido, AutoML (entrenamiento automático probando múltiples algoritmos), Model Registry, endpoints de despliegue gestionados, pipelines de ML como código.

**Por qué existe:** Para que una empresa no tenga que ensamblar manualmente infraestructura de cómputo, almacenamiento, control de acceso y monitoreo para sus proyectos de ML — todo integrado con el resto del ecosistema Azure (Azure AD para permisos, Synapse/Fabric para datos, Azure DevOps para CI/CD).

**Cuándo vale la pena:** Cuando la empresa ya vive en el ecosistema Microsoft/Azure (muy común en banca, telecom y gobierno en LatAm) y quiere minimizar la cantidad de proveedores e integraciones distintas que administra.

### AWS SageMaker

**Qué hace:** El equivalente de Amazon Web Services — notebooks gestionados, entrenamiento y tuning de hiperparámetros distribuido, Model Registry, despliegue de endpoints (tiempo real o batch), SageMaker Pipelines para orquestación, y una gama muy amplia de herramientas especializadas (SageMaker Ground Truth para etiquetado de datos, SageMaker Clarify para explicabilidad/sesgo).

**Cuándo vale la pena:** Cuando la empresa ya está fuertemente comprometida con AWS — es, junto con Azure ML, de las plataformas más maduras y con mayor variedad de servicios especializados del mercado, a costa de una curva de aprendizaje considerable por la cantidad de servicios distintos que hay que entender.

### Vertex AI

**Qué hace:** La plataforma de Google Cloud — unifica AutoML, notebooks, entrenamiento personalizado, Model Registry y despliegue, con fuerte integración con BigQuery (puedes entrenar modelos directamente sobre datos de BigQuery sin moverlos) y acceso privilegiado a los modelos de Google (familia Gemini) para casos de IA generativa.

**Cuándo vale la pena:** Cuando la empresa ya usa BigQuery/Google Cloud, o cuando el caso de uso prioritario involucra modelos de Google o AutoML con mínima fricción de configuración.

---

## Plataformas Unificadas de Datos + ML

### Databricks

**Qué hace:** Ya cubierta en detalle en [[04-Ingenieria-de-Datos#Databricks]] desde el ángulo de datos — desde el ángulo de ML, Databricks agrega notebooks colaborativos, MLflow nativo integrado, entrenamiento distribuido con Spark, y AutoML, todo operando directamente sobre los datos del Lakehouse sin necesidad de exportarlos a otro sistema.

**Cuándo vale la pena:** Cuando el equipo de datos y el equipo de ML necesitan trabajar sobre **la misma copia de los datos** sin fricciones de exportación/sincronización, y el volumen de datos justifica el uso de Spark por debajo.

### Microsoft Fabric

**Qué hace:** Ya cubierta en [[04-Ingenieria-de-Datos#Microsoft Fabric]] — su componente de Data Science (basado en notebooks con Spark) agrega experiment tracking (integración con MLflow) y despliegue de modelos, dentro del mismo entorno donde ya viven los datos y los reportes de Power BI.

**Cuándo vale la pena:** Similar a Databricks pero para organizaciones ya profundamente integradas en el ecosistema Microsoft, priorizando una sola plataforma "todo en uno" sobre la profundidad técnica especializada.

### Snowflake

**Qué hace:** Aunque nació como Data Warehouse (ver [[04-Ingenieria-de-Datos#Snowflake]]), ha ido incorporando capacidades de ML (Snowpark ML) que permiten entrenar y desplegar modelos **directamente dentro del warehouse**, sin mover los datos a una plataforma externa.

**Cuándo vale la pena:** Cuando la empresa ya tiene sus datos consolidados en Snowflake y los casos de uso de ML son relativamente directos (no requieren Deep Learning pesado) — evita la fricción y el costo de mover datos hacia y desde una plataforma de ML separada.

---

## Plataformas Especializadas en Ciencia de Datos Colaborativa

### Domino Data Lab

**Qué hace:** Una plataforma diseñada específicamente para que **equipos grandes de científicos de datos** trabajen de forma colaborativa y gobernada — entornos de trabajo reproducibles (contenedores estandarizados), control de versiones de experimentos, y gobernanza centralizada de qué herramientas/paquetes puede usar cada equipo, agnóstica de la nube subyacente.

**Por qué existe:** En organizaciones grandes con docenas o cientos de científicos de datos, sin una plataforma central, cada uno termina con su propio entorno configurado a mano de forma distinta ("en mi máquina funciona") — Domino estandariza esto sin forzar a todos a usar el mismo lenguaje o librería específica.

**Cuándo vale la pena:** Empresas grandes (bancos, farmacéuticas, aseguradoras) con equipos numerosos de ciencia de datos que necesitan gobernanza y reproducibilidad estricta, pero que quieren mantenerse agnósticas de un solo proveedor cloud.

### Dataiku

**Qué hace:** Una plataforma de ciencia de datos con fuerte enfoque en **accesibilidad** — interfaz visual de arrastrar y soltar para construir pipelines de datos y modelos de ML, pensada para que analistas con menos experiencia en código puedan participar del proceso junto a científicos de datos que sí programan (modelo "colaborativo" entre perfiles técnicos y de negocio).

**Cuándo vale la pena:** Organizaciones donde se quiere democratizar la ciencia de datos más allá del equipo técnico especializado — permitir que analistas de negocio construyan y mantengan flujos de análisis/modelos simples sin depender 100% de un Data Scientist para cada iteración.

### H2O (H2O.ai)

**Qué hace:** Una plataforma/librería open-source con fuerte foco en **AutoML** — dado un dataset y un objetivo, prueba y ajusta automáticamente docenas de algoritmos y combinaciones, entregando el mejor modelo posible con mínima intervención manual. Tiene versión open-source (H2O-3) y una plataforma empresarial más completa (H2O AI Cloud).

**Cuándo vale la pena:** Cuando se necesita generar modelos baseline de alta calidad rápidamente, o cuando el equipo quiere automatizar la exploración de algoritmos/hiperparámetros como primer paso antes de refinar manualmente.

### KNIME

**Qué hace:** Una plataforma **visual, de arrastrar y soltar (low-code/no-code)** para construir flujos de análisis de datos y ML — cada paso del pipeline (cargar datos, limpiar, transformar, entrenar, evaluar) es un bloque visual conectado a los demás, open-source en su núcleo.

**Cuándo vale la pena:** Equipos con analistas que entienden bien el negocio y el proceso analítico pero no necesariamente programan con fluidez — o para prototipar rápido flujos de análisis antes de formalizarlos en código para producción.

### RapidMiner

**Qué hace:** Muy similar en propósito a KNIME — plataforma visual low-code para ciencia de datos, con fuerte foco histórico en minería de datos y analítica predictiva empresarial, orientada a facilitar que equipos de negocio construyan modelos sin escribir mucho código.

**Cuándo vale la pena:** Contextos similares a KNIME/Dataiku — organizaciones que priorizan la velocidad de adopción por parte de perfiles no puramente técnicos por encima del control fino que da el código.

---

## Plataformas de Orquestación de ML (open-source, "hazlo tú mismo")

### MLflow

Ya cubierta en detalle en [[07-Librerias-de-Data-Science-y-ML#MLflow]]. Vale la pena repetir aquí su posición en el panorama de plataformas: es la opción **open-source, ligera y agnóstica de nube** para experiment tracking y Model Registry — la eligen equipos que no quieren atarse a una plataforma cloud completa (SageMaker/Vertex/Azure ML) pero sí quieren algo más estructurado que "un Excel con los resultados de cada corrida".

### Kubeflow

**Qué hace:** Una plataforma open-source para ejecutar flujos de trabajo de ML **sobre Kubernetes** — pipelines de ML como código, entrenamiento distribuido, servicio de modelos, todo aprovechando la orquestación de contenedores de Kubernetes que la empresa probablemente ya usa para el resto de su software.

**Por qué existe:** Nace específicamente para que las empresas que ya invirtieron en Kubernetes como su estándar de infraestructura no tengan que adoptar una plataforma de ML completamente distinta y separada — extiende la misma infraestructura de contenedores que ya administran para correr también las cargas de trabajo de ML.

**Cuándo vale la pena:** Empresas con equipos de infraestructura maduros en Kubernetes que prefieren mantener todo su cómputo (aplicaciones y ML) bajo un mismo paradigma de orquestación, evitando depender de una plataforma cloud propietaria de ML. El costo: Kubeflow tiene una curva de aprendizaje y complejidad operativa considerablemente más alta que una plataforma gestionada tipo SageMaker.

---

## Cómo Decidir entre "plataforma gestionada" vs. "ensamblar tú mismo" (open-source)

Esta es, en la práctica, la decisión de fondo detrás de toda esta nota:

|Factor|Favorece plataforma gestionada (SageMaker, Vertex, Azure ML, Databricks)|Favorece open-source ensamblado (MLflow + Airflow + Kubeflow)|
|---|---|---|
|Velocidad de arranque|Alta — todo integrado desde el día uno|Baja — hay que ensamblar e integrar cada pieza|
|Costo a largo plazo|Puede ser alto (pricing por uso/asiento)|Menor en licencias, mayor en horas de ingeniería|
|Dependencia del proveedor (_lock-in_)|Alta|Baja — cada pieza es reemplazable|
|Equipo de infraestructura propio|No indispensable|Necesario y con experiencia real en la pila elegida|
|Gobernanza/compliance out-of-the-box|Fuerte, ya integrada|Hay que construirla|

**La heurística práctica:** equipos pequeños/medianos sin un equipo de plataforma/infraestructura dedicado casi siempre salen ganando con una plataforma gestionada, aunque cueste más por uso — el costo de ingeniería de ensamblar y mantener una pila open-source propia suele superar el ahorro en licencias. Empresas grandes con equipos de plataforma dedicados, alto volumen (donde el costo por uso escala mucho) y necesidad de evitar _lock-in_ de proveedor, tienden hacia el ensamblaje open-source.

---

## Ver también

- [[04-Ingenieria-de-Datos]]
- [[07-Librerias-de-Data-Science-y-ML]]
- [[09-MLOps-en-Profundidad]]