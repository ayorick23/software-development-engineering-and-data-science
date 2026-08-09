---
Fecha de creación: 2026-04-18T14:07:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-04-18
---
[[Clase 10 - Diseño hacia Paradigmas NoSQL|← Clase anterior]] | [[Clase 13 - Estrategias de Implementación Cassandra|Clase siguiente →]]

# Introducción a Apache Cassandra
(ver [[Clase 03 - Gestores de Bases de Datos NoSQL#Apache Cassandra - Columnas Anchas|primera mención en la Clase 03]] y la [[Clase 04 - Configuración de Bases de Datos NoSQL#Cassandra: Arquitectura Peer-to-Peer|arquitectura peer-to-peer]] vista en la Clase 04)

## ¿Qué es Apache Cassandra?

Es un sistema de gestión de bases de datos NoSQL distribuido y de código abierto. Fue diseñado por Facebook en 2007 para manejar volúmenes masivos de datos.

- **Híbrido:** Combina Amazon Dynamo (P2P) y Google Bigtable (Tabular).
- **Fault Tolerant:** No tiene punto único de fallo (SPOF).
- **Escalable:** Crecimiento lineal sin pérdida de rendimiento.

## Arquitectura de Anillo (Ring)

Cassandra organiza los nodos en una topología circular. Cada nodo es responsable de un rango de datos determinado por **tokens hash**.

- **Consistnet Hashing:** Distribuye datos uniformemente.
- **Virtual Nodes (vnodes):** Permite que un nodo físico maneje múltiples segmentos del anillo.

![[Drawing 2026-04-18 14.26.00.excalidraw]]

Se utiliza un algoritmo de **Hash (Murmur3)** aplicando a la **Partition Key**. El resultado es un token que ubica el dato en el rango asignado a un nodo específico.

## Comunicación: Protocolo Gossip

Es el método de comunicación descentralizado donde los nodos intercambian información de forma aleatoria (_"chismes"_).

- **Mecanismo:**
	- Cada nodo envía información sobre sí mismo y otros nodos que conoce a un máximo de 3 nodos cada segundo.

## Persistencia: Flujo de Escritura

La escritura es extremadamente rápida porque es secuencial y ocurre en tres niveles:

1. **Commit Log:** Registro inmediato en disco para durabilidad.
2. **Memtable:** Almacenamiento temporal en RAM.
3. **SSTable:** Archivo inmutable final volcado al disco.

## Modelos de Datos

- **Keyspace**
	- Contenedor de nivel superior, similar a una base de datos SQL. Define el factor de replicación.
- **Table**
	- Llamada anteriormente 'Column Family'. Almacena filas con columnas variables.
- **Primary Key**
	- Indispensable. Se divide en **Partition Key** y **Clustering Key**.

## Claves: Partición y Orden

### Partition Key

Determina **en qué nodo** vive el dato. Crítico para el balanceo de carga en el anillo.

### Clustering Key

Determina **el orden** del dato dentro de la partición. Optimiza consultas de rango.

```cql
PRIMARY KEY (pk_id, clustering_col)
```

## Diferencias entre CQL vs. SQL

| Característica       | SQL (Relacional)     | CQL (Cassandra)           |
| -------------------- | -------------------- | ------------------------- |
| **Operaciones JOIN** | Nativas / Amplias    | No existen                |
| **Esquema**          | Rígido / Normalizado | Flexible / Desnormalizado |
| **Consistencia**     | ACID Inmediato       | Eventual (Tuneable)       |
| **Diseño**           | Entidad-Relación     | Orienta a Consultas       |

## Mecanismos de Resiliencia

- **Hinted Handoff**
	- Si el nodo destino está caído, el coordinador guarda la escritura y se la entrega cuando regrese.
- **Bloom Filter**
	- Estructura de datos probabilística que ahorra lecturas de disco innecesarias si el dato no existe.
- **Anti-Entropy**
	- Reparación periódica que compara réplicas mediante árboles de Merkle para sincronizarlas.
