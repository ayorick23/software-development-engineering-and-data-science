---
Fecha de creación: 2026-04-25T14:01:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-04-25
---
[[Clase 12 - Introducción a Apache Cassandra|← Clase anterior]] | [[Clase 14 - Instalación y Configuración de Cassandra|Clase siguiente →]]

# Estrategias de Implementación

Apache Cassandra se ha consolidado como la base de datos NoSQL distribuida líder para manejar **Petabytes** de datos con latencias predecibles.

Utilizada por gigantes como **Netflix, Apple y Spotify**, su arquitectura peer-to-peer permite alta disponibilidad sin puntos únicos de fallo. , otros proyectos han experimentado **FRACASOS COSTOSOS** debido a modelados inadecuados o subestimación de la complejidad operacional.

## Casos de Éxito

### Netflix

Cassandra sirve como columna vertebral para una amplia gama de casos de uso dentro de Netflix, que van desde registros de usuarios y almacenamiento de historiales de visualización hasta soporte de análisis en tiempo real y transmisión en vivo. la abstracción aprovecha las capacidades de partición y agrupación de Cassandra.

EI ID del registro actúa como clave de partición y la clave del elemento como columna de agrupación:

| ID   | Key   | Value       | Value Metadata        |
| ---- | ----- | ----------- | --------------------- |
| 1234 | Key 1 | Value 1     | ``{"metadata": ...}`` |
| 1234 | Key 2 | ``<empty>`` | ``{"metadata": ...}`` |

### Spotify

Spotify utiliza Apache Cassandra para manejar grandes volúmenes de datos generados por sus usuarios en tiempo real, como historiales de reproducción, recomendaciones y estados de sesión, aprovechando su arquitectura distribuida sin nodo maestro para garantizar alta disponibilidad, baja latencia y escalabilidad global; aunque sacrifica características como ``JOINS`` y consistencia inmediata, obtiene un sistema capaz de procesar millones de eventos por segundo y ofrecer una experiencia personalizada y rápida a usuarios en todo el mundo.

## Casos de Fracaso

### Fanatics

Fanatics intentó usar Apache Cassandra para soportar picos de tráfico (como eventos deportivos grandes), pero tuvo problemas por un mal modelado de datos y uso inadecuado de la consistencia. Diseñaron estructuras más cercanas a bases relacionales (intentando hacer consultas flexibles tipo SQL, select en cualquier columna), 10 que en Cassandra provoca lecturas ineficientes, hotspots y latencias altas.

Además, no ajustaron bien el **nivel de consistencia** ni la distribución de particiones **(Partition Key)**, lo que generó datos inconsistentes y caídas de rendimiento bajo carga. EI resultado fue una mala experiencia en momentos críticos (carritos, inventario, órdenes), y eventualmente tuvieron que rediseñar su arquitectura y la forma en que usaban Cassandra.

### Comcast

Comcast enfrentó problemas iniciales al usar Apache Cassandra debido a una **mala elección de niveles de consistencia** (como ONE en datos críticos) y una deficiente distribución de particiones que generaba hotspots y latencias altas bajo carga; esto provocó inconsistencias en datos como estados de dispositivos y fallos en momentos de alto tráfico, pero al migrar a niveles como ``LOCAL_QUORUM``, rediseñar las partition keys (_incluyendo técnicas como time bucketing_) y ajustar la replicación entre datacenters, lograron estabilizar el sistema y escalar correctamente para manejar millones de eventos en tiempo real con buen balance entre rendimiento y consistencia.

### Expedia

Cassandra les permite procesar millones de consultas con baja latencia y alta disponibilidad, algo clave en un negocio donde los usuarios esperan resultados inmediatos.

Sin embargo, también enfrentaron retos típicos: al inicio, problemas de rendimiento por **mal diseño de particiones** (que generaban hotspots) y decisiones incorrectas en niveles de consistencia, lo que podía causar datos desactualizados en búsquedas o precios.

Para solucionarlo, rediseñaron su modelo de datos basado en consultas (query-driven), distribuyeron mejor las partition keys y adoptaron niveles como ``LOCAL_QUORUM`` para equilibrar consistencia y velocidad. En resumen, Expedia logró escalar con Cassandra, pero solo después de ajustar correctamente su arquitectura y entender cómo funciona el modelo distribuido.

## Ventajas de Cassandra

- **Alta Disponibilidad y Tolerancia a Fallos:** Cassandra elimina el punto único de fallo mediante su arquitectura masterless, donde todos los nodos son iguales. EI diseño peer-to-peer de Cassandra asegura que el sistema continúe operando incluso cuando múltiples nodos fallan simultáneamente:
	- Replicación automática entre nodos y centros de datos
	- Sin necesidad de failover manual
	- Degradación gradual en lugar de fallo total
	- Recuperación automática de nodos

Según un estudio de Datastax (2021), Cassandra puede alcanzar disponibilidades del 99.999% (menos de 5 minutos de downtime anual) con configuraciones adecuadas de replicación multi-datacenter.

### Alta Disponibilidad

- **Arquitectura Masterless:** Todos los nodos son iguales; no existe un "líder" que pueda fallar.
- **Replicación Automática:** Los datos se copian entre nodos y centros de datos de forma transparente.
- **Degradación Gradual:** EI sistema sigue operando incluso si múltiples nodos fallan simultáneamente.

### Escalabilidad Lineal/Horizontal

Agregar nodos al clúster aumenta proporcionalmente tanto el **throughput** como la capacidad de almacenamiento.

**Evidencia:**

- **Netflix:** Escaló de 50 a 500+ nodos sin rediseño.
- **Apple:** Opera clústeres de 75,000+ nodos.
- **Instagram:** Maneja 400TB+ de datos globales.

### Rendimiento de Escritura

Cassandra puede procesar hasta 1 millón de escrituras por segundo por nodo debido a su arquitectura optimizada:

- **Escritura Secuencial:** Se registra inmediatamente en el Commit Log (disco).
- **Memtable:** EI dato se almacena en memoria RAM para respuesta rápida.
- **SSTable:** Flush asíncrono a disco para persistencia final.

>[!IMPORTANT] El Commit Log es un registro de escritura anticipada (Write-Ahead Log).
>Asegura que si un nodo falla antes de persistir la Memtable en la SSTable, los datos puedan recuperarse al reiniciar el nodo, leyendo secuencialmente este registro.

## GDPR

La GDPR busca proteger los datos personales y la forma en la que las organizaciones los procesan, almacenan y, finalmente, destruyen, cuando esos datos ya no son requeridos.

La ley provee control individual en relación a cómo las compañías pueden usar la información que está directa y personalmente relacionada con los individuos, y otorga ocho derechos específicos:

- EI derecho a ser informado
- EI derecho al acceso
- EI derecho de la rectificación
- EI derecho al borrado de datos
- EI derecho a restringir el procesamiento
- EI derecho de portabilidad de los datos
- EI derecho de objeción
- EI derecho en relación con la creación de perfiles y toma de decisiones automatizadas.

## Distribución Geográfica

Soporte nativo para replicación entre múltiples centros de datos (Multi-DC).

- **Localidad de datos:** Menor latencia para usuarios globales.
- **Cumplimiento:** Regulaciones GDPR o Regulación General de Protección de Datos
- **Disaster Recovery:** Continuidad de negocio total.

## Consistencia Ajustable Quorum

**Read Consistency Levels**

| Level            | Replicas                                                                                                             | Consistency             | Availability |
| ---------------- | -------------------------------------------------------------------------------------------------------------------- | ----------------------- | ------------ |
| ``ALL``          | All                                                                                                                  | Highest                 | Lowest       |
| ``EACH_QUORUM``  | Quorum in each datacenter                                                                                            | Same across datacenters |              |
| ``QUORUM``       | Quorum of all nodes across all datacenters. Some level of failure is possible.                                       |                         |              |
| ``LOCAL_QUORUM`` | Quorum of replicas in the same datacenter as the coordinator node. Avoids communication latency between datacenters. | Low in multi-datacenter |              |

## Desventajas de Cassandra

### Limitaciones en Consultas Ad-Hoc

Operaciones no soportadas nativamente:

- Agregaciones complejas (``GROUP BY``, ``SUM``, ``AVG`` sin materialización).
- Subconsultas
- ``JOINs`` entre tablas
- Filtros arbitrarios sin índices secundarios

### Consistencia Eventual por Defecto

Aunque ajustable, la consistencia eventual puede causar anomalías en aplicaciones que requieren consistencia fuerte. Vogels (2009) advierte que "la consistencia eventual requiere que las aplicaciones sean diseñadas para tolerar lecturas potencialmente desactualizadas".

**Problemas comunes:**

- Lecturas de escrituras propias no garantizadas con ``LOCAL_ONE``.
- Conflictos de escritura que requieren resolución manual.
- Complejidad en implementar transacciones multi-registro.

### Complejidad del Modelado

- Sin JOINs:
	- No se pueden combinar tablas en tiempo de ejecución. Los datos deben unirse al guardar.
- Desnormalización:
	- Obligatorio

## Cambio de Paradigma

- SQL Tradicional:
	- Se enfoca en las Entidades y sus relaciones. El diseño busca eliminar la redundancia (Normalización). _"Diseña ahora, consulta después"_

```sql
SELECT * FROM usuarios
WHERE email LIKE '%@gmail.com'
```

- Cassandra (CQL):
	- Se enfoca en las consultas (queries) que la aplicación va a ejecutar. El diseño parte de "¿cómo voy a leer estos datos?" y la tabla se modela alrededor de esa consulta, aceptando redundancia si hace falta. _"Consulta primero, diseña después"_ — lo opuesto al enfoque de SQL.

```cql
-- Requiere tabla especifica
CREATE TABLE usuarios_por_dominio (
	dominio text,
	email text,
	usuario_id uuid,
	PRIMARY KEY (dominio, email)
)
```

## Sobrecarga Operacional

**Requerimientos de Producción:**

- **RAM:** 8GB a 32GB (Heap de JVM crítico).
- **CPU:** 4 cores mínimo para compactación.
- **Disco:** SSDs obligatorios.
- **Red:** 10Gbps+ para replicación.

El 40% de las empresas subestiman estos costos iniciales.

## ``cassandra.yaml``

**Parámetros Esenciales:**

```yaml
cluster_name: 'prod_cluster'
seeds: "10.0.1.1.,10.0.1.2"
listen address: 10.0.1.1
endpoint_snitch:
	GossipingPropertyFileSnitch
authenticator:
	PasswordAuthenticator
```

- **Seeds:** Nodos de contacto inicial para el Gossip.
- **Snitch:** Define cómo Cassandra entiende la topología (Racks/DC).
- **Heap:** Mal configurado causa pausas de GC fatales.

### Seguridad en el Clúster

Control de acceso basado en roles (RBAC) y encriptación:

```cql
-- Crear Rol Administrativo
CREATE ROLE admin WITH PASSWORD =
AND SUPERUSER = true AND LOGIN = true;

-- Otorgar Permisos
GRANT SELECT, MODIFY ON KEYSPACE app_data TO app_user;
```

- **Encriptación Internode:** Obligatoria para proteger datos que viajan entre centros de datos geográficos.
