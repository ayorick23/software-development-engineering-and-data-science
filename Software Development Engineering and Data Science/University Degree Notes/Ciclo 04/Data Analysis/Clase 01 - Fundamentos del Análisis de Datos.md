---
Fecha de creación: 2026-07-07T18:19:00
Materia:
  - Data Analysis
Fecha de clase: 2026-07-07
---
# Fundamentos del Análisis de Datos

El **análisis de datos** es el proceso de recolectar, limpiar, transformar y modelar datos para descubrir información útil y respaldar la toma de decisiones. Su propósito es convertir datos sin procesar en conocimiento estratégico para entender tendencias, optimizar procesos o resolver problemas empresariales o científicos.

## Escalas de Datos

Las **escalas de datos** (o escalas de medición) definen la naturaleza matemática de la información y qué análisis estadísticos son válidos. Se dividen en cinco niveles:

1. **Nominal**
	Etiquetas de identificación básica (sin orden inherente).
2. **Categórica**
	Grupos clasificados, denota clase, pero sin magnitud.
3. **Ordinal**
	Establece orden bajo transitividad, sin definir distancias.
4. **Intervalo**
	Medición cuantitativa pura. Orden y distancia medible exacta.
5. **Proporción**
	Máximo nivel informativo. Valores adimensionales y ratios absolutos.

---

## Datos Estructurados

Son datos que encajan perfectamente en un modelo de datos tabular con filas y columnas. Siguen un esquema estricto, lo que los hace fácilmente legibles, buscables y analizables tanto por máquinas como por humanos.

- **Características:** Esquema fijo, altamente organizados, almacenados en tablas relacionales (RDBMS).
- **Ejemplos:** Bases de datos SQL, hojas de cálculo (Excel, Google Sheets), números de tarjetas de crédito, fechas o números de identificación.
- **Almacenamiento/Herramientas:** Bases de datos relacionales como MySQL, PostgreSQL, Oracle y Microsoft SQL Server.

## Datos Semi-Estructurados

Son un punto intermedio entre los datos estructurados y los no estructurados. No tienen un modelo o esquema rígido, pero utilizan etiquetas, metadatos u otros marcadores semánticos para separar elementos y organizar las jerarquías o registros dentro del archivo.

- **Características:** Flexibles para adaptarse a esquemas variables, usan etiquetas clave-valor, jerárquicos.
- **Ejemplos:** Archivos JSON, archivos XML, correos electrónicos (tienen texto libre pero también metadatos como _Remitente_, _Destinatario_ y _Fecha_).
- **Almacenamiento/Herramientas:** Bases de datos NoSQL basadas en documentos (como MongoDB, Couchbase o PostgreSQL con soporte JSON).

## Datos No Estructurados

Carecen de un modelo de datos predefinido o un formato fijo. Representan la mayor parte del volumen de datos en el mundo y son complejos de procesar usando métodos tradicionales, por lo que suelen requerir Inteligencia Artificial (IA) para su análisis.

- **Características:** Sin formato, pesados, requieren mayor procesamiento computacional.
- **Ejemplos:** Imágenes (JPG, PNG), videos (MP4), grabaciones de audio, mensajes de redes sociales, documentos de texto sin formato, archivos PDF, logs de servidores.
- **Almacenamiento/Herramientas:** Data Lakes (lagos de datos) y sistemas de almacenamiento distribuido como Hadoop, Amazon S3, Azure Blob Storage o Google Cloud Storage.

---

## Procesamiento de Datos

(ver [[04-Ingenieria-de-Datos#ETL vs. ELT]] [[06-Familia-de-Algoritmos-ML#NLP (Natural Language Processing / Procesamiento de Lenguaje Natural)]] y [[06-Familia-de-Algoritmos-ML#Computer Vision (Visión por Computadora)]])

El procesamiento de datos abarca un conjunto de técnicas diseñadas para transformar datos sin procesar en información útil.

- **NLP:** Natural Language Processing para extraer entidades y sentimientos de textos.
- **Computer Vision:** Redes neuronales para etiquetas y clasificar contenido en imágenes y video.
- **ETL/ELT:** Transformación de JSON y XML hacia tablas analíticas para BI.

### Flujo de Trabajo Típico

En proyectos complejos de Big Data e Inteligencia Artificial, estos procesos suelen trabajar en conjunto:

- **ETL/ELT** ingiere millones de registros estructurados y no estructurados desde distintas plataformas.
- **NLP** y **Computer Vision** se aplican como pasos avanzados o etapas de "Transformación" dentro del flujo para extraer metadatos de textos e imágenes (ej. transcribir texto de una imagen o categorizar comentarios de un foro).
- Los datos procesados se cargan en bases de datos analíticas para alimentar dashboards de inteligencia de negocios o entrenar modelos predictivos.

---

## OLTP y OLAP

El procesamiento analítico en línea (OLAP) y el procesamiento de transacciones en línea (OLTP) son sistemas de procesamiento de datos que ayudan a almacenar y analizar datos empresariales. Puede recopilar y almacenar datos de múltiples fuentes, como sitios web, aplicaciones, medidores inteligentes y sistemas internos.

### OLTP (Online Transaction Processing)

**OLTP** (_Procesamiento de Transacciones en Línea_) es la arquitectura de base de datos que impulsa las operaciones empresariales diarias. Su función principal es procesar un volumen masivo de transacciones concurrentes (como compras en línea, transferencias bancarias o reservas) de forma rápida, segura y en tiempo real.

### OLAP (Online Analytical Processing)

**OLAP** (_Procesamiento Analítico en Línea_) es una tecnología utilizada en Inteligencia de Negocios para analizar grandes volúmenes de datos desde múltiples perspectivas. A diferencia de los sistemas transaccionales, organiza la información en **cubos multidimensionales**, permitiendo realizar consultas complejas y reportes financieros a alta velocidad sin afectar el rendimiento operativo.

###  Diferencias clave entre el OLAP y el OLTP

El propósito principal del procesamiento analítico en línea (OLAP) es analizar los datos agregados, mientras que el propósito principal del procesamiento de transacciones en línea (OLTP) es procesar las transacciones de bases de datos. Diferencias clave:

- Formato de datos
- Arquitectura de datos
- Rendimiento
- Requisitos

### Cuándo usar el OLAP o el OLTP

El OLAP y el OLTP son dos sistemas de procesamiento de datos diferentes diseñados para diferentes propósitos. El sistema de OLAP está optimizado para análisis de datos e informes complejos, mientras que el sistema de OLTP está optimizado para el procesamiento transaccional y las actualizaciones en tiempo real.

| **Criterios**           | **OLAP**                                                                                                                        | **OLTP**                                                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Objetivo                | Un sistema de OLAP le permite analizar grandes volúmenes de datos para respaldar la toma de decisiones.                         | El OLTP le permite administrar y procesar transacciones en tiempo real.                                        |
| Origen de datos         | Un sistema de OLAP utiliza datos históricos y agregados de varios orígenes.                                                     | Un sistema de OLTP utiliza datos transaccionales y en tiempo real de un solo origen.                           |
| Estructura de datos     | Un sistema de OLAP usa bases de datos multidimensionales (cubos) o relacionales.                                                | Un sistema de OLTP usa bases de datos relacionales.                                                            |
| Modelo de datos         | Un sistema de OLAP utiliza un esquema en estrella, un esquema en forma de copo de nieve u otros modelos analíticos.             | Un sistema de OLTP utiliza modelos normalizados o desnormalizados.                                             |
| Volumen de datos        | Un sistema de OLAP tiene grandes requisitos de almacenamiento. Piense en terabytes (TB) y petabytes (PB).                       | Un sistema de OLTP tiene requisitos de almacenamiento comparativamente más pequeños. Piense en gigabytes (GB). |
| Tiempo de respuesta     | Un sistema de OLAP tiene tiempos de respuesta más largos, normalmente en segundos o minutos.                                    | Un sistema de OLTP tiene tiempos de respuesta más cortos, normalmente en milisegundos                          |
| Aplicaciones de ejemplo | Un sistema de OLAP es ideal para analizar tendencias, predecir el comportamiento de los clientes e identificar la rentabilidad. | Un sistema de OLTP es bueno para procesar pagos, administrar datos de clientes y procesar pedidos.             |

---

## Minería de Datos

La **minería de datos** es una técnica asistida por computadora que se utiliza en los análisis para procesar y explorar grandes conjuntos de datos. Gracias a las herramientas y métodos de minería de datos, las organizaciones pueden descubrir patrones y relaciones ocultas en sus datos. La minería de datos transforma datos en bruto en conocimiento práctico. Las compañías utilizan dicho conocimiento para resolver problemas, analizar las consecuencias en el futuro de decisiones empresariales y aumentar sus márgenes de beneficio.

- **Mito**
	La "minería" no consiste en extraer los datos; los datos ya existen almacenados (en bases relacionales, data warehouses).
- **Realidad**
	Es el proceso sistemático de extraer conocimiento práctico y significativo a partir de esos grandes volúmenes de datos en bruto.

### ¿Cómo funciona la minería de datos?

El proceso estándar interindustrial para la minería de datos **(CRISP-DM)** es una excelente guía para iniciar el proceso de minería de datos. CRISP-DM es tanto una metodología como un modelo de proceso que es neutral en cuanto al sector, la herramienta y la aplicación.

- Como metodología, describe las fases típicas de un proyecto de minería de datos, indica las tareas implicadas en cada etapa y explica las relaciones entre estas tareas.
- Como modelo de proceso, CRISP-DM proporciona información general sobre el ciclo de vida de la minería de datos.

### ¿Cuáles son las seis fases del proceso de minería de datos?

Al utilizar las fases flexibles de CRISP-DM, los equipos de datos pueden pasar de una fase a otra según sea necesario. Además, las tecnologías de _software_ pueden realizar algunas de estas tareas o apoyarlas.

#### Metodología CRISP-DM

1. **Comprensión del negocio:** Identificar objetivos empresariales.
2. **Comprensión de los datos:** Recopilación y evaluación.
3. **Preparación de los datos:** Limpieza y formateo crítico.
4. **Modelado:** Aplicación de algoritmos ML.
5. **Evaluación:** Medición frente al objetivo inicial.
6. **Implementación:** Despliegue para generar inteligencia.

### Preparación de Datos

- **Consumo de Tiempo:** Es, por mucho, la fase más larga y crítica del ciclo CRISP-DM. Los algoritmos requieren alta calidad.
- **Limpiar:** Gestionar datos faltantes, valores nulos predeterminados y errores físicos de medición.
- **Integrar:** Fusionar distintos conjuntos de datos dispares (OLAP, CSV, JSON) en un esquema unificado.
- **Formatear:** Configurar y transformar estrictamente los tipos y escalas de datos para la tecnología de minería.

### Técnicas de Extracción: Clasificación y Clústeres

Las técnicas de minería de datos se basan en varios campos de aprendizaje que se entrelazan, como el análisis estadístico, el **Machine Learning (ML)** y las matemáticas. A continuación, se exponen algunos ejemplos.

- Minería de reglas de asociación
- Clasificación (Supervisado)
- Agrupación en Clústeres (No Supervisado)
- Análisis de secuencias y trayectorias
