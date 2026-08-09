---
Fecha de creación: 2025-10-17T19:24:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-10-17
---
[[Clase 10 - Joins en MySQL|← Clase anterior]] | [[Clase 13 - Vistas en MySQL|Clase siguiente →]]

# Constraints e Índices en MySQL
(ver [[Constraints]] y [[Indexes]]; complementa las llaves [[Clase 04 - Introducción a Ambientes SQL#MySQL ``PRIMARY KEY CONSTRAINT``|PRIMARY]] y [[Clase 04 - Introducción a Ambientes SQL#MySQL ``FOREIGN KEY CONSTRAINT``|FOREIGN KEY]] ya vistas en la Clase 04)

Los **constraints** (restricciones) son reglas que se aplican a las columnas de una tabla para garantizar la integridad y exactitud de los datos. Impiden que se inserten datos inválidos.

## MySQL ``NOT NULL``
(ver [[Constraints#`NOT NULL` - No Nulo|NOT NULL en SQL]])

Asegura que una columna no pueda tener un valor ``NULL``.

```sql
CREATE TABLE Personas (
    ID int NOT NULL,
    Nombre varchar(255) NOT NULL,
    Edad int
);
```

## MySQL ``UNIQUE``
(ver [[Constraints#`UNIQUE` - Valores Únicos|UNIQUE en SQL]])

Asegura que todos los valores de una columna sean diferentes entre sí. A diferencia de ``PRIMARY KEY``, una tabla puede tener varias columnas ``UNIQUE`` y sí admite un valor ``NULL``.

```sql
CREATE TABLE Personas (
    ID int NOT NULL UNIQUE,
    Nombre varchar(255) NOT NULL,
    Email varchar(255) UNIQUE
);
```

## MySQL ``DEFAULT``
(ver [[Constraints#`DEFAULT` - Valor por Defecto|DEFAULT en SQL]])

Asigna un valor predeterminado a una columna cuando no se especifica ninguno al insertar un registro.

```sql
CREATE TABLE Personas (
    ID int NOT NULL,
    Ciudad varchar(255) DEFAULT 'San Salvador'
);
```

## MySQL ``CHECK``
(ver [[Constraints#`CHECK` - Validación de Valores|CHECK en SQL]])

Se asegura de que los valores de una columna cumplan una condición específica.

```sql
CREATE TABLE Personas (
    ID int NOT NULL,
    Edad int,
    CHECK (Edad >= 18)
);
```

## MySQL ``AUTO_INCREMENT``

Permite generar automáticamente un número único cada vez que se inserta un nuevo registro, comúnmente usado junto con ``PRIMARY KEY``.

```sql
CREATE TABLE Personas (
    ID int NOT NULL AUTO_INCREMENT,
    Nombre varchar(255) NOT NULL,
    PRIMARY KEY (ID)
);
```

# Índices en MySQL
(ver [[Indexes#Tipos de Índices|Tipos de Índices]])

Un **índice** es una estructura que acelera la búsqueda y recuperación de registros en una tabla, a costa de un poco más de espacio y de tiempo en las operaciones de escritura (``INSERT``, ``UPDATE``, ``DELETE``).

## MySQL ``CREATE INDEX``
(ver [[Indexes#Creación de Índices (`CREATE INDEX`)|Creación de Índices]])

```sql
CREATE INDEX idx_nombre
ON Personas (Nombre);
```

Si se quiere que los valores del índice sean únicos:

```sql
CREATE UNIQUE INDEX idx_nombre
ON Personas (Nombre);
```

## MySQL ``DROP INDEX``
(ver [[Indexes#Eliminación de Índices (`DROP INDEX`)|Eliminación de Índices]])

```sql
ALTER TABLE Personas
DROP INDEX idx_nombre;
```

## ¿Cuándo usar índices?
(ver [[Indexes#Cuándo y Dónde Usar Índices|Cuándo y Dónde Usar Índices]])

- En columnas que se consultan frecuentemente en cláusulas ``WHERE``, ``JOIN`` o ``ORDER BY``.
- No abusar de ellos: cada índice adicional ralentiza las operaciones de escritura y ocupa espacio en disco.
