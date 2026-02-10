---
Fecha de creación: 2026-01-31T13:39:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-01-31
---
# SQL vs. NoSQL

Las bases de datos SQL se diseñaron para garantizar consistencia transaccional estricta (ACID) y manejar datos altamente estructurados con relaciones complejas. Son excelentes para aplicaciones donde la integridad de datos es crítica: sistemas bancarios, ERP, gestión de inventarios y contabilidad.

Por otro lado, las bases de datos NoSQL surgieron como respuesta a limitaciones específicas de escalabilidad horizontal, flexibilidad de esquema y disponibilidad continua. Empresas como Google, Amazon y Facebook pioneras en el desarrollo de soluciones NoSQL para manejar volúmenes de datos sin precedentes (Bradshaw et al., 2024).

## Análisis Comparativo

Las bases de datos NoSQL destacan en escenarios donde se manejan grandes volúmenes de datos, alta velocidad de escritura y estructuras flexibles. Su escalabilidad horizontal permite distribuir datos en múltiples nodos, ofreciendo rendimiento óptimo en aplicaciones como redes sociales, IoT y análisis de big data.

### Bases de Datos SQL
(ver [[Clase 02 - Estándar SQL]])

Organizan información en tablas con filas y columnas. Cada tabla tiene un esquema rígido predefinido que especifica tipos de datos, restricciones y relaciones. Las entidades relacionadas se distribuyen en múltiples tablas conectadas mediante claves foráneas. Este diseño requiere normalización para eliminar redundancia y garantizar consistencia (Carpenter & Hewitt, 2024).

**Ejemplo, en un sistema de comercio electrónico SQL:**

```shell
Tabla Clientes (id, nombre, email, dirección)
Tabla Pedidos (id, cliente_id, fecha, total)
Tabla Productos (id, nombre, precio, stock)
Tabla Detalle_Pedido (pedido_id, producto_id, cantidad)
```

### Bases de Datos NoSQL

Ofrecen múltiples modelos de datos según el tipo: documentos (``JSON``/``BSON``), clave-valor, columnar o grafos. No requieren esquema predefinido, permitiendo que cada registro tenga estructura diferente. Los datos relacionados frecuentemente se almacenan juntos (desnormalización) para optimizar lectura (Bradshaw et al., 2024).

**Ejemplo, sistema de comercio electrónico en MongoDB (documental):**

```json
{ "_id"- "pedido_001":
	"cliente": {
		"nombre": "Claudia Pérez"
		"email": "claudia@email-com"
		"productos": [
			{"nombre": 'Laptop": "precio": 1200, "cantidad": l}:
			{"nombre": 'Mouse", "precio": 25: "cantidad": 2}
	"fecha": "2024-10-15", "total": 1250 }
```

## Lenguaje de Consulta

### SQL (Structured Query Language)

Lenguaje estándar estandarizado por ANSI/ISO desde 1986. Las consultas son declarativas, especificando QUÉ datos se desean sin detallar CÓMO obtenerlos. SQL es universal entre bases relacionales (PostgreSQL, MySQL, Oracle, SQL Server) con variaciones menores de dialecto (Joyanes Aguilar, 2013).

### NoSQL:

No existe un lenguaje universal. Cada sistema tiene su propia sintaxis:

- MongoDB: MongoDB Query Language (MQL) basado en JavaScript
- Cassandra: CQL (Cassandra Query Language) similar a SQL pero con limitaciones
- Neo4j: Cypher para consultas de grafos
- Redis: Comandos específicos (``GET``, ``SET``, ``HGET``)

Esta falta de estandarización complica la portabilidad y requiere curvas de aprendizaje específicas para cada tecnología (Carpenter & Hewitt, 2024).

### Esquema y Flexibilidad

**SQL - Esquema Rígido:**

Cambios en la estructura requieren migraciones formales mediante comandos ``ALTER TABLE``. Agregar una columna a una tabla con millones de registros puede requerir downtime significativo. Esta rigidez garantiza consistencia pero reduce agilidad en desarrollo iterativo.

```sql
-- POSTGRESQL
INSERT INTO users  (id, name, status)
VALUES (1001, 'Marcos', 'activo');
SELECT status FROM users WHERE id = 1001;

-- MySQL
INSERT INTO users  (id, name, status)
VALUES (1001, 'Marcos', 'activo');
SELECT status FROM users WHERE id = 1001;

-- Oracle
INSERT INTO users  (id, name, status)
VALUES (1001, 'Marcos', 'activo');
SELECT status FROM users WHERE id = 1001;

-- SQL Server
INSERT INTO users  (id, name, status)
VALUES (1001, 'Marcos', 'activo');
SELECT status FROM users WHERE id = 1001;
```

**NoSQL - Esquema Flexible:**

Los documentos pueden tener campos diferentes sin modificar configuración. Los desarrolladores agregan campos simplemente usándolos. Esta flexibilidad es ideal para desarrollo ágil donde requisitos evolucionan rápidamente, pero puede generar inconsistencias si no se gestiona adecuadamente (Bradshaw et al., 2024).

```json
#Redis
SET user:1001:status "activo"
GET user:1001:status
```

```json
#MongoDB
db.users.insertOne({
	_id: ObjectId(""),
	name: "Marcos",
	status: "pendiente"
})
db.users.updateOne(
	{_id: ObjectId("")
	},
	{$set: {satus: "activo"}}
)
```

```cql
#Cassandra
INSERT INTO users (id, name, status)
VALUES (1001, 'Marcos', 'activo');

SELECT status FROM users WHERE id = 1001;
```

### Relaciones entre Datos

**SQL:** Diseñado específicamente para manejar relaciones complejas mediante [[Joins]]. Las consultas pueden combinar eficientemente datos de múltiples tablas relacionadas. Soporta integridad referencial mediante claves foráneas y constraints.

**NoSQL:** Generalmente evita ``JOINs``. Las relaciones se manejan mediante:

- **Embedding (incrustación):** Datos relacionados dentro del mismo documento.
- **Referencing (referencias):** IDs que apuntan a otros documentos, requiriendo múltiples consultas.
- **Desnormalización:** Duplicación intencional de datos.

Este enfoque optimiza lecturas pero complica actualizaciones cuando datos duplicados deben sincronizarse (Joyanes Aguilar, 2015).

## Escalabilidad Vertical vs. Horizontal

| Criterio       | Escalabilidad Vertical (Scale-Up)                                                                                                                                                                                                                                                                                   | Escalabilidad Horizontal (Scale-Out)                                                                                                                                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Tipo de Escala | Tradicionalmente "hacia arriba" escalan (scale-up) mejorando el hardware del servidor: más CPU, RAM, discos más rápidos. Este enfoque tiene limitaciones físicas y costos exponenciales. Un servidor con ITB de RAM cuesta significativamente más que el doble de un servidor con 512GB (Carpenter & Hewitt, 2024). | Escalan "hacia afuera" (scale-<br>out) distribuyendo datos entre múltiples servidores commodity. Agregar capacidad significa incorporar nodos adicionales al clúster. Este enfoque es prácticamente ilimitado y<br>económicamente viable (Bradshaw et 2024). |
| Ventajas       | Simplicidad operacional (un solo<br>servidor). Sin complejidad de<br>sincronización distribuida. Transacciones ACID completas                                                                                                                                                                                       | Escalabilidad casi ilimitada<br>Costos lineales (duplicar<br>capacidad = duplicar servidores). Alta disponibilidad mediante<br>replicación. Distribución geográfica.                                                                                         |
| Limitaciones   |                                                                                                                                                                                                                                                                                                                     |                                                                                                                                                                                                                                                              |
| Optimización   |                                                                                                                                                                                                                                                                                                                     |                                                                                                                                                                                                                                                              |

## Casos de Estudio

**Netflix y Cassandra:**

Netflix procesa más de 1 billón de eventos diarios usando Apache Cassandra. La capacidad de escritura horizontal les permite ingestar telemetría de millones de dispositivos simultáneamente sin degradación de rendimiento. SQL sería prohibitivamente costoso a esta escala (Carpenter & Hewitt, 2024).

**Uber y PostgreSQL/Cassandra:**

Uber inicialmente usó PostgreSQL (SQL) pero migró componentes específicos a Cassandra. Los datos de geolocalización en tiempo real requieren escrituras masivas distribuidas globalmente, mejor manejadas por NoSQL. Sin embargo, mantienen PostgreSQL para datos transaccionales críticos donde ACID es esencial (Joyanes Aguilar, 2015).
