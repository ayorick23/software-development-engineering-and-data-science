---
Fecha de creación: 2026-02-14T14:11:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-02-14
---
[[Clase 03 - Gestores de Bases de Datos NoSQL|← Clase anterior]] | [[Clase 05 - Prueba de Concepto (POC)|Clase siguiente →]]

# Configuración e Implementación de Bases de Datos NoSQL

Implementar una base de datos NoSQL no es simplemente instalar el software.  
Implica diseñar:

- La arquitectura del clúster
- La topología de red
- La estrategia de replicación
- El dimensionamiento del hardware
- Los mecanismos de respaldo y recuperación
- Los niveles de consistencia

En entornos empresariales, una mala configuración puede provocar:

- Pérdida de datos
- Cuellos de botella
- Alta latencia
- Fallos de disponibilidad

Por eso esta clase se enfoca en la **infraestructura y configuración avanzada**.

## Introducción Básica a Servidores (Conceptos Fundamentales)

### ¿Qué es un servidor?

Un servidor es una máquina (física o virtual) que proporciona servicios a otras máquinas (clientes).

Puede ser:

- Físico (hardware dedicado, por ejemplo un servidor Dell en un rack)
- Virtual (máquina virtual sobre VMware o similar)
- En la nube (instancias en Azure, AWS, etc.)

Ejemplo mencionado en clase:

> Servidor DELL → Virtualizado con VMware → Máquina virtual → MongoDB instalado → IP privada 10.0.10.4

## Recursos importantes del servidor

Cuando implementamos NoSQL debemos calcular:

### CPU

Número de núcleos disponibles para:

- Procesar consultas
- Ejecutar replicación
- Manejar concurrencia

### RAM

Crítica en NoSQL. En MongoDB, por ejemplo:

> Los índices deben caber en RAM para evitar acceso constante a disco.

Si los índices no caben en memoria:

- Aumenta la latencia
- Se degrada el rendimiento

### Disco

Debe considerarse:

- Tipo (SSD recomendado)
- IOPS
- Capacidad futura
- Crecimiento estimado

## Desafíos en la Implementación NoSQL

No es recomendable instalar una base NoSQL en el mismo servidor que una base SQL productiva porque:

- Compiten por CPU y RAM
- Tienen patrones de uso distintos
- Generan cargas diferentes (lectura vs escritura)
- Pueden interferir en rendimiento

En producción, se recomienda:

- Separación de roles
- Aislamiento de recursos
- Arquitecturas dedicadas

## Topologías y Arquitecturas de Clúster

Un clúster es un conjunto de nodos que trabajan como un sistema distribuido.

Cada motor NoSQL tiene su propia arquitectura.

### MongoDB: Replica Set

MongoDB utiliza una arquitectura llamada **Replica Set**. Un Replica Set típico tiene:

- 1 nodo primario (_Primary_)
- 2 nodos secundarios (_Secondary_)

Ejemplo con 3 servidores:

```nginx
Servidor 1 (Primary)   → 10.0.10.4
Servidor 2 (Secondary) → 10.0.10.5
Servidor 3 (Secondary) → 10.0.10.6
```

Funcionamiento:

- El Primary recibe escrituras.
- Los Secondary replican los datos.
- Si el Primary falla, se elige automáticamente uno nuevo.

Esto garantiza:

- Alta disponibilidad
- Tolerancia a fallos

## Cassandra: Arquitectura Peer-to-Peer

Apache Cassandra no tiene nodo primario.

Todos los nodos:

- Aceptan lectura y escritura
- Se replican entre sí
- Se organizan por datacenters

Esto permite:

- Escalabilidad horizontal pura
- Alta tolerancia a fallos
- Distribución geográfica

### Azure Cosmos DB

Azure Cosmos DB maneja:

- Replicación automática global
- Multi-región
- Configuración administrada desde la nube

El usuario define:

- Regiones activas
- Nivel de consistencia
- Throughput

## Configuración de la Topología

Al diseñar el clúster debemos definir:

- Número de nodos
- Ubicación (datacenters)
- Segmentación de red
- IPs privadas
- Balanceo de carga

También es necesario optimizar:

- Latencia entre nodos
- Ancho de banda
- Seguridad (firewalls, puertos)

## Gestión de Memoria y Almacenamiento

En MongoDB:

- Los índices deben caber en RAM.
- WiredTiger usa cache interna (~50% de RAM disponible).
- Se recomienda SSD.

En Cassandra:

- Memtables (en memoria)
- SSTables (en disco)
- Commit log para durabilidad

Una mala configuración puede generar:

- Lecturas lentas
- Escrituras bloqueadas
- Compactaciones excesivas

## Estrategias de Particionamiento y Sharding

Cuando los datos crecen demasiado para un solo nodo, se usa **sharding**.

## ¿Qué es Sharding?

Es dividir los datos en múltiples nodos llamados shards.

En MongoDB:

- Se define una shard key.
- Los datos se distribuyen automáticamente.

Arquitectura típica:

```shell
Cliente → Router (mongos) → Config Server → Shards
```

Tipos de particionamiento:

- Rango
- Hash
- Geográfico

Una mala shard key puede causar:

- Hotspots
- Distribución desigual
- Bajo rendimiento

## Replicación y Consistencia

## MongoDB

Configuración mediante Write Concern y Read Preference.

Permite definir:

- Escritura confirmada por cuántos nodos
- Lectura desde primario o secundarios

## Cassandra

Define niveles de consistencia como:

- **ONE:** Solo una réplica debe responder. Proporciona la menor latencia y mayor disponibilidad, ideal para alta velocidad de escritura, pero arriesga consistencia eventual.
- **QUORUM:** La mayoría de las réplicas ($\frac{n}{2}+1$) deben responder. Ofrece una consistencia fuerte en todo el clúster.
- **ALL:** Todas las réplicas deben responder. Ofrece la mayor consistencia pero la menor disponibilidad (si un nodo cae, la operación falla).

Ejemplo:

Si replication_factor = 3:

- ONE → responde 1 nodo
- QUORUM → mayoría (2 nodos)
- ALL → los 3 nodos

Esto permite balancear entre: Consistencia y Disponibilidad.

## Sistemas Operativos Recomendados

MongoDB recomienda:

- Linux (Ubuntu Server, RHEL, Debian)
- No recomendado en producción Windows

En entornos empresariales:

- Servidores virtualizados
- Discos SSD
- Red dedicada

## Backups en NoSQL

A diferencia de SQL, donde pueden usarse triggers o backups integrados en el motor, en MongoDB los respaldos se realizan mediante herramientas externas.

## Práctica: MongoDB Backup y Restore

### Backup

Verificar que el contenedor esté activo:

```bash
docker ps
```

Crear backup dentro del contenedor:

```bash
mongodump --out dumpClase4
```

Copiar al host:

```bash
docker cp mongo-clase3:/dumpClase4 .
```

### Restore

Copiar backup al contenedor:

```bash
docker cp mongoBackup mongo-clase3:/mongoBackup
```

Entrar al contenedor:

```bash
docker exec -it mongo-clase3 /bin/bash
```

Restaurar:

```bash
mongorestore mongoBackup
```
