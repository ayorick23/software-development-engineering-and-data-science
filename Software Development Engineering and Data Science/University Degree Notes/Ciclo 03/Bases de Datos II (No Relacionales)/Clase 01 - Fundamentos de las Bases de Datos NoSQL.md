---
Fecha de creación: 2026-01-24T14:24:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-01-24
---
# Fundamentos de las Bases de Datos NoSQL

## Introducción a las Bases de Datos No Relacionales

Los sistemas de gestión de bases de datos NoSQL (Not Only SQL) están creados para guardar, manejar y tratar grandes cantidades de datos semi-estructurados o no estructurados.  NoSQL permite la distribución eficaz de datos a través de varios servidores gracias a su capacidad para escalar horizontalmente y su flexibilidad en el esquema de datos, lo que contrasta con las bases de datos relacionales tradicionales.

 "NoSQL" no quiere decir que SQL haya sido completamente eliminado, sino que es una evolución hacia sistemas más adaptables que pueden convivir con tecnologías relacionales clásicas.  Estas bases de datos están optimizadas para aplicaciones web contemporáneas que necesitan una disponibilidad elevada, un tiempo de respuesta reducido y la habilidad de gestionar grandes cantidades de información.
 
### Origen del Término

El término NoSQL fue acuñado originalmente por Carlo Strozzi en 1998 para denominar su base de datos ligera de código abierto que no exponía una interfaz SQL estándar. Sin embargo, el significado y uso del término cambió radicalmente en 2009, cuando Johan Oskarsson, desarrollador de Last.fm, organizó un evento en San Francisco sobre bases de datos distribuidas de código abierto. En esta conferencia se popularizó el término para referirse a sistemas de bases de datos no relacionales diseñados para manejar big data y aplicaciones web de alta escala.

## Características Principales

- **Alta Velocidad:** Procesamiento rápido de consultas y escrituras masivas gracias a estructuras optimizadas y almacenamiento en memoria.
- **Flexibilidad:** Esquemas dinámicos que permiten almacenar datos sin una estructura rígida predefinida.
- **Escalabilidad:** Capacidad de crecimiento horizontal mediante la distribución de datos entre múltiples nodos.
- **Distribución:** Replicación automática de datos en múltiples servidores para garantizar disponibilidad.

![[Drawing 2026-01-24 14.28.24.excalidraw]]

## Contexto de las Bases de Datos

Durante más de cuatro décadas, las bases de datos relacionales dominaron la industria tecnológica. El modelo relacional, propuesto por Edgar F. Codd en 1970, estableció los fundamentos teóricos basados en álgebra relacional, normalización y lenguaje SQL. Sistemas como Oracle, MySQL, PostgreSQL y SQL Server se convirtieron en estándares industriales, ofreciendo consistencia transaccional mediante propiedades ACID (Atomicidad, Consistencia, Aislamiento, Durabilidad).

Sin embargo, la llegada de la Web 2.0 en los años 2000 transformó radicalmente los requisitos de almacenamiento de datos. Empresas como Google, Amazon, Facebook y Twitter comenzaron a generar volúmenes de datos sin precedentes, enfrentando limitaciones significativas con bases de datos relacionales tradicionales: dificultad para escalar horizontalmente, rigidez de esquemas, y costos elevados de infraestructura.

### Principios Fundamentales

#### El Teorema CAP (Brewer, 2000)

El Teorema CAP, formulado por Eric Brewer en 2000 y formalmente demostrado por Seth Gilbert y Nancy Lynch en 2002, establece que un sistema de base de datos distribuido solo puede garantizar simultáneamente dos de las siguientes tres propiedades:

- **Consistencia:** Todos los nodos del sistema ven exactamente los mismos datos al mismo tiempo. Cada lectura recibe la escritura más reciente o un error.
- **Disponibilidad:** El sistema responde a todas las peticiones (lectura/escritura) incluso si algunos nodos están caídos. Cada solicitud recibe una respuesta sin garantía de que contenga la versión más reciente.
- **Tolerancia a Particiones:** El sistema continúa operando a pesar de fallos arbitrarios de red que dividen el sistema en particiones aisladas que no pueden comunicarse entre sí.

![[Drawing 2026-01-24 14.53.51.excalidraw]]

#### Implicaciones del Teorema CAP

En sistemas distribuidos reales, las particiones de red no se pueden evitar (por ejemplo, debido a fallos de hardware, congestión de la red o cables cortados).  Por ende, la tolerancia a particiones es obligatoria.  La elección real se limita a:

Sistemas CP (tolerancia a la partición + consistencia):  Ponen la consistencia por encima de la disponibilidad.  El sistema puede declinar peticiones o volverse temporalmente inaccesible para preservar la consistencia durante una partición de red.  Algunos ejemplos son HBase, Redis Cluster y MongoDB (en su modo por defecto).

Sistemas AP (tolerancia a la partición + disponibilidad):  Prefieren la disponibilidad a la consistencia.  El sistema siempre responde, pero durante las particiones puede devolver información anticuada.  Cuando la partición se soluciona, los datos eventualmente convergen.  Algunos ejemplos son: CouchDB, Cassandra, DynamoDB.

Sistemas CA (Disponibilidad + Consistencia):  Solo posibles en sistemas sin distribución o con la garantía de una red perfecta.  Ejemplos: bases de datos relacionales convencionales en un solo servidor.

## Modelo BASE vs ACID

Mientras las bases de datos relacionales siguen el modelo ACID, muchas bases NoSQL adoptan el modelo BASE:

- **Básicamente disponible:** El sistema garantiza disponibilidad básica. Las respuestas pueden no contener los datos más actualizados pero el sistema permanece operativo.
- **Estado blando:** El estado del sistema puede cambiar con el tiempo sin nuevas entradas debido a la propagación asíncrona de actualizaciones entre nodos.
- **Eventualmente consistente:** Los datos eventualmente alcanzarán consistencia si el sistema deja de recibir nuevas actualizaciones. No hay garantía de cuándo ocurrirá esta convergencia.

| ACID                                                                 | BASE                                                                                                  |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Atomicity                                                            | Basically Available                                                                                   |
| Las transacciones son todo o nada. Se completan totalmente o fallan. | El sistema garantiza disponibilidad básica. Responde siempre aunque no tenga los datos mas recientes. |
| Consistency                                                          | Soft State                                                                                            |
| Los datos siempre cumplen todas las reglas de integridad.            | El estado del sistema puede cambiar sin nuevas entradas debido a la propagación asíncrona de datos.   |
| Isolation                                                            | Eventually Consistent                                                                                 |
| Transacciones concurrentes no interfieren entre sí.                  | Los datos eventualmente alcanzarán consistencia si el sistema deja de recibir actualizaciones.        |
| Durability                                                           |                                                                                                       |
| Los datos confirmados persisten incluso ante fallos del sistema.     |                                                                                                       |

## Tipos de Bases de Datos NoSQL

### Bases de Datos Clave-Valor

El modelo más simple de base de datos NoSQL. Almacena datos como colecciones de pares clave-valor, donde cada clave es única y se asocia con un valor opaco (``string``, ``JSON``, ``blob binario``). El sistema no interpreta ni indexa el contenido del valor.

**Características:**

- Operaciones básicas: ``GET``, ``PUT``, ``DELETE`` ([[Clase 01 - Introducción a la Programación Web#Métodos ``HTTP``|Métodos HTTP]])
- Rendimiento extremadamente alto (sub-milisegundo)
- Ideal para almacenamiento en memoria
- Esquema completamente flexible

**Ejemplos:**

- **Redis:** Base de datos en memoria, soporta estructuras de datos complejas (listas, conjuntos, hashes)
- **Riak:** Distribuida, alta disponibilidad, modelo AP del teorema CAP
- **Amazon DynamoDB:** Servicio gestionado de AWS, escalabilidad automática

**Casos de uso:**

- Caché de sesiones de usuario
- Carritos de compra en e-commerce
- Almacenamiento de configuraciones
- Rate limiting y contadores
- Sistemas de caché (Memcached, Redis)

**Ventajas:** Simplicidad, velocidad extrema, escalabilidad lineal.
**Desventajas:** No soporta consultas complejas, no hay relaciones entre datos.

| Key  | Value  |
| ---- | ------ |
| key1 | value1 |
| key2 | value2 |
| key3 | value3 |

### Bases de Datos Documentales

Almacenan datos como documentos semi-estructurados (``JSON``, ``BSON``, ``XML``) ([[Clase 03 - Formatos de Datos - JSON vs XML]]). Cada documento es autocontenido y puede tener estructura diferente. Soportan consultas complejas, índices y agregaciones.

**Características:**

- Documentos con esquema flexible
- Anidación de estructuras complejas
- Índices en cualquier campo
- Consultas ricas (filtros, ordenamiento, agregaciones)

**Ejemplos:**

- **MongoDB:** Base documental más popular, ``BSON`` (``Binary JSON``), agregation framework.
- **CouchDB:** Replicación multi-master, consultas MapReduce.
- **Firestore:** Base de datos de Google, tiempo real, integración móvil.

**Casos de uso:**

- Catálogos de productos con atributos variables
- Sistemas de gestión de contenidos (CMS)
- Perfiles de usuario con información heterogénea
- Aplicaciones móviles y web en tiempo real

**Ventajas:** Flexibilidad de esquema, desarrollo ágil, consultas poderosas.
**Desventajas:** Potencial duplicación de datos, complejidad en transacciones distribuidas.

```json
{
	"name": "Alice",
	"age": 30,
	"city": "New York"
}
```

### Bases de Datos Columnares

Organiza datos en columnas en lugar de filas. Optimizadas para lectura y escritura de columnas específicas en grandes volúmenes de datos. Basadas en el paper Bigtable de Google.

**Características:**

- Almacenamiento orientado a columnas
- Compresión eficiente de datos similares
- Excelente para consultas analíticas
- Familias de columnas agrupan datos relacionados

**Ejemplos principales:**

- Apache Cassandra: Alta disponibilidad, modelo AP, sin punto único de fallo
- Apache HBase: Sobre Hadoop HDFS, modelo CP, consultas en tiempo real sobre big data
- Google Bigtable: Servicio gestionado de Google Cloud

**Casos de uso:**

- Análisis de big data y data warehousing
- Series temporales (IoT, métricas, logs)
- Sistemas de recomendación
- Análisis de comportamiento de usuarios

**Ventajas:** Escalabilidad masiva, compresión eficiente, rendimiento en agregaciones.
**Desventajas:** Curva de aprendizaje pronunciada, complejidad operacional.

| column1 | column2 | column3 |
| ------- | ------- | ------- |
| 1       | 2       | 3       |
| 4       | 5       | 6       |
| 7       | 8       | 9       |

### Bases de Datos de Grafos

Diseñada para almacenar y navegar relaciones entre entidades. Los datos se representan como nodos (entidades) conectados por aristas (relaciones) con propiedades.

**Características:**

- Nodos y relaciones como ciudadanos de primera clase
- Navegación eficiente de conexiones profundas
- Consultas basadas en patrones (Cypher, Gremlin)
- Ideal para datos altamente interconectados

**Ejemplos:**

- Neo4j: Base de grafos más popular, lenguaje Cypher, ACID compliant
- ArangoDB: Multi-modelo (documentos + grafos + key-value)
- Amazon Neptune: Servicio gestionado de AWS

**Casos de uso:**

- Redes sociales (conexiones entre usuarios)
- Motores de recomendación
- Detección de fraude (patrones anómalos)
- Gestión de conocimiento y ontologías
- Redes de supply chain

**Ventajas:** Consultas de relaciones extremadamente rápidas, modelado natural de conexiones
**Desventajas:** No optimizadas para operaciones agregadas masivas, curva de aprendizaje

![[Drawing 2026-01-24 17.21.00.excalidraw]]

## Características Principales de NoSQL

1. **Esquema Flexible**
2. **Escalabilidad Horizontal**
3. **Replicación y Alta Disponibilidad**
4. **Rendimiento Optimizado**
5. **Modelos de Consistencia**

### Arquitecturas Distribuidas

- **Master-Slave:** Un nodo maestro coordina escrituras, esclavos replican para lecturas
- **Peer-to-Peer:** Todos los nodos son iguales, sin punto único de fallo (Cassandra)
- **Multi-Master:** Múltiples nodos aceptan escrituras, requiere resolución de conflictos

## Ventajas

- **Escalabilidad y Rendimiento Superior**
- **Flexibilidad en Desarrollo**
- **Alta Disponibilidad y Tolerancia a Fallos**
- **Costo-Efectividad**
- **Manejo Eficiente de Big Data**

### Casos de Éxito Documentados

- **Twitter:** Utiliza Manhattan (sistema key-value interno) para timeline y tweets.
- **Uber:** Redis para geolocalización en tiempo real de conductores y pasajeros.
- **LinkedIn:** Voldemort para almacenamiento de perfiles y grafos sociales.
- **Spotify:** Cassandra para perfiles de usuario y playlists.

## Desventajas y Limitaciones

- **Falta de Estandarización**
- **Consistencia Eventual**
- **Limitaciones Transaccionales**
- **Madurez y Soporte Limitado**
- **Complejidad Operacional**

## Las 4 V's del Big Data

1. Variedad
2. Volumne
3. Veracidad y Calidad
4. Velocidad
