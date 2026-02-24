---
Fecha de creación: 2026-02-21T14:07:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-02-21
---
# ¿Qué es una Prueba de Concepto (POC)?

Una **Prueba de Concepto (Proof of Concept)** es una implementación pequeña y controlada que se realiza antes de un despliegue completo, con el objetivo de validar que una tecnología realmente cumple con los requisitos técnicos y de negocio.

No es un sistema productivo. Es un entorno de validación.

## Ejemplo General

### Sistema tradicional

- **Fase 1 – POC:** 2 usuarios, pruebas locales.
- **Fase 2 – Servicio completo:** 8 usuarios, ambiente productivo.

### En Bases de Datos No Relacionales

**Fase 1 – POC**

- 1 servidor
- MongoDB standalone
- Validación de modelado y consultas

**Fase 2 – Servicio completo**

- Replica Set (3 servidores)
- Máquinas virtuales (por ejemplo en VMWare)
    - Servidor 1
    - Servidor 2
    - Servidor 3

Aquí ya hablamos de alta disponibilidad y tolerancia a fallos.

## Objetivos de una POC en NoSQL

### 1. Validación técnica

- Probar modelado de datos (documentos embebidos vs referencias).
- Validar consultas.
- Probar agregaciones.
- Verificar si se necesitan transacciones.

### 2. Rendimiento

- Medir latencias CRUD.
- Evaluar throughput bajo concurrencia.
- Identificar cuellos de botella.

### 3. Escalabilidad

- Simular crecimiento de usuarios.
- Probar sharding.
- Medir comportamiento ante grandes volúmenes de datos.

### 4. Reducción de riesgos

- Detectar limitaciones técnicas.
- Evaluar curva de aprendizaje del equipo.
- Identificar malas decisiones de modelado temprano.

Una POC bien hecha evita rediseños costosos después.

## Estructura Jerárquica en MongoDB

MongoDB tiene una estructura jerárquica diferente a SQL.

```plain text
Database (Base de datos)
└── Collection (Colección)
	└──	Document (Documento BSON)
		└──	Field: Value (Campo: Valor)
			└──	Embedded Document / Array
```

### Comparación con SQL

|SQL|MongoDB|
|---|---|
|Base de datos|Database|
|Tabla|Collection|
|Fila|Document|
|Columna|Field|

MongoDB usa **BSON (Binary JSON)**, lo que permite estructuras anidadas y arreglos dentro del mismo documento.

## CRUD en MongoDB (Comparación SQL vs NoSQL)

### CREATE

En SQL:

```sql
INSERT INTO Users (name, age, email, city)
VALUES ('Dereck', 26, 'dereck@mail.com', 'San Salvador');
```

En MongoDB (NoSQL):

```javascript
use clase4

// Insertar uno
db.users.insertOne({
  name: "Dereck",
  age: 26,
  email: "dereck@mail.com",
  city: "San Salvador"
})
```

 >[!IMPORTANT] Si no se proporciona `_id`, MongoDB genera automáticamente un **ObjectId de 12 bytes**.
 
 Insertar múltiples documentos:
 
```javascript
db.users.insertMany([
  {name: "Ana", age: 25, city: "Santa Ana"},
  {name: "Luis", age: 30, city: "San Miguel"}
])
```

### READ

En MongoDB se utiliza `find()`.

Consulta básica en SQL:

```sql
SELECT * FROM Users;
```

Consulta básica en MongoDB (NoSQL):

```javascript
db.users.find({age: 25})
```

 >[!IMPORTANT] Las consultas son case sensitive.
 
#### Proyección (seleccionar columnas específicas)

En SQL:

```sql
SELECT name FROM Users WHERE age = 25;
```

En MongoDB (NoSQL):

```javascript
db.users.find(
  {age: 25},
  {name: 1, _id: 0}
)
```

Aquí se reduce transferencia de datos y mejora rendimiento.

#### Operadores Comunes

En SQL:

```sql
SELECT * FROM Users WHERE age > 30; -- Mayor que
SELECT * FROM Users WHERE age >= 30; -- Mayor o igual que
SELECT * FROM Users WHERE age < 30; -- Menor que
SELECT * FROM Users WHERE age <= 30; -- Menor o igual que
SELECT * FROM Users WHERE age <> 25; -- Diferente que

SELECT * FROM Users WHERE age IN (20,25,30); -- IN
SELECT * FROM Users WHERE age = 25 AND city = 'San Salvador'; -- AND
SELECT * FROM Users WHERE age = 25 OR city = 'San Salvador'; -- OR

SELECT * FROM Users WHERE name REGEXP 'Der'; -- RegEx

SELECT * FROM Users ORDER BY age DESC; -- Ordenamiento
```

En MongoDB (NoSQL):

```javascript
db.users.find({age: {$gt: 30}}) // Greater than
db.users.find({age: {$gte: 30}}) // Greater than or equal
db.users.find({age: {$lt: 30}}) // Lower than
db.users.find({age: {$lte: 30}}) // Lower than or equal
db.users.find({age: {$ne: 25}}) // Not equal

db.users.find({age: {$in: [20,25,30]}}) // IN
db.users.find({
  age: 25,
  city: "San Salvador"
}) // AND
db.users.find({
  $or: [
    {age: 25},
    {city: "Santa Ana"}
  ]
}) // OR

db.users.find({
  name: {$regex: "Der"}
}) // RegEx

db.users.find().sort({age: -1}) // Ordenamiento
```

- ``1`` - ascendente
- ``-1`` - descendente

### UPDATE

En SQL:

```sql
UPDATE Users
SET age = 27
WHERE name = 'Dereck';
```

En MongoDB (NoSQL):

```javascript
db.users.updateOne(
  {name: "Dereck"},
  {$set: {age: 27}}
)
```

Actualizar múltiples documentos:

```javascript
db.users.updateMany(
  {city: "San Salvador"},
  {$set: {department: "San Salvador"}}
)
```

### UPSERT (UPdate + inSERT)

Si no existe, lo crea.

```javascript
db.users.updateOne(
  {email: "nuevo@mail.com"},
  {$set: {name: "Nuevo", age: 22}},
  {upsert: true}
)
```

Muy útil para sincronización de datos.

### DELETE

En SQL:

```sql
DELETE FROM Users WHERE age = 30;
```

En MongoDB (NoSQL):

```javascript
db.users.deleteOne({age: 30})
```

Eliminar múltiples documentos:

```javascript
db.users.deleteMany({city: "San Miguel"})
```

## Introducción a Optimización con Índices

Un índice es una estructura que permite buscar información más rápido.

Sin índice:

- MongoDB escanea toda la colección (collection scan).

Con índice:

- Usa estructura tipo árbol (B-Tree).
- Reduce tiempos de búsqueda.
