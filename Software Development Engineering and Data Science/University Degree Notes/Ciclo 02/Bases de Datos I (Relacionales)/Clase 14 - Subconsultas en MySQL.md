---
Fecha de creación: 2025-10-17T18:06:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-10-17
---
# Subconsultas en MySQL
(ver [[Subqueries]])

## Subconsulta

En SQL, Las subconsultas (también conocidas como consultas internas o consultas anidadas) son una herramienta para realizar operaciones en varios pasos. Por ejemplo, si quisiera sumar varias columnas y promediar todos esos valores, necesitaría realizar cada agregación en un paso distinto.

Las subconsultas se pueden usar en varios lugares dentro de una consulta, pero es más fácil empezar con la instrucción [[DQL (Data Query Language)#2. `FROM` (_Origen de los Datos_)|FROM]]. A continuación, se muestra un ejemplo de una subconsulta básica con tablas que tienen datos sobre la delincuencia en San Francisco:

```sql
SELECT sub.*
FROM(
	SELECT *
	FROM tutorial.sf_crime_incidents_2014_01
	WHERE day_of_week = 'Friday'
) sub
 WHERE sub.resolution = 'NONE'
```

Analicemos lo que sucede cuando se ejecuta la consulta anterior:

Primero, la base de datos ejecuta la "consulta interna", la parte entre paréntesis:

```sql
SELECT *
FROM tutorial.sf_crime_incidents_2014_01
WHERE day_of_week = 'Friday'
```

Si se ejecutara esto por sí solo, produciría un conjunto de resultados como cualquier otra consulta. Puede parecer obvio, pero es importante: la consulta interna debe ejecutarse por sí sola, ya que la base de datos la tratará como una consulta independiente. Una vez ejecutada la consulta interna, la consulta externa se ejecutará utilizando los resultados de la consulta interna como tabla subyacente:

```sql
SELECT sub.*
FROM(
   <<aquí van los resultados de la consulta interna>>
) sub
WHERE sub.resolution = 'NONE'
```

Las subconsultas deben tener nombres, que se añaden entre paréntesis de la misma forma que se añadiría un alias a una tabla normal . En este caso, hemos usado el nombre "sub".

Al usar subconsultas, es importante recordar que es importante que el lector determine fácilmente qué partes de la consulta se ejecutarán juntas. La mayoría de las personas lo hacen sangrando la subconsulta de alguna manera.

### Uso de Consultas para Agregar en Múltiples Etapas

Las subconsultas pueden ser muy útiles para casos en los que se necesitan pre-cálculos antes de dar un resultado final. Por ejemplo, teniendo las siguientes tablas del ejemplo anterior: _¿Qué sucedería si quisiera calcular cuántos incidentes se reportan cada día de la semana?_ Mejor aún, _¿Qué sucedería si quisiera saber cuántos incidentes ocurren, en promedio, un viernes de diciembre? ¿Y en enero?_ Este proceso consta de dos pasos: contar el número de incidentes diarios (consulta interna) y determinar el promedio mensual (consulta externa).

La subconsulta sería ejemplificada de la siguiente forma:

```sql
SELECT LEFT(sub.date, 2) AS cleaned_month,
	sub.day_of_week,
	AVG(sub.incidents) AS average_incidents
FROM(
	SELECT day_of_week,
		date,
		COUNT(incidnt_num) AS incidents
	FROM tutorial.sf_crime_incidents_2014_01
	GROUP BY 1,2
) sub
GROUP BY 1,2
ORDER BY 1,2
```

### Subconsultas en Lógica Condicional

Podemos utilizar subconsultas en lógica condicional (junto con [[DQL (Data Query Language)#3. `WHERE` (_Filtrado de Filas_)|WHERE]], [[Clase 10 - Joins en MySQL#MySQL Inner Join|JOIN]]/``ON`` o [[Clase 08 - Funciones de Control de Flujo y Condición#``CASE``|CASE]]). La siguiente consulta devuelve todas las entradas desde la fecha más antigua del conjunto de datos:

```sql
SELECT *
FROM tutorial.sf_crime_incidents_2014_01
WHERE Date = (SELECT MIN(date)
              FROM tutorial.sf_crime_incidents_2014_01
              )
```

Otro ejemplo con [[Clase 03 - Sentencias SQL en MySQL#MySQL ``IN``|IN]] podría ser el siguiente:

```sql
SELECT *
FROM tutorial.sf_crime_incidents_2014_01
WHERE Date IN (SELECT date
               FROM tutorial.sf_crime_incidents_2014_01
               ORDER BY date
               LIMIT 5
               )
```

Otra forma de elaborar la sentencia del ejemplo anterior sería la siguiente:

```sql
SELECT *
FROM tutorial.sf_crime_incidents_2014_01 incidents
JOIN( SELECT date
	  FROM tutorial.sf_crime_incidents_2014_01
	  ORDER BY date
      LIMIT 5
      ) sub
ON incidents.date = sub.date
```

Un último ejemplo combinando diferentes funciones:

```sql
SELECT incidents.*,
       sub.incidents AS incidents_that_day
FROM tutorial.sf_crime_incidents_2014_01 incidents
JOIN( SELECT date,
	  COUNT(incidnt_num) AS incidents
	  FROM tutorial.sf_crime_incidents_2014_01
	  GROUP BY 1
      ) sub
ON incidents.date = sub.date
ORDER BY sub.incidents DESC, time
```

## MySQL ``EXISTS``

El operador ``EXISTS`` se utiliza para probar la existencia de cualquier registro en una subconsulta.

Devuelve **VERDADERO** si la subconsulta devuelve uno o más registros.

**La sintaxis es la siguiente:**

```sql
SELECT nombre_columna(s)
FROM tabla
WHERE EXISTS
(SELECT nombre_columna FROM tabla WHERE condición);
```

**Ejemplos:**

```sql
SELECT SupplierName
FROM Suppliers
WHERE EXISTS (SELECT ProductName
			  FROM Products 
			  WHERE Products.SupplierID = Suppliers.supplierID AND Price < 20);
```

```sql
SELECT SupplierName
FROM Suppliers
WHERE EXISTS (SELECT ProductName 
			  FROM Products
			  WHERE Products.SupplierID = Suppliers.supplierID AND Price = 22);
```

## MySQL ``ANY``, ``ALL``

Los operadores ``ANY`` y ``ALL`` le permiten realizar una comparación entre un solo valor de columna y un rango de otros valores.

### Operador ``ANY``

- Devuelve un valor booleano como resultado.
- Devuelve **VERDADERO** si **CUALQUIERA** de los valores de la subconsulta cumple la condición.
- ``ANY`` significa que la condición será verdadera si la operación es verdadera para cualquiera de los valores en el rango.

**Sintaxis de ``ANY``:**

```sql
SELECT nombre_columna(s)
FROM tabla
WHERE nombre_columna operador ANY
  (SELECT nombre_columna
  FROM tabla
  WHERE condición);
```

El operador debe ser un operador de comparación estándar (``=``, ``<>``, ``!=``, >, ``>=``, ``<``, o ``<=``).

### Operador ``ALL``

- Devuelve un valor booleano como resultado.
- Devuelve **VERDADERO** si **TODOS** los valores de la subconsulta cumplen la condición.
- Se utiliza con las declaraciones ``SELECT``, ``WHERE`` y ``HAVING``.
- ALL significa que la condición será verdadera solo si la operación es verdadera para todos los valores en el rango.

**Sintaxis de ``ALL``:**

```sql
SELECT ALL nombre_columna(s)
FROM tabla
WHERE condición;
```

```sql
SELECT nombre_columna(s)
FROM tabla
WHERE nombre_columna operador ALL
  (SELECT nombre_columna
  FROM tabla
  WHERE condición);
```

A continuación algunos ejemplos:

```sql
SELECT ProductName
FROM Products
WHERE ProductID = ANY
  (SELECT ProductID
  FROM OrderDetails
  WHERE Quantity = 10);
```

```sql
SELECT ProductName
FROM Products
WHERE ProductID = ALL
  (SELECT ProductID
  FROM OrderDetails
  WHERE Quantity = 10);
```
