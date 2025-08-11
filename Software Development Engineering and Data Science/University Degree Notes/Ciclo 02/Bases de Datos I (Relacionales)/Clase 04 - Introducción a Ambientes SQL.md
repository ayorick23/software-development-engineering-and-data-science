---
Fecha de creación: 2025-08-09T15:58:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-08-09
---
# Instalación de MySQL

## Crear Bases de Datos y Tablas en MySQL
(Similar a [[DDL (Data Definition Language)]])

### MySQL ``CREATE DATABASE``

Esta sentencia es utilizada para crear una nueva base de datos.

Para crear una base de datos es necesario tener privilegios de administrador en el entorno MySQL.

La sintaxis es la siguiente:

```sql
CREATE DATABASE nombre_de_basededatos;
```

Una vez creada la base de datos podemos chequear que esté creada con el siguiente comando:

```sql
SHOW DATABASES;
```

### MySQL ``DROP DATABASE``

Esta sentencia es utilizada para eliminar una base de datos existente.

Hay que tener mucho cuidado con esta sentencia dado que la consecuencia es que al eliminar una base de datos estaríamos perdiendo toda la información almacenada.

La sintaxis es la siguiente:

```sql
DROP DATABASE nombre_de_basededatos;
```

### MySQL ``CREATE TABLE``

Esta sentencia es utilizada para crear una tabla dentro de una base de datos existente.

La sintaxis es la siguiente:

```sql
CREATE TABLE tabla (  
    columna1 tipo_de_dato,  
    columna2 tipo_de_dato,  
    columna3 tipo_de_dato,  
   ....  
);
```

Hay que hacer notar que existen diferentes tipos de datos que revisaremos más adelante en esta asignatura con más profundidad.

También como variante es posible crear una tabla a partir de otra ya existente.

La sintaxis es la siguiente:

```sql
CREATE TABLE nueva_tabla AS  
    SELECT columna1, columna2,...  
    FROM tabla_existente  
    WHERE ....;
```

### MySQL ``DROP TABLE``

Esta sentencia es utilizada para eliminar una tabla existente dentro de una base de datos.

Hay que tener mucho con cuidado con esta sentencia pues la consecuencia es eliminar toda la información contenida en la tabla.

La sintaxis es la siguiente:

```sql
DROP TABLE tabla;
```

### MySQL ``ALTER TABLE``

Esta sentencia es utilizada para añadir, eliminar o modificar columnas dentro de una tabla.

La sintaxis para añadir una columna es la siguiente:

```sql
ALTER TABLE tabla  
ADD columna tipo_de_dato;
```

La sintaxis para eliminar una columna es la siguiente:

```sql
ALTER TABLE tabla  
DROP COLUMN columna;
```

La sintaxis para cambiar el tipo de datos de una columna existente es la siguiente:

```sql
ALTER TABLE tabla  
MODIFY COLUMN columna datatype;
```

## Llaves Primarias y Llaves Foráneas en MySQL

En el lenguaje de MySQL existen diferentes formas de asignar reglas de validación de datos, las 2 mas importantes para la integridad de los datos son:

### MySQL ``PRIMARY KEY CONSTRAINT``
(ver [[Constraints#`PRIMARY KEY` - Clave Primaria|PRIMARY KEY en SQL]])

Se utiliza para identificar de manera única un registro en una tabla. Las llaves primarias deben contener valores "únicos" y no pueden contener valores ``NULL``, es decir no pueden ser nulos.

Una tabla puede tener una sola [[Constraints#`PRIMARY KEY` - Clave Primaria|llave primaria]] y dentro de la definición de dicha tabla puede ser una sola columna o la combinación de varias columnas (campos).

La sintaxis se explica mejor con el siguiente ejemplo donde al momento de crear la tabla "Personas" queremos de una vez asignar la columna "ID" como llave primaria:

```sql
CREATE TABLE Personas (  
    ID int NOT NULL,  
    Apellido varchar(255) NOT NULL,  
    Nombre varchar(255),  
    Edad int,  
    PRIMARY KEY (ID)  
);
```

Si en el ejemplo anterior quisiéramos que la llave primaria estuviera compuesta de varias columnas como ``ID`` y ``Apellido``, la sintaxis sería la siguiente:

```sql
CREATE TABLE Personas (  
    ID int NOT NULL,  
    Apellido varchar(255) NOT NULL,  
    Nombre varchar(255),  
    Edad int,  
    CONSTRAINT PK_Persona PRIMARY KEY (ID,Apellido)  
);
```

Ahora revisaremos como creamos una llave primaria con la sentencia ``ALTER TABLE``:

Con una columna:

```sql
ALTER TABLE Personas  
ADD PRIMARY KEY (ID);
```

Con varias columnas:

```sql
ALTER TABLE Personas  
ADD CONSTRAINT PK_Person PRIMARY KEY (ID,Apellido);
```

Para eliminar una llave primaria existente utilizamos la siguiente sentencia:

```sql
ALTER TABLE Personas  
DROP PRIMARY KEY;
```

### MySQL ``FOREIGN KEY CONSTRAINT``
(ver [[Constraints#`FOREIGN KEY` - Clave Foránea|FOREIGN KEY en SQL]])

Es usada para que no se destruyan las relaciones entre tablas, es de las sentencias más importantes en SQL para asegurar la integridad de los datos. Una [[Constraints#`FOREIGN KEY` - Clave Foránea|llave foránea]] por definición es un campo (o colección de campos) en una tabla, que hace referencia a la llave primaria de otra tabla.

La tabla con la llave foránea es llamada "tabla hija", y la tabla con la llave primaria es llamada la tabla referenciada o "tabla padre".

Para la sintaxis lo explicaremos con un ejemplo:

```sql
CREATE TABLE Ordenes (  
    OrdenID int NOT NULL,  
    OrdenNumero int NOT NULL,  
    PersonaID int,  
    PRIMARY KEY (OrdenID),  
    FOREIGN KEY (PersonaID) REFERENCES Persona(PersonaID)  
);
```

Una ``FOREIGN KEY`` puede ser nombrada de la siguiente forma:

```sql
CREATE TABLE Ordenes (  
    OrdenID int NOT NULL,  
    OrdenNumero int NOT NULL,  
    PersonaID int,  
    PRIMARY KEY (OrdenID),  
    CONSTRAINT FK_PersonaOrden FOREIGN KEY (PersonaID)  
    REFERENCES Personas(PersonaID)  
);
```

En ambos ejemplos la llave foránea se ha definido entre la "tabla hija" que es "Ordenes" y la "tabla padre" que es ``Personas`` a través del campo ``PersonaID`` que está en ambas tablas.