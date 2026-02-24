---
Fecha de creación: 2026-01-31T13:39:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-01-31
---
# SQL vs. NoSQL

## Contexto Histórico y Necesidad Tecnológica

Las bases de datos **[[Clase 02 - Estándar SQL#Definición de Estándar SQL|SQL]] (relacionales)** dominan la industria desde los años 70, cuando el modelo relacional fue propuesto por Edgar F. Codd. Su diseño responde a una necesidad clara: **garantizar integridad, consistencia y manejo estructurado de datos con relaciones complejas**.

Durante décadas, motores como:

- PostgreSQL
- MySQL
- Oracle Database
- Microsoft SQL Server

han sido la base de sistemas bancarios, ERP, inventarios y plataformas financieras.

Sin embargo, con la llegada de la Web 2.0 y el crecimiento masivo de datos generados por usuarios, empresas como: Google, Amazon, Facebook comenzaron a enfrentar problemas de escalabilidad que el modelo relacional tradicional no resolvía eficientemente a gran escala distribuida. De ahí surge el movimiento [[Clase 01 - Fundamentos de las Bases de Datos NoSQL#¿Qué es NoSQL?|NoSQL]], no como reemplazo, sino como respuesta a nuevos requerimientos.

## Diferencia Fundamental

La diferencia entre SQL y NoSQL no es solo técnica, sino filosófica.

### SQL: Consistencia como prioridad

Las bases relacionales están diseñadas bajo el principio [[Clase 01 - Fundamentos de las Bases de Datos NoSQL#ACID (Relacionales)|ACID]], lo que garantiza:

- Transacciones atómicas
- Consistencia inmediata
- Aislamiento entre operaciones
- Durabilidad permanente

En otras palabras, si una operación ocurre, ocurre completamente o no ocurre.

Este enfoque es indispensable en sistemas financieros donde perder consistencia es inaceptable.

### NoSQL: Escalabilidad y disponibilidad como prioridad

Las bases NoSQL suelen basarse en el principio [[Clase 01 - Fundamentos de las Bases de Datos NoSQL#BASE (Muchos NoSQL)|BASE]] (Basically Available, Soft State, Eventually Consistent).

Aquí la consistencia puede ser eventual, lo que significa que los datos pueden tardar un breve tiempo en sincronizarse entre nodos distribuidos.

Este modelo es adecuado cuando:

- Se prioriza disponibilidad constante
- Se manejan millones de usuarios concurrentes
- Se trabaja en sistemas globalmente distribuidos

## Modelo de Datos y Organización

### SQL: Modelo Relacional

Los datos se organizan en tablas con un esquema rígido previamente definido. Cada columna tiene un tipo específico y existen restricciones formales.

Ejemplo clásico de comercio electrónico:

```sql
Clientes(id, nombre, email, direccion)
Pedidos(id, cliente_id, fecha, total)
Productos(id, nombre, precio, stock)
Detalle_Pedido(pedido_id, producto_id, cantidad)
```

Las relaciones se gestionan mediante claves foráneas y consultas JOIN:

```sql
SELECT c.nombre, p.fecha, p.total
FROM Clientes c
JOIN Pedidos p ON c.id = p.cliente_id;
```

Este modelo favorece:

- Integridad estructural
- Relaciones complejas
- Consultas analíticas consistentes

Pero puede volverse costoso en sistemas masivos distribuidos.

### NoSQL: Modelos Flexibles

NoSQL no utiliza un único modelo. Puede adoptar diferentes estructuras según el tipo de motor:

- Documental
- Clave–valor
- Columnar
- Grafos

Por ejemplo, en MongoDB, los datos se almacenan como documentos ``JSON``:

```json
{
  "_id": "pedido_001",
  "cliente": {
    "nombre": "Claudia Pérez",
    "email": "claudia@email.com"
  },
  "productos": [
    {"nombre": "Laptop", "precio": 1200, "cantidad": 1},
    {"nombre": "Mouse", "precio": 25, "cantidad": 2}
  ],
  "fecha": "2024-10-15",
  "total": 1250
}
```

Aquí los datos relacionados se almacenan juntos (desnormalización), lo que reduce la necesidad de [[Joins#Joins|JOIN]] y mejora el rendimiento de lectura.

## Esquema: Rígido vs Flexible

En SQL, modificar la estructura implica cambios formales:

```sql
ALTER TABLE users ADD COLUMN telefono VARCHAR(20);
```

En bases con millones de registros, esto puede requerir planificación, migraciones y posibles tiempos de inactividad.

En contraste, en MongoDB simplemente se agrega el nuevo campo:

```javascript
db.users.insertOne({
  name: "Ana",
  telefono: "555-1234"
})
```

Esta flexibilidad favorece metodologías ágiles, aunque puede generar inconsistencias si no se controla adecuadamente.

## Lenguaje de Consulta

SQL es un lenguaje estandarizado desde 1986 (ANSI/ISO). Su sintaxis es relativamente universal entre motores relacionales.

NoSQL no posee un estándar único. Cada motor define su propia sintaxis:

- **MongoDB:** _MongoDB Query Language_ **(MQL)** basado en JavaScript.

```javascript
db.users.updateOne(
	{_id: ObjectId("")
	},
	{$set: {satus: "activo"}}
)
```

- **Apache Cassandra:** _Cassandra Query Language_ (**CQL**) similar a SQL pero con limitaciones.

```cql
INSERT INTO users (id, name, status)
VALUES (1001, 'Marcos', 'activo');

SELECT status FROM users WHERE id = 1001;
```

- **Redis:** Comandos específicos (``GET``, ``SET``, ``HGET``)

```Redis
SET user:1001:status "activo"
GET user:1001:status
```

- **Neo4j:** Cypher para consultas de grafos

Esta diversidad aumenta la curva de aprendizaje y dificulta la portabilidad entre sistemas.

## Manejo de Relaciones

SQL fue diseñado específicamente para manejar relaciones complejas. Los JOIN permiten combinar múltiples tablas con integridad garantizada.

NoSQL generalmente evita JOIN. En su lugar utiliza:

- **Embedding (incrustación):** Datos relacionados dentro del mismo documento.
- **Referencing (referencias):** IDs que apuntan a otros documentos, requiriendo múltiples consultas.
- **Desnormalización:** Duplicación intencional de datos.

Esto optimiza la lectura pero puede complicar actualizaciones cuando hay duplicación de información.

## Escalabilidad: Vertical vs Horizontal

### Escalabilidad Vertical (Scale-Up)

Tradicional en SQL. Se mejora el hardware del servidor: más RAM, CPU o almacenamiento.

**Ventajas:**

- Arquitectura simple
- Transacciones completas ACID

**Limitaciones:**

- Costos exponenciales
- Límite físico del hardware
- Punto único de falla

### Escalabilidad Horizontal (Scale-Out)

Común en NoSQL. Se agregan más nodos al sistema distribuido.

**Ventajas:**

- Crecimiento casi ilimitado
- Costos lineales
- Alta disponibilidad
- Distribución geográfica

**Limitaciones:**

- Complejidad distribuida
- Consistencia eventual
- Conflictos de sincronización

## Casos Reales en la Industria

### Netflix

Utiliza Apache Cassandra para procesar miles de millones de eventos diarios.  
Necesita alta disponibilidad global y escritura masiva distribuida.

### Uber

Usa arquitectura híbrida:

- PostgreSQL para datos transaccionales críticos.
- Apache Cassandra para datos de geolocalización en tiempo real.

>[!IMPORTANT] Esto demuestra que el debate no es SQL vs NoSQL, sino **qué herramienta usar según el problema**.
