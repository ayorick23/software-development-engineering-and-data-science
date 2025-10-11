---
Fecha de creación: 2025-09-12T19:01:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-09-12
---
# Funciones de Agregación
(ver [[SQL/Aggregate Functions|Aggregate Functions]])

## Función ``MIN()``

Retorna el valor más pequeño de un conjunto de valores de una columna seleccionada.

**Sintaxis:**

```sql
SELECT MIN(columna)
FROM tabla
WHERE condición;
```

**Ejemplo:**

```sql
SELECT MIN(Precio) AS PrecioMenor
FROM Productos;
```

## Función ``MAX()``

Retorna el valor más grande de un conjunto de valores de una columna seleccionada.

**Sintaxis:**

```sql
SELECT MAX(columna)
FROM tabla
WHERE condición;
```

**Ejemplo:**

```sql
SELECT MAX(Precio) AS PrecioMayor
FROM Productos;
```

## Función ``COUNT()``

Retorna el número de registros de una tabla según condiciones establecidas.

**Sintaxis:**

```sql
SELECT COUNT(columna)
FROM tabla
WHERE condición;
```

**Ejemplo:**

```sql
SELECT COUNT(ProductoID)
FROM Productos;
```

## Función ``AVG()``

Retorna el valor promedio de una columna numérica según condiciones establecidas.

**Sintaxis:**

```sql
SELECT AVG(columna)
FROM tabla
WHERE condición;
```

**Ejemplo:**

```sql
SELECT AVG(Precio)
FROM Productos;
```

## Función ``SUM()``

Retorna el total de la suma de valores de una columna numérica según condiciones establecidas.

**Sintaxis:**

```sql
SELECT SUM(columna)
FROM tabla
WHERE condición;
```

**Ejemplo:**

```sql
SELECT SUM(Cantidad)
FROM DetalleOrdenes;
```

En estas últimas 3 funciones de agregación es importante mencionar que los valores ``NULL`` son ignorados.

# Sentencias de Agrupación

Sentencia [[Clase 03 - Sentencias SQL en MySQL#MySQL ``GROUP BY``|GROUP BY]]: como su nombre lo dice, agrupa registros con los mismo valores, se utiliza con las funciones de agregación revisadas anteriormente.

**Sintaxis:**

```sql
SELECT columnas
FROM tabla
WHERE condición
GROUP BY columnas
ORDER BY columnas;
```

Notemos en la sintaxis que estamos utilizando la sentencia [[DQL (Data Query Language)#6. `ORDER BY` (_Ordenación de Resultados_)|ORDER BY]], como su nombre lo dice, ordena los registros de acuerdo con el orden de las columnas indicadas en dicha sentencia.

**Ejemplo:**

```sql
SELECT COUNT(ClienteID), País
FROM Clientes
GROUP BY País
ORDER BY COUNT(ClienteID) DESC;
```

En combinación con la sentencia ``GROUP BY`` se utiliza para hacer condicionado de registros la cláusula [[Clase 03 - Sentencias SQL en MySQL#MySQL ``HAVING``|HAVING]].

**Sintaxis:**

```sql
SELECT columnas
FROM tabla
WHERE condición
GROUP BY columnas
HAVING condiciones
ORDER BY columnas;
```
