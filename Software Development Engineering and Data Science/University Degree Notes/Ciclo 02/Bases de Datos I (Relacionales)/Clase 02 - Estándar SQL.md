---
Fecha de creación: 2025-07-18T18:34:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-07-18
---
# Definición de Estándar SQL

Las siglas [[Clase 01 - Definición de Base de Datos#Conceptos Claves|SQL]] significan en el idioma inglés "Structured Query Language" traducido al español como "Lenguaje de Consulta Estructurada". SQL permite acceder y manipular bases de datos.

SQL se convirtió en un estándar ANSI en 1986 e ISO en 1987.

## ¿Qué puedo hacer con SQL?

- Ejecutar consultas a la base de datos
- Recuperar datos
- Insertar registros
- Actualizar registros
- Eliminar registros
- Crear nuevas bases de datos
- Crear nuevas tablas dentro de una base de datos

## ¿Cómo utilizo SQL en las aplicaciones?

Por ejemplo, si tengo que crear un “sitio web” que muestre datos de una base de datos como    mínimo necesito lo siguiente:

- Un RDBMS
- Un lenguaje para utilizar del lado del servidor como PHP o ASP
- Utilizar SQL para acceder los datos
- Utilizar HTML / CSS para darle estilo a las páginas web

## ¿Qué es un RDBMS?

En inglés significa “Relational Database Management System”, o en español “Sistema de gestión de bases de datos relacionales”. Es la base para SQL y para todos los sistemas de bases de datos como Microsoft SQL Server, IBM DB2, Oracle, MySQL y Microsoft Access.

## Sintaxis de SQL

Las "declaraciones" SQL consisten en distintas palabras claves que son fáciles de entender.

Para consultar todos los registros de una tabla, por ejemplo, supongamos una base de datos que una de sus tablas es "Clientes" y quiero consultar todos sus registros se utiliza la declaración [[DML (Data Manipulation Language)#`SELECT`|SELECT]]:

```sql
SELECT * FROM Clientes;
```

## Comandos de SQL

De los comandos más importantes de SQL están los siguientes:
- **[[DML (Data Manipulation Language)#`SELECT`|SELECT]] –** extrae datos de una base de datos
- **[[DML (Data Manipulation Language)#`UPDATE`|UPDATE]] –** actualiza datos en una base de datos
- **[[DML (Data Manipulation Language)#`DELETE`|DELETE]] –** elimina datos en una base de datos
- **[[DML (Data Manipulation Language)#`INSERT`|INSERT]] INTO –** adiciona nuevos datos en una base de datos
- **[[DDL (Data Definition Language)#`CREATE`|CREATE]] DATABASE –** crea una nueva base de datos
- **[[DDL (Data Definition Language)#`ALTER`|ALTER]] DATABASE –** modifica una base de datos
- **CREATE TABLE –** crea una nueva tabla
- **ALTER TABLE –** modifica una tabla
- **DROP TABLE –** elimina una tabla
- **CREATE INDEX –** crea un [[Indexes#Indexes|índice]] (llave de búsqueda)
- **DROP INDEX –** elimina un índice.

### Comando ``SELECT``

**SELECT** es utilizado como su nombre la indica para seleccionar datos de una base de datos.

Sintaxis:
```sql
SELECT columna1, columna2,...
FROM tabla;
```

Si se desea retornar todas las columnas se utiliza de la siguiente forma:
```sql
SELECT * FROM tabla;
```

Si de una misma tabla se quisieran los valores diferentes de una o más columnas:
```sql
SELECT DISTINCT columna1, columna2,...
FROM tabla;
```

Si de una misma tabla queremos filtrar datos, es decir, poner una condición de búsqueda utilizamos la cláusula [[DQL (Data Query Language)#3. `WHERE` (_Filtrado de Filas_)|WHERE]].
```sql
SELECT columna1, columna2,...
FROM tabla
WHERE condición;
```

## Ejemplo de Uso

Supongamos que tenemos la tabla CLIENTES del ejemplo inicial y queremos que esta tabla nos devuelva los registros que cumplan con la condición de que dichos clientes sean del país México, la declaración SQL sería la siguiente:
```sql
SELECT * FROM CLIENTES
WHERE Country='Mexico';
```

Importante mencionar que para la condición WHERE se pueden utilizar distintos operadores (ver [[DQL (Data Query Language)#Operadores Comunes en `WHERE`]])

## Más Comandos de SQL

### ``ORDER BY``

Sirve para ordenar los registros en orden ascendente o descendente.

Sintaxis:
```sql
SELECT columna1, columna2,… ...  
FROM tabla  
ORDER BY columna1, columna2, ... ASC|DESC;
```

### ``AND``

Es un operador que se utiliza dentro de la cláusula WHERE para colocar distintas condiciones anidadas, es decir, si todas las condiciones se cumplen.

Sintaxis:
```sql
SELECT columna1, columna2,… ...  
FROM tabla  
WHERE condición1 AND condición2 AND condición3 ...;
```

### ``OR``

Se utiliza dentro de la cláusula WHERE para colocar distintas condiciones anidadas, pero a diferencia de la cláusula AND, los registros se extraen si alguna de las condiciones se cumple.

Sintaxis:
```sql
SELECT columna1, columna2, ...  
FROM tabla  
WHERE condición1 OR condición2 OR condición3 ...;
```

### ``NOT``

Es un operador que combinado con otros operadores devuelve el resultado opuesto.

Sintaxis:
```sql
SELECT columna1, columna2, ...  
FROM tabla  
WHERE **NOT** condición;
```

### ``NULL``

Es un operador que se usa para identificar valores “vacíos” dentro de columnas de una tabla de bases de datos.

Sintaxis:
```sql
SELECT columna1, columna2, ...  
FROM tabla  
WHERE columnan is NULL;
```

### ``TOP``

Es una cláusula que se usa para limitar la cantidad de registros que devuelve una consulta SELECT.

En los distintos manejadores de bases de datos podría tener algunas variantes, pero en general su sintaxis es la siguiente:

**SQL Server:**
```sql
SELECT **TOP** número|porcentaje columnas  
FROM tabla  
WHERE condición;
```

**MySQL:**
```sql
SELECT columnas  
FROM tabla  
WHERE condición  
**LIMIT** número;
```

### ``LIKE``

Es utilizado en la cláusula WHERE para buscar un patrón en una columna.

Sintaxis:
```sql
SELECT columna1, columna2, ...  
FROM tabla  
WHERE columnaN LIKE patrón;
```

## Funciones de Agregación
(ver [[Aggregate Functions]])

Las funciones de agregación (en inglés: aggregate functions) se utilizan para realizar cálculos de un conjunto de valores que retornan un solo valor.

En SQL las más comúnmente utilizadas son las siguientes:
- **MIN() –** retorna el menor valor de los valores de una columna
- **MAX() –** retorna el máximo valor de los valores de una columna
- **COUNT() –** retorna el conteo de registros de una tabla de acuerdo a las condiciones establecidas
- **SUM() –** retorna la suma total de una columna de valor numérico
- **AVG() –** retorna el valor promedio de una columna de valor numérico.

## SQL Joins
(ver [[Joins]])

Es una cláusula para combinar registro de dos o más tablas en una sola sentencia SELECT.

Hay diferentes tipos de JOINS en SQL:
- **(INNER) JOIN:** retorna registros que coinciden en ambas tablas.
- **LEFT (OUTER) JOIN:** retorna todos los registros de la tabla izquierda y los que coinciden con la tabla derecha.
- **RIGHT (OUTER) JOIN:** retorna todos los registros de la tabla derecha y los que coinciden con la tabla izquierda.
- **FULL (OUTER) JOIN:** retorna todos los registros cuando haya una coincidencia ya sea en la table izquierda o en la derecha.
