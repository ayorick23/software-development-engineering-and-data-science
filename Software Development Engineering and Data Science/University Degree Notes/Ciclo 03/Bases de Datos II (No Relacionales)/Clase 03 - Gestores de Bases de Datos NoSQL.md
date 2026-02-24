---
Fecha de creación: 2026-02-07T18:41:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-02-07
---
# Gestores de Bases de Datos NoSQL

## Diferencias Clave entre Gestores NoSQL

No todas las bases NoSQL son iguales. Aunque comparten la filosofía de escalabilidad horizontal y flexibilidad, cada gestor fue diseñado para resolver problemas distintos.

En esta clase analizamos tres enfoques principales:

- MongoDB → Modelo documental
- Apache Cassandra → Almacén de columnas anchas
- Azure Cosmos DB → Multi-modelo distribuido globalmente

## MongoDB - Modelo Documental

MongoDB almacena información en documentos [[JSON and YAML#Archivos JSON|JSON]]/``BSON``.

Características conceptuales:

- Esquema flexible
- Documentos auto-contenidos
- Orientado a desarrollo ágil
- Fácil de integrar con aplicaciones web

Se adapta muy bien a:

- Sistemas de e-commerce
- Perfiles de usuario
- Catálogos
- Aplicaciones web modernas

El enfoque es orientado a **lectura eficiente y flexibilidad estructural**.

## Apache Cassandra - Columnas Anchas

Cassandra no es relacional ni documental.  
Es un sistema distribuido orientado a:

- Escritura masiva
- Alta disponibilidad
- Escalabilidad horizontal pura
- Arquitectura peer-to-peer (sin master)

Su modelo se basa en:

- Keyspaces (similar a bases de datos)
- Tablas diseñadas según patrones de consulta
- Claves primarias obligatorias para consultas eficientes

Es ideal para:

- IoT (millones de sensores)
- Sistemas de logs
- Plataformas de streaming
- Datos en tiempo real

Aquí el diseño de tabla depende del tipo de consulta que se realizará.

## Azure Cosmos DB - Multi-modelo

Azure Cosmos DB permite trabajar con:

- Documentos
- Columnas
- Grafos
- Clave–valor

Está diseñado para:

- Distribución global automática
- Baja latencia mundial
- Integración nativa con servicios cloud

Es una solución empresarial orientada a la nube.

## Introducción a Docker

### ¿Qué es Docker?
(ver [[Introduction to Docker]])

Docker es una plataforma de contenedores que permite ejecutar aplicaciones aisladas dentro de entornos portables y ligeros llamados _contenedores_.

Un contenedor:

- Incluye el software
- Incluye dependencias
- Funciona igual en cualquier sistema
- Es más ligero que una máquina virtual

En esta práctica utilizamos Docker para:

- Levantar instancias locales de MongoDB
- Levantar instancias locales de Cassandra
- Simular entornos reales de producción

## Práctica 1: MongoDB con Docker

Verificar versión de Docker:

```bash
docker --version
```

Crear contenedor MongoDB:

```bash
docker run -d --name mongo-clase3 -p 9002:27017 mongo:latest
```

Parámetros:

- `-d`: modo detached
- `--name`: nombre del contenedor
- `-p 9002:27017`: puerto local 9002 mapeado al 27017 del contenedor
- ``mongo:latest``: imagen de Docker Hub.

Ver contenedores activos:

```bash
docker ps
```

Entrar al contenedor:

```bash
docker exec -it mongo-clase3 mongosh
```

### Comandos básicos en MongoDB

Mostrar las bases de datos en el modelo y usar una base en específico:

```javascript
SHOW DBS;
USE clase3;
```

Crear colección e insertar documento:

```javascript
db.usuarios.insertOne({
  nombre: "Dereck",
  email: "dereck@email.com",
  edad: 26
})
```

Consulta básica:

```javascript
db.usuarios.find()
```

Consulta con filtro:

```javascript
db.usuarios.find({ edad: 26 })
```

Actualizar:

```javascript
db.usuarios.updateOne(
  { nombre: "Carlos" },
  { $set: { edad: 26 } }
)
```

### MongoDB Compass

MongoDB Compass es la interfaz gráfica oficial.

Conexión:

```shell
mongodb://localhost:9002
```

Permite:

- Ver bases de datos
- Crear documentos
- Ejecutar consultas gráficamente
- Visualizar estructura JSON

## Práctica 2: Cassandra con Docker

Crear contenedor:

```bash
docker run -d --name cassandra-clase3 -p 9042:9042 cassandra:latest
```

Acceder a CQL Shell:

```bash
docker exec -it cassandra-clase3 cqlsh
```

### Comandos básicos en Cassandra

Ver keyspaces:

```cql
DESCRIBE KEYSPACES;
```

Crear keyspace:

```cql
CREATE KEYSPACE clase3 
WITH replication = {
  'class': 'SimpleStrategy', 
  'replication_factor': 1
};
```

Seleccionar keyspace:

```cql
USE clase3;
```

Crear tabla:

```cql
CREATE TABLE usuarios (
  id UUID PRIMARY KEY,
  nombre TEXT,
  email TEXT,
  edad INT
);
```

Ver tablas:

```cql
DESCRIBE TABLES;
```

Insertar datos:

```cql
INSERT INTO usuarios (id, nombre, email, edad)
VALUES (uuid(), 'Dereck', 'dereck@email.com', 26);
```

Consulta básica:

```cql
SELECT * FROM usuarios;
```

### Diferencia importante con WHERE

En Cassandra:

El `WHERE` solo puede utilizar columnas que formen parte de la clave primaria.

**Ejemplo válido:**

```cql
SELECT * FROM usuarios WHERE id = 123e4567-e89b-12d3-a456-426614174000;
```

**Ejemplo inválido:**

```cql
SELECT * FROM usuarios WHERE edad = 28;
```

Esto falla porque `edad` no es clave primaria.

Esto refleja una diferencia filosófica:

> En Cassandra se diseña la tabla según la consulta, no la consulta según la tabla.

### DBeaver

DBeaver es una herramienta universal para gestión de bases de datos.

Para Cassandra:

- Instalar driver JDBC manual
- Conectar a localhost:9042

Permite:

- Ejecutar CQL
- Visualizar tablas
- Administrar estructuras
