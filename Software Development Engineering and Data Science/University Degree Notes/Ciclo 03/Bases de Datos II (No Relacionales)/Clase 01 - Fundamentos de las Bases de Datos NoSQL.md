---
Fecha de creación: 2026-01-24T14:24:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-01-24
---
# Fundamentos de las Bases de Datos NoSQL

## ¿Qué es NoSQL?

**NoSQL (Not Only SQL)** es un conjunto de sistemas de bases de datos diseñados para manejar:

- Grandes volúmenes de datos
- Datos semi-estructurados o no estructurados
- Aplicaciones distribuidas
- Alta escalabilidad horizontal

>[!IMPORTANT] No significa “sin SQL”, sino que: No dependen exclusivamente del modelo relacional tradicional.

## ¿Por qué surgieron las bases NoSQL?

Durante décadas dominaron las bases relacionales (modelo de **Edgar F. Codd, 1970**).

Ejemplos clásicos:

- Oracle
- MySQL
- PostgreSQL
- SQL Server

Funcionan bajo el modelo **ACID** y usan SQL.

### Problema

Con la llegada de la **Web 2.0 (2000+)**, empresas como: Google, Amazon, Facebook, Twitter empezaron a generar:

- Millones de usuarios
- Datos en tiempo real
- Big Data
- Contenido no estructurado

Las bases relacionales comenzaron a presentar limitaciones:

- Escalabilidad horizontal compleja
- Esquema rígido
- Costos elevados
- Dificultad para manejar datos distribuidos

## Características Principales de NoSQL

- Esquema flexible
- Escalabilidad horizontal
- Replicación automática
- Alta disponibilidad
- Diseñadas para sistemas distribuidos

---

## Teorema CAP (Fundamental)

Propuesto por **Eric Brewer (2000)**. Un sistema distribuido solo puede garantizar 2 de 3 propiedades:

![[Drawing 2026-01-24 14.53.51.excalidraw]]

### C — Consistency

Todos los nodos ven los mismos datos al mismo tiempo.

### A — Availability

Siempre responde a las peticiones.

### P — Partition Tolerance

Sigue funcionando aunque haya fallos de red.

### En sistemas reales:

La tolerancia a particiones es obligatoria. Por lo tanto, el sistema debe elegir entre:

### 🔵 CP (Consistencia + Partición)

Prioriza consistencia. Puede dejar de responder para mantener datos correctos.

Ejemplos:

- MongoDB (configuración por defecto)
- HBase

### 🟢 AP (Disponibilidad + Partición)

Siempre responde. Puede devolver datos desactualizados temporalmente.

Ejemplos:

- Cassandra
- CouchDB
- DynamoDB

### 🟡 CA (Consistencia + Disponibilidad)

Solo posible sin distribución real.  
Ejemplo:

- Base relacional en un solo servidor

---

## ACID vs BASE

### ACID (Relacionales)

- Atomicity
- Consistency
- Isolation
- Durability

Transacciones totalmente confiables.

## BASE (Muchos NoSQL)

- Basically Available
- Soft State
- Eventually Consistent

Los datos eventualmente convergen.

 >[!IMPORTANT] No garantiza consistencia inmediata.

---

# Tipos de Bases NoSQL

## 1. Clave–Valor

Modelo más simple.

```nginx
clave → valor
```

**Ejemplo:**

|Key|Value|
|---|---|
|user1|{JSON}|
|user2|{JSON}|
### Características:

- Ultra rápidas
- Muy escalables
- Ideal para caché

### Casos de uso:

- Sesiones
- Carrito de compras
- Contadores
- Rate limiting

### Ejemplos:

- Redis
- DynamoDB
- Riak

## 2. Documentales

Almacenan documentos JSON/BSON.

**Ejemplo:**

```json
{
  "nombre": "Dereck",
  "edad": 26,
  "intereses": ["IA", "Data Science"]
}
```

### Características:

- Esquema flexible
- Soporta consultas complejas
- Índices avanzados

### Casos de uso:

- Perfiles de usuario
- Catálogos de productos
- CMS
- Apps web

### Ejemplos:

- MongoDB
- CouchDB
- Firestore

## 3. Columnares

Almacenamiento orientado a columnas.

**Ejemplo:**

| column1 | column2 | column3 |
| ------- | ------- | ------- |
| 1       | 2       | 3       |
| 4       | 5       | 6       |
| 7       | 8       | 9       |

Optimizado para:

- Big Data
- Analítica
- Series temporales

### Casos de uso:

- IoT
- Logs
- Data Warehousing

### Ejemplos:

- Cassandra
- HBase
- Bigtable

## 4. Bases de Datos de Grafos

Modelan relaciones.

Estructura:

- Nodos
- Aristas
- Propiedades

Ideal para:

- Redes sociales
- Recomendadores
- Detección de fraude

**Ejemplo conceptual:**

```scss
(Ana) —[amiga_de]→ (Carlos)
```

![[Drawing 2026-01-24 17.21.00.excalidraw]]

Ejemplos:

- Neo4j
- ArangoDB
- Amazon Neptune

---

## Arquitecturas Distribuidas

### Master-Slave

Un nodo escribe. Otros replican.

### Peer-to-Peer

Todos los nodos iguales.  
Ejemplo: Cassandra.

### Multi-Master

Múltiples nodos aceptan escritura.

## Ventajas de NoSQL

- Escalabilidad horizontal
- Manejo eficiente de Big Data
- Alta disponibilidad
- Flexibilidad de desarrollo
- Costos reducidos en infraestructura distribuida

## Desventajas

- Consistencia eventual
- Menor estandarización
- Complejidad operativa
- Transacciones limitadas
- Curva de aprendizaje

### Casos de Éxito Documentados

- **Twitter:** Utiliza Manhattan (sistema key-value interno) para timeline y tweets.
- **Uber:** Redis para geolocalización en tiempo real de conductores y pasajeros.
- **LinkedIn:** Voldemort para almacenamiento de perfiles y grafos sociales.
- **Spotify:** Cassandra para perfiles de usuario y playlists.

## Las 4 V del Big Data

1. Volumen
2. Velocidad
3. Variedad
4. Veracidad

NoSQL surge para responder a estas 4 V.
