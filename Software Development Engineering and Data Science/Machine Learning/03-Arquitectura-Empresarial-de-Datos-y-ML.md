---
tags: [arquitectura-de-datos, mlops, data-engineering, sistemas]
aliases: [Arquitectura de Datos y ML, Flujo ERP a Reentrenamiento]
---
# Parte 3 — Arquitectura Empresarial Completa: de ERP a Reentrenamiento

> Esta nota recorre el flujo completo de cómo nace un dato en una empresa hasta que termina alimentando un modelo en producción — y por qué **cada eslabón existe**. Es la continuación natural de la sección 13 de [[01-Fundamentos-y-Panorama-General]], vista ahora a nivel de sistemas de empresa. Para el detalle técnico de las herramientas mencionadas aquí, ver [[04-Ingenieria-de-Datos]].

---

## Tabla de Contenidos

1. [[#El Flujo Completo|El Flujo Completo]]
2. [[#ERP (Enterprise Resource Planning)|ERP (Enterprise Resource Planning)]]
3. [[#CRM (Customer Relationship Management)|CRM (Customer Relationship Management)]]
4. [[#Bases de Datos (operacionales / transaccionales)|Bases de Datos (operacionales / transaccionales)]]
5. [[#ETL / ELT|ETL / ELT]]
6. [[#Data Warehouse|Data Warehouse]]
7. [[#Data Lake|Data Lake]]
8. [[#Lakehouse|Lakehouse]]
9. [[#Feature Store|Feature Store]]
10. [[#Entrenamiento|Entrenamiento]]
11. [[#Registro del Modelo (_Model Registry_)|Registro del Modelo (_Model Registry_)]]
12. [[#Despliegue (Deployment)|Despliegue (Deployment)]]
13. [[#Monitoreo|Monitoreo]]
14. [[#Reentrenamiento|Reentrenamiento]]
15. [[#La idea central de esta nota|La idea central de esta nota]]
16. [[#Ver también|Ver también]]

---

## El Flujo Completo

```
ERP → CRM → Bases de Datos → ETL → Data Warehouse → Data Lake → Lakehouse
  → Feature Store → Entrenamiento → Registro del Modelo → Despliegue
  → Monitoreo → Reentrenamiento (vuelve al ciclo)
```

Vamos eslabón por eslabón: qué es, por qué existe, y qué pasaría si no existiera.

---

## ERP (Enterprise Resource Planning)

**Qué es:** Un sistema que integra los procesos operativos centrales de una empresa — finanzas, inventario, compras, producción, recursos humanos — en una sola plataforma (ej. SAP, Oracle ERP, Microsoft Dynamics).

**Por qué existe:** Antes de los ERP, cada departamento (finanzas, inventario, RRHH) tenía su propio sistema aislado, y nadie tenía una vista unificada de la empresa. Reconciliar esa información a mano era lento y propenso a errores. El ERP centraliza la **operación** del negocio.

**Qué problema resuelve:** Fragmentación operativa. Sin ERP: "finanzas dice que tenemos X inventario, pero el almacén dice otra cosa".

**Rol en el flujo de datos:** Es una de las **fuentes primarias de datos transaccionales** — de aquí salen datos de ventas, compras, inventario, nómina.

---

## CRM (Customer Relationship Management)

**Qué es:** Sistema que gestiona la relación con clientes — contactos, historial de interacciones, oportunidades de venta, tickets de soporte (ej. Salesforce, HubSpot, Zoho).

**Por qué existe:** Antes del CRM, la información de clientes vivía en la cabeza de los vendedores o en hojas de Excel dispersas. Cuando un vendedor se iba de la empresa, se perdía todo el historial de la relación con sus clientes.

**Qué problema resuelve:** Pérdida de conocimiento sobre clientes y falta de visibilidad del embudo de ventas.

**Rol en el flujo de datos:** Otra fuente primaria — de aquí salen datos de clientes, interacciones, ventas, tickets, que son oro puro para modelos de churn, propensión de compra, segmentación.

---

## Bases de Datos (operacionales / transaccionales)

**Qué es:** Los sistemas de almacenamiento **detrás** del ERP, el CRM y cualquier aplicación (la app móvil, el sitio web, el sistema de facturación). Típicamente bases de datos relacionales (PostgreSQL, MySQL, SQL Server, Oracle) optimizadas para **transacciones** ([[Clase 01 - Fundamentos del Análisis de Datos#OLTP (Online Transaction Processing)|OLTP]] — Online Transaction Processing): escrituras rápidas, una fila a la vez, con integridad garantizada.

**Por qué existe:** Una aplicación necesita guardar y recuperar datos de forma confiable y rápida, fila por fila (ej. "guarda este pedido", "actualiza el saldo de esta cuenta").

**Qué problema resuelve:** Persistencia confiable de datos operacionales con garantías de integridad (transacciones [[Clase 01 - Fundamentos de las Bases de Datos NoSQL#ACID (Relacionales)|ACID]]).

**La limitación clave (y por qué nace el siguiente eslabón):** Estas bases están optimizadas para **muchas transacciones pequeñas**, no para **analizar millones de filas a la vez**. Si un analista corre una consulta pesada de análisis sobre la base de datos que usa la aplicación en producción, puede **ralentizar la aplicación real** para los usuarios reales. Esto crea la necesidad de sacar los datos de ahí y llevarlos a un sistema separado, pensado para análisis ([[Clase 01 - Fundamentos del Análisis de Datos#OLAP (Online Analytical Processing)|OLAP]]).

---

## ETL / ELT

**Qué es:** El proceso que **extrae** datos de las fuentes (ERP, CRM, bases operacionales), los **transforma** (limpia, une, calcula) y los **carga** en un sistema analítico. Ver el detalle técnico completo (herramientas, ETL vs ELT) en [[04-Ingenieria-de-Datos]].

**Por qué existe:** Es el "puente" físico que resuelve exactamente el problema del punto anterior: mover los datos de los sistemas operacionales (optimizados para transacciones) a un sistema optimizado para análisis, sin tocar ni ralentizar la operación real.

**Qué problema resuelve:** Separación de cargas de trabajo (operacional vs. analítica) + consolidación de múltiples fuentes en un solo lugar consistente.

---

## Data Warehouse

**Qué es:** Una base de datos especializada en **análisis** (OLAP), con datos ya **estructurados, limpios y modelados** (típicamente en esquemas dimensionales: tablas de hechos y dimensiones). Ejemplos: Snowflake, BigQuery, Redshift, Synapse.

**Por qué existe:** Los analistas necesitan hacer consultas complejas (agregaciones sobre millones de filas, joins entre muchas tablas) rápido, y necesitan que los datos ya tengan sentido de negocio (nombres de columnas entendibles, cálculos ya hechos), no el formato crudo de la base operacional.

**Qué problema resuelve:** Consultas analíticas lentas sobre datos crudos y desorganizados; falta de una "fuente única de verdad" para reportes de negocio.

**Su limitación:** Solo maneja bien datos **estructurados** (tablas). No es bueno ni barato para guardar imágenes, logs masivos, JSON anidado, video, audio — el tipo de datos que el Machine Learning moderno y el Deep Learning necesitan.

![[Pasted image 20260714105119.png]]

---

## Data Lake

**Qué es:** Un repositorio de almacenamiento masivo y barato que guarda datos **en su formato original** (estructurado, semi-estructurado o no estructurado) sin necesidad de definir un esquema de antemano ("schema-on-read" en vez de "schema-on-write"). Ejemplos: Amazon S3, Azure Data Lake Storage, Google Cloud Storage.

**Por qué existe:** Justo para resolver la limitación del Data Warehouse — necesitas un lugar donde guardar **todo**, sin importar el formato (CSV, JSON, imágenes, logs, video), a un costo de almacenamiento muy bajo, y decidir después cómo estructurarlo (o si necesitas estructurarlo).

![[Data-Lake-Data-Types-Diagram.webp]]

**Qué problema resuelve:** Rigidez del Data Warehouse (hay que definir el esquema antes de cargar) + costo alto de almacenar datos no estructurados o datos "por si acaso los necesito después".

**Su problema (el que causó el siguiente eslabón):** Sin disciplina, un Data Lake se convierte en un "**data swamp**" (pantano de datos) — miles de archivos sin gobierno, sin garantías de calidad, sin transacciones, difícil de auditar o de usar de forma confiable para reportes de negocio serios.

---

## Lakehouse

**Qué es:** Una arquitectura que combina lo mejor de ambos mundos: el almacenamiento barato y flexible del Data Lake **más** las garantías de calidad, transacciones (ACID) y estructura del Data Warehouse — sobre el mismo almacenamiento de objetos barato. Habilitado por formatos como **Delta Lake, Apache Iceberg, Apache Hudi** (ver [[04-Ingenieria-de-Datos]]). Ejemplos de plataformas: [[08-Plataformas-de-Datos-y-ML#Databricks|Databricks]], [[08-Plataformas-de-Datos-y-ML#Microsoft Fabric|Microsoft Fabric]].

**Por qué existe:** Durante años las empresas mantenían _dos_ sistemas separados y duplicados: un Data Lake para datos crudos/no estructurados y un Data Warehouse para análisis de negocio — con procesos de sincronización costosos y datos desactualizados entre ambos. El Lakehouse nace para **unificar ambos en una sola plataforma**.

**Qué problema resuelve:** Duplicación de infraestructura, costos, y la latencia/inconsistencia de mantener datos sincronizados entre Lake y Warehouse.

![[lakehouse-warehouse-datalake-1024x538.png.webp]]

---

## Feature Store

**Qué es:** Un sistema especializado en almacenar, versionar y **servir features** (ver [[01-Fundamentos-y-Panorama-General#¿Qué significa Feature?]]) de forma consistente tanto para entrenamiento como para inferencia en producción. Ejemplos: Feast, Tecton, Databricks Feature Store, SageMaker Feature Store.

**Por qué existe:** Un problema muy real y costoso llamado **training-serving skew**: el Data Scientist calcula una feature de una forma en su notebook de entrenamiento (ej. "promedio de compras de los últimos 30 días", calculado con Pandas sobre un CSV), pero en producción esa misma feature se calcula con lógica ligeramente distinta (ej. con SQL en tiempo real) — y esa pequeñísima diferencia hace que el modelo se comporte distinto en producción de como se comportó en las pruebas.

![[feature-store-feature-image.webp]]

**Qué problema resuelve:** Inconsistencia entre el cálculo de features en entrenamiento vs. en producción + duplicación de esfuerzo (cada equipo recalculando las mismas features) + falta de reutilización de features entre proyectos.

---

## Entrenamiento

**Qué es:** Ver [[01-Fundamentos-y-Panorama-General#¿Qué significa Entrenamiento?]]. A nivel de arquitectura empresarial, este paso normalmente corre en infraestructura especializada (clusters con GPU, notebooks gestionados, jobs programados) y consume datos ya limpios desde el Data Warehouse/Lakehouse y features desde el Feature Store.

**Por qué existe en esta posición del flujo:** Es el punto donde todo el trabajo previo (ERP → CRM → BD → ETL → Warehouse/Lake → Feature Store) se convierte finalmente en algo que puede predecir o decidir.

---

## Registro del Modelo (_Model Registry_)

**Qué es:** Un sistema que **versiona y cataloga modelos entrenados**, junto con su metadata: qué datos se usaron, qué métricas obtuvo, quién lo entrenó, cuándo, y en qué etapa del ciclo de vida está (staging, producción, archivado). Ejemplos: MLflow Model Registry, SageMaker Model Registry, Vertex AI Model Registry.

![[Pasted image 20260714105708.jpg]]

**Por qué existe:** Sin un registro, los modelos entrenados terminan como archivos sueltos en el laptop de alguien o en una carpeta sin control de versiones — nadie sabe cuál es la versión "actual" en producción, ni cómo reproducir un modelo anterior si algo sale mal.

**Qué problema resuelve:** Trazabilidad, reproducibilidad y gobernanza de modelos — el equivalente de Git, pero para modelos entrenados en vez de código.

![[Pasted image 20260714105715.png]]

---

## Despliegue (Deployment)

**Qué es:** El proceso de tomar un modelo del registro y exponerlo para que pueda hacer [[01-Fundamentos-y-Panorama-General#¿Qué significa Inferencia?|inferencia]] sobre datos reales — normalmente como una API (para inferencia en tiempo real) o como un job programado (para inferencia por lotes/batch).

**Por qué existe:** Es el puente entre "tengo un modelo que funciona en mis pruebas" y "el negocio realmente está usando este modelo para decidir algo". Ver [[01-Fundamentos-y-Panorama-General#¿Qué significa Producción?]].

**Qué problema resuelve:** Sin un proceso formal de despliegue, poner un modelo en producción es manual, propenso a errores humanos, y difícil de revertir si algo sale mal (de ahí prácticas como _blue-green deployment_ o _canary releases_, tomadas prestadas de la ingeniería de software tradicional).

---

## Monitoreo

**Qué es:** Vigilancia continua del modelo **ya en producción**: ¿sigue prediciendo bien? ¿la distribución de los datos de entrada cambió respecto a los datos con los que se entrenó (**data drift**)? ¿la relación entre las features y el target cambió (**concept drift**)? ¿hay errores técnicos, latencia alta, caídas?

**Por qué existe:** A diferencia del software tradicional (que falla de forma visible — un error 500, un crash), un modelo de ML puede **fallar en silencio**: sigue respondiendo, sigue "funcionando" técnicamente, pero sus predicciones se van volviendo cada vez menos precisas porque el mundo cambió y el modelo no. Sin monitoreo, nadie se entera hasta que el daño de negocio ya es grande.

**Qué problema resuelve:** Degradación silenciosa del rendimiento del modelo con el tiempo.

---

## Reentrenamiento

**Qué es:** Volver a entrenar el modelo con datos más recientes, ya sea de forma programada (ej. cada mes) o disparada automáticamente cuando el monitoreo detecta _drift_ significativo.

**Por qué existe:** El mundo cambia — el comportamiento de los clientes, las condiciones de mercado, patrones estacionales — y un modelo entrenado con datos de hace un año puede volverse obsoleto. El reentrenamiento es lo que convierte el ciclo en verdaderamente **continuo** en vez de un proyecto de una sola vez.

**Cierra el ciclo:** El reentrenamiento vuelve a consumir datos frescos del Data Warehouse/Lakehouse y del Feature Store, produce un nuevo modelo, que se vuelve a registrar, desplegar y monitorear — el flujo completo se repite indefinidamente. Este ciclo cerrado y automatizado es, en esencia, la razón de ser del [[02-Roles-y-Carreras-en-DS-ML#MLOps Engineer|MLOps Engineer]].

---

## La idea central de esta nota

Cada eslabón de esta cadena **nació como respuesta a una limitación específica del eslabón anterior**:

- El ERP/CRM centralizan operación → pero no son buenos para análisis pesado.
- El ETL mueve datos sin afectar la operación → pero necesita un destino optimizado.
- El Data Warehouse optimiza el análisis estructurado → pero no maneja datos no estructurados baratos.
- El Data Lake maneja cualquier dato barato → pero sin disciplina se vuelve un pantano.
- El Lakehouse unifica ambos → habilitando ML/DL sobre datos gobernados.
- El Feature Store evita que el entrenamiento y la producción calculen las cosas distinto.
- El Model Registry evita perder trazabilidad de qué modelo es cuál.
- El Despliegue formaliza cómo un modelo llega a usarse de verdad.
- El Monitoreo evita que un modelo falle en silencio.
- El Reentrenamiento evita que el modelo se quede fosilizado en el pasado.

Cuando evalúes cualquier herramienta o plataforma nueva, pregúntate: **¿qué eslabón de esta cadena resuelve, y qué limitación del eslabón anterior está atacando?** Esa pregunta es la que evita aprender herramientas de forma aislada y sin contexto.

---

## Ver también

- [[01-Fundamentos-y-Panorama-General]]
- [[02-Roles-y-Carreras-en-DS-ML]]
- [[04-Ingenieria-de-Datos]]
