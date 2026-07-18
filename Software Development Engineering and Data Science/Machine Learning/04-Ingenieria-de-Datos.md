---
tags: [data-engineering, etl, spark, kafka, data-warehouse, herramientas]
aliases: [Data Engineering, Herramientas de Ingeniería de Datos]
---
# Parte 4 — Ingeniería de Datos: Conceptos y Herramientas

> Complementa [[03-Arquitectura-Empresarial-de-Datos-y-ML]]. Aquí bajamos al detalle técnico: para cada concepto/herramienta, **por qué nació, qué problema resolvió, y cuándo una empresa la elegiría sobre otra**. No es una lista de definiciones — es una lista de decisiones de diseño.

---

## Tabla de Contenidos

1. [[#Conceptos Fundamentales|Conceptos Fundamentales]]
	1. [[#Conceptos Fundamentales#ETL vs. ELT|ETL vs. ELT]]
		1. [[#ETL vs. ELT#¿Por qué nació ELT si ya existía ETL?|¿Por qué nació ELT si ya existía ETL?]]
	2. [[#Conceptos Fundamentales#Streaming vs. Batch|Streaming vs. Batch]]
2. [[#Motores de procesamiento|Motores de procesamiento]]
	1. [[#Motores de procesamiento#Apache Spark|Apache Spark]]
	2. [[#Motores de procesamiento#PySpark|PySpark]]
	3. [[#Motores de procesamiento#DuckDB|DuckDB]]
	4. [[#Motores de procesamiento#Polars|Polars]]
3. [[#Orquestación|Orquestación]]
	1. [[#Orquestación#Apache Airflow|Apache Airflow]]
4. [[#Transformación|Transformación]]
	1. [[#Transformación#dbt (_data build tool_)|dbt (_data build tool_)]]
5. [[#Streaming / Mensajería|Streaming / Mensajería]]
	1. [[#Streaming / Mensajería#Apache Kafka|Apache Kafka]]
6. [[#Formatos de Almacenamiento|Formatos de Almacenamiento]]
	1. [[#Formatos de Almacenamiento#Parquet|Parquet]]
	2. [[#Formatos de Almacenamiento#ORC (Optimized Row Columnar)|ORC (Optimized Row Columnar)]]
	3. [[#Formatos de Almacenamiento#Delta Lake / Apache Iceberg|Delta Lake / Apache Iceberg]]
7. [[#Motores de consulta / Bases de Datos Analíticas|Motores de consulta / Bases de Datos Analíticas]]
	1. [[#Motores de consulta / Bases de Datos Analíticas#Hive (Apache Hive)|Hive (Apache Hive)]]
	2. [[#Motores de consulta / Bases de Datos Analíticas#Trino (antes PrestoSQL)|Trino (antes PrestoSQL)]]
8. [[#Plataformas de Datos en la Nube|Plataformas de Datos en la Nube]]
	1. [[#Plataformas de Datos en la Nube#Snowflake|Snowflake]]
	2. [[#Plataformas de Datos en la Nube#BigQuery|BigQuery]]
	3. [[#Plataformas de Datos en la Nube#Redshift|Redshift]]
	4. [[#Plataformas de Datos en la Nube#Synapse (Azure Synapse Analytics)|Synapse (Azure Synapse Analytics)]]
	5. [[#Plataformas de Datos en la Nube#Microsoft Fabric|Microsoft Fabric]]
	6. [[#Plataformas de Datos en la Nube#Databricks|Databricks]]
9. [[#Cómo Pensar la Elección entre Plataformas Cloud (Resumen)|Cómo Pensar la Elección entre Plataformas Cloud (Resumen)]]
10. [[#Ver también|Ver también]]

---

## Conceptos Fundamentales

### ETL vs. ELT

**ETL (Extract, Transform, Load):** Extraes los datos de la fuente, los **transformas antes de cargarlos** (limpieza, joins, cálculos) en un motor de procesamiento intermedio, y luego cargas el resultado ya limpio al destino.

**ELT (Extract, Load, Transform):** Extraes los datos y los **cargas primero, crudos**, directamente al destino (típicamente un Data Warehouse moderno), y transformas **después, dentro del propio warehouse**, usando su poder de cómputo.

![[Infographic-Option-4-2.webp]]

#### ¿Por qué nació ELT si ya existía ETL?

Porque los Data Warehouses modernos en la nube (Snowflake, BigQuery) se volvieron tan potentes y baratos para procesar datos que dejó de tener sentido transformar los datos en un servidor intermedio separado — es más simple y más rápido cargar todo crudo y dejar que el propio warehouse haga el trabajo pesado de transformación con SQL. Esto también dio origen a herramientas como **dbt**, que viven exclusivamente en la "T" de ELT.

**Cuándo elegir cuál:** ETL sigue siendo necesario cuando hay que limpiar/enmascarar datos sensibles _antes_ de que toquen el warehouse (cumplimiento, PII), o cuando la fuente es tan grande que transformar antes de cargar ahorra costos de almacenamiento/cómputo. ELT gana cuando el warehouse es potente y barato y el equipo prioriza velocidad de iteración (SQL simple en vez de pipelines de transformación complejos).

---

### Streaming vs. Batch

- **Batch (procesamiento por lotes):** Los datos se procesan en **grupos**, de forma periódica (cada hora, cada noche, cada semana) — se acumulan y luego se procesan todos juntos.

- **Streaming (procesamiento en tiempo real/continuo):** Los datos se procesan **evento por evento, a medida que llegan**, con latencia de segundos o milisegundos.

**Por qué existe streaming si ya existía batch:** Batch es más simple, más barato y suficiente para la mayoría de reportes de negocio ("¿cuánto vendimos ayer?"). Pero hay casos donde **esperar hasta la próxima corrida por lotes cuesta dinero o es inaceptable**: detección de fraude en el momento de la transacción, recomendaciones en vivo, alertas de sistemas críticos, precios dinámicos. Streaming nace específicamente para esos casos de baja latencia.

**Cuándo elegir cuál:** Si la decisión de negocio puede esperar horas, usa batch — es más simple de construir, depurar y mantener. Si cada segundo de retraso cuesta dinero o seguridad, necesitas streaming — con el costo adicional de una arquitectura más compleja (manejo de estado, ventanas de tiempo, exactamente-una-vez vs. al-menos-una-vez).

---

## Motores de procesamiento

### Apache Spark

![[Pasted image 20260714105955.png|330]]

**Qué es:** Un motor de procesamiento distribuido de datos de propósito general, capaz de procesar tanto batch como streaming, con APIs en Python (PySpark), Scala, Java y R.

**Por qué nació:** Antes de Spark (2009-2014), el estándar era **Hadoop MapReduce** — potente para procesar datos masivos distribuidos en muchas máquinas, pero **lentísimo** porque escribía resultados intermedios a disco en cada paso. Spark resolvió esto procesando la mayoría de las operaciones **en memoria RAM**, lo que lo hizo entre 10 y 100 veces más rápido para muchas cargas de trabajo típicas.

**Qué problema resuelve:** Procesar datasets que no caben en una sola máquina, distribuyendo el trabajo entre un cluster de servidores, sin que el desarrollador tenga que programar manualmente la distribución (Spark lo hace por debajo).

**Cuándo se elige:** Cuando el volumen de datos es genuinamente grande (varios GB a TB+) y no cabe cómodamente en la memoria de una sola máquina, o cuando ya existe un cluster/plataforma (Databricks, EMR) construida alrededor de Spark.

### PySpark

![[Pasted image 20260714110045.png|344]]

Es simplemente **la API de Python para Apache Spark**. Existe porque Python es el lenguaje dominante en ciencia de datos, y PySpark permite a un Data Scientist o Data Engineer usar una sintaxis muy parecida a Pandas pero que se ejecuta de forma distribuida en un cluster, en vez de en una sola máquina.

### DuckDB

![[Pasted image 20260714110105.png|330]]

**Qué es:** Un motor de bases de datos **analítico, embebido** (corre dentro de tu propio proceso, sin servidor separado) — piénsalo como "SQLite pero para análisis (OLAP) en vez de transacciones (OLTP)".

**Por qué nació:** Spark es potente pero tiene overhead considerable (levantar un cluster, incluso uno local, es "pesado") para trabajos que en realidad caben perfectamente en la memoria de una laptop o un solo servidor. DuckDB nace para el caso, extremadamente común, de "tengo unos cuantos GB de datos y quiero hacer SQL analítico rápido, sin la complejidad de un cluster".

**Cuándo se elige:** Análisis exploratorio rápido, pipelines de datos medianos que no justifican un cluster, o como motor de consulta embebido dentro de una aplicación.

![[Pasted image 20260714124244.png]]
### Polars

![[Pasted image 20260714110159.jpg|331]]

**Qué es:** Una librería de manipulación de dataframes (alternativa a pandas), escrita en Rust, diseñada desde cero para ser **rápida y eficiente en memoria**, con paralelización automática.

**Por qué nació:** Pandas fue diseñada en 2008 y tiene limitaciones de arquitectura conocidas: es de un solo hilo (no usa varios núcleos automáticamente) y su manejo de memoria es ineficiente para datasets grandes. Polars nace para ofrecer una API moderna, similar a pandas pero mucho más rápida, sin necesitar un cluster distribuido como Spark.

**Cuándo se elige:** Cuando pandas se queda lento/se queda sin memoria pero el dataset todavía no justifica la complejidad de Spark — el punto intermedio entre pandas y Spark.

---

## Orquestación

### Apache Airflow

![[Pasted image 20260714124056.png|379]]

**Qué es:** Una herramienta para **definir, programar y monitorear pipelines de datos** como código (Python), representados como DAGs (Directed Acyclic Graphs — grafos de tareas con dependencias).

**Por qué nació:** Antes de Airflow, programar pipelines de datos se hacía con cron jobs sueltos, sin visibilidad de qué falló, sin manejo de dependencias entre tareas ("la tarea B solo debe correr si la tarea A terminó bien"), sin reintentos automáticos, sin un lugar central para ver el estado de todo.

**Qué problema resuelve:** Falta de orquestación, visibilidad y confiabilidad en pipelines con múltiples pasos interdependientes.

**Cuándo se elige:** Es el estándar de facto para orquestación batch compleja. Se elige casi por defecto en cualquier equipo de datos con más de un puñado de pipelines interdependientes.

![[Pasted image 20260714124046.png]]

---

## Transformación

### dbt (_data build tool_)

![[Pasted image 20260714124416.png|339]]

**Qué es:** Una herramienta que permite escribir transformaciones de datos **usando solo SQL** (con templating de Jinja), aplicando buenas prácticas de ingeniería de software — control de versiones, testing, documentación, modularidad (una transformación puede referenciar a otra) — directamente dentro del Data Warehouse.

**Por qué nació:** Antes de dbt, las transformaciones SQL vivían dispersas en scripts sueltos, stored procedures o herramientas de ETL con interfaces gráficas pesadas, sin testing, sin control de versiones real, sin documentación. dbt trajo la disciplina de la ingeniería de software (piensa: "Git para SQL") al mundo de la transformación de datos analíticos.

**Qué problema resuelve:** Falta de reproducibilidad, testing y documentación en transformaciones SQL; es la pieza central que hizo viable el patrón ELT moderno (ver arriba) y dio origen al rol de [[02-Roles-y-Carreras-en-DS-ML#Analytics Engineer|Analytics Engineer]].

---

## Streaming / Mensajería

### Apache Kafka

![[Pasted image 20260714124519.png|370]]

**Qué es:** Una plataforma de **streaming de eventos distribuida** — piénsalo como un sistema de mensajería de alto rendimiento donde múltiples "productores" publican eventos y múltiples "consumidores" los leen, de forma desacoplada y con capacidad de manejar millones de eventos por segundo.

**Por qué nació:** Cuando una empresa tiene decenas de sistemas que necesitan comunicarse entre sí en tiempo real (el sistema de pedidos le avisa al de inventario, que le avisa al de facturación, que le avisa al de analítica...), conectarlos todos punto a punto crea una maraña insostenible de integraciones. Kafka nace en LinkedIn específicamente para resolver este problema: un solo "bus de eventos" central al que todos publican y del que todos consumen, desacoplados entre sí.

**Qué problema resuelve:** Acoplamiento excesivo entre sistemas + la necesidad de procesar eventos en tiempo real a gran escala + necesidad de que múltiples consumidores lean el mismo flujo de datos de forma independiente.

**Cuándo se elige:** Cuando existen múltiples sistemas productores/consumidores de eventos en tiempo real, o cuando el volumen de eventos es alto y se necesita que varios equipos consuman el mismo flujo de datos sin interferir entre sí.

---

## Formatos de Almacenamiento

### Parquet

**Qué es:** Un formato de archivo **columnar** (guarda los datos organizados por columna, no por fila) para almacenar datos estructurados de forma eficiente y comprimida.

**Por qué nació:** Los formatos tradicionales como CSV son "orientados a fila" — para leer una sola columna de un CSV de 50 columnas, tienes que leer el archivo entero. Para análisis (donde normalmente agregas/sumas/promedias una o pocas columnas sobre millones de filas), esto es tremendamente ineficiente. Parquet, al guardar los datos por columna, permite leer **solo las columnas que necesitas**, además de comprimir mejor (valores similares quedan juntos).

**Qué problema resuelve:** Lecturas analíticas lentas e ineficientes en formatos orientados a fila (CSV, JSON).

### ORC (Optimized Row Columnar)

**Qué es:** Un formato columnar muy similar a Parquet en propósito y beneficios, creado originalmente en el ecosistema Hadoop/Hive.

**Diferencia práctica con Parquet:** Ambos resuelven el mismo problema. Parquet se volvió el estándar más ampliamente adoptado fuera del ecosistema Hadoop (Spark, herramientas cloud modernas lo prefieren por defecto); ORC sigue siendo común en instalaciones Hive/Hadoop tradicionales. Hoy, si empiezas un proyecto nuevo sin restricciones heredadas, Parquet es la elección por defecto.

### Delta Lake / Apache Iceberg

**Qué son:** "Formatos de tabla" (_table formats_) que se construyen **encima** de archivos Parquet, agregándoles capacidades que Parquet por sí solo no tiene: transacciones ACID, versionado ("time travel" — poder consultar cómo se veían los datos en un momento pasado), manejo de esquemas cambiantes (_schema evolution_), y actualizaciones/borrados eficientes sobre datos en un Data Lake.

**Por qué nacieron:** Este es exactamente el problema que mencionamos en [[03-Arquitectura-Empresarial-de-Datos-y-ML#Lakehouse]] — un Data Lake de solo archivos Parquet sueltos no tiene garantías transaccionales: si dos procesos escriben al mismo tiempo, o si un job falla a la mitad, puedes terminar con datos corruptos o inconsistentes. Delta Lake (creado por Databricks) e Iceberg (originado en Netflix, ahora proyecto de Apache) nacen para traerle a los Data Lakes las garantías que un Data Warehouse siempre tuvo — habilitando así la arquitectura **Lakehouse**.

**Cuándo elegir cuál:** Delta Lake está más integrado de forma nativa si usas Databricks. Iceberg es más neutral respecto a motores de cómputo (funciona bien con Spark, Trino, Flink, Snowflake, etc.) y tiene fuerte adopción en arquitecturas multi-motor. Hudi (mencionado menos aquí) es una tercera alternativa con fuerte foco en cargas de trabajo con muchas actualizaciones incrementales (upserts).

---

## Motores de consulta / Bases de Datos Analíticas

### Hive (Apache Hive)

![[Apache_Hive_logo.svg|356]]

**Qué es:** Una capa que permite consultar datos almacenados en Hadoop **usando SQL**, en vez de tener que escribir programas MapReduce complejos a mano.

**Por qué nació:** Cuando Hadoop se popularizó, escribir trabajos de análisis directamente en MapReduce (Java) era lento de desarrollar y requería programadores muy especializados. Hive nace para que analistas que ya sabían SQL pudieran consultar esos mismos datos masivos sin aprender programación distribuida de bajo nivel.

**Contexto actual:** Sigue vivo en muchas instalaciones on-premise heredadas, pero motores más modernos (Spark SQL, Trino, Presto) lo han superado en velocidad para la mayoría de casos nuevos.

### Trino (antes PrestoSQL)

![[Pasted image 20260714124901.png|387]]

**Qué es:** Un motor de consulta SQL distribuido diseñado para consultar datos **donde sea que estén** — Data Lakes, Data Warehouses, bases de datos relacionales, todo desde una sola consulta SQL — sin necesidad de mover los datos primero.

**Por qué nació:** Facebook lo creó (como Presto) porque Hive era demasiado lento para consultas interactivas (analistas esperando respuestas en segundos, no minutos). Trino además resuelve un problema adicional: las empresas grandes tienen datos dispersos en múltiples sistemas, y mover todo a un solo lugar antes de poder consultarlo es costoso — Trino permite "federar" consultas entre sistemas distintos.

**Cuándo se elige:** Cuando necesitas consultas SQL rápidas e interactivas sobre datos que viven en múltiples sistemas distintos, sin consolidarlos primero.

---

## Plataformas de Datos en la Nube

Esta es la parte donde más se paga por la marca y el ecosistema, así que la comparación de **por qué una empresa elegiría una sobre otra** importa mucho.

### Snowflake

![[Pasted image 20260714124938.png|348]]

**Qué es:** Un Data Warehouse en la nube, agnóstico de proveedor cloud (corre sobre AWS, Azure o GCP), con una característica distintiva: **separa completamente el almacenamiento del cómputo**, permitiendo escalar cada uno de forma independiente y pagar solo por lo que usas de cada uno.

**Por qué una empresa lo elegiría:** Simplicidad operativa (administración mínima), la separación cómputo/almacenamiento es muy atractiva para cargas de trabajo variables, y es neutral respecto al proveedor cloud (útil si la empresa no quiere atarse a AWS/Azure/GCP específicamente, o ya usa una nube distinta para otras cosas).

### BigQuery

![[Pasted image 20260714125009.png|354]]

**Qué es:** El Data Warehouse serverless de Google Cloud — no hay clusters que administrar en absoluto; simplemente ejecutas consultas SQL y Google gestiona toda la infraestructura por debajo.

**Por qué una empresa lo elegiría:** Si la empresa ya vive en el ecosistema de Google Cloud, o valora no tener que pensar en absoluto en infraestructura (verdaderamente serverless), o necesita integración fuerte con herramientas de Google (Looker, Google Analytics, Google Ads).

### Redshift

![[Pasted image 20260714125038.png|338]]

**Qué es:** El Data Warehouse de Amazon Web Services (AWS), uno de los pioneros de los warehouses en la nube (2012).

**Por qué una empresa lo elegiría:** Si la empresa ya está fuertemente comprometida con AWS (usa S3, Lambda, etc.), la integración nativa con el resto del ecosistema AWS es una ventaja fuerte.

### Synapse (Azure Synapse Analytics)

![[Pasted image 20260714125120.png|354]]

**Qué es:** La plataforma de analítica de datos de Microsoft Azure, que combina Data Warehouse, Data Lake y herramientas de Big Data/Spark en un solo servicio integrado.

**Por qué una empresa lo elegiría:** Si la empresa ya usa el ecosistema Microsoft (Azure, Power BI, Office 365, Active Directory) — muy común en empresas grandes tradicionales (banca, telecom, gobierno) que ya tienen contratos e infraestructura Microsoft.

### Microsoft Fabric

![[Fabric-transparent-logo-1.webp|356]]

**Qué es:** La plataforma más reciente de Microsoft (2023), que unifica en un solo producto SaaS varias piezas que antes estaban separadas: Data Engineering, Data Warehouse, Data Science, Real-Time Analytics y Power BI, todas compartiendo una sola capa de almacenamiento (OneLake) basada en formato Delta/Parquet.

**Por qué una empresa lo elegiría:** Es la apuesta de Microsoft por el modelo "todo en una sola plataforma integrada" (similar a la propuesta de Databricks pero desde el lado de Microsoft), atractiva para empresas ya profundamente integradas con Power BI y el resto del stack Microsoft que quieren reducir la cantidad de herramientas distintas que administran.

### Databricks

![[Databricks-Logo.webp|351]]

**Qué es:** La plataforma creada por los autores originales de Apache Spark, construida alrededor de la arquitectura **Lakehouse** (usando Delta Lake), que unifica ingeniería de datos, ciencia de datos, ML y BI sobre una sola copia de los datos.

**Por qué una empresa lo elegiría:** Cuando el caso de uso central es fuertemente de **ciencia de datos/ML a gran escala** (no solo BI/reportes tradicionales) — Databricks tiene el ecosistema más maduro para notebooks colaborativos, entrenamiento distribuido, MLflow integrado, y manejo de datos no estructurados junto con estructurados en un mismo lugar. Es agnóstico de nube (corre sobre AWS, Azure o GCP).

---

## Cómo Pensar la Elección entre Plataformas Cloud (Resumen)

En la práctica, la decisión rara vez es puramente técnica — depende de:

1. **Nube ya adoptada** por la empresa (si ya son 100% AWS, el camino de menor fricción es Redshift/AWS Glue; si son Azure, Synapse/Fabric).
2. **Tipo de carga de trabajo dominante**: BI/reportes tradicionales sobre datos estructurados (Snowflake/BigQuery/Redshift brillan) vs. ML/DL a gran escala sobre datos mixtos (Databricks brilla).
3. **Modelo de costos**: pago por cómputo separado del almacenamiento (Snowflake, BigQuery) vs. modelos más tradicionales de cluster reservado.
4. **Apetito por vendor lock-in**: algunas empresas priorizan explícitamente herramientas multi-nube (Snowflake, Databricks, Trino) para no depender de un solo proveedor.

---

## Ver también

- [[01-Fundamentos-y-Panorama-General]]
- [[02-Roles-y-Carreras-en-DS-ML]]
- [[03-Arquitectura-Empresarial-de-Datos-y-ML]]
- [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]]
