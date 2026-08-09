---
Fecha de creación: 2025-09-18T18:48:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-09-18
---
[[Clase 09 - Funciones de Agregación y Agrupación|← Clase anterior]] | [[Clase 11 - Constraints e Índices en MySQL|Clase siguiente →]]

# MySQL Joins
(ver [[Joins]])

Esta sentencia es utilizada para combinar registros de 2 o más tablas basado en la relación entre ellas.

Para una mejor ilustración del concepto observemos las siguientes tablas:

**Tabla ``Orders``:**

| OrderID | CustomerID | OrderDate  |
| ------- | ---------- | ---------- |
| 10308   | 2          | 1996-09-18 |
| 10309   | 37         | 1996-09-19 |
| 10310   | 77         | 1996-09-20 |

**Tabla ``Customers``:**

| CustomerID | CustomerName                       | ContactName    | Country |
| ---------- | ---------------------------------- | -------------- | ------- |
| 1          | Alfreds Futterkiste                | Maria Anders   | Germany |
| 2          | Ana Trujillo Emparedados y helados | Ana Trujillo   | Mexico  |
| 3          | Antonio Moreno Taquería            | Antonio Moreno | Mexico  |

Notemos que la columna ``CustomerID`` en la tabla ``Orders`` se refiere a la columna ``CustomerID`` en la tabla ``Customers``.

## MySQL Inner Join
(ver [[Clase 03 - Sentencias SQL en MySQL#MySQL ``INNER JOIN``|MySQL INNER JOIN]])

Selecciona los registros que coinciden los valores entre 2 tablas.

Se ilustra de la siguiente forma:

![[Drawing 2025-09-18 18.53.55.excalidraw]]

**Sintaxis:**

```sql
SELECT columnas
FROM tabla1
INNER JOIN tabla2
ON tabla1.columna = tabla2.columna;
```

**Ejemplo:**

```sql
SELECT Orders.OrderID, Customers.CustomerName
FROM Orders
INNER JOIN Customers ON Orders.CustomerID = Customers.CustomerID;
```

## MySQL Left Join

Selecciona los registros de la tabla de la izquierda y los que coinciden con la de la derecha, si existen.

![[Drawing 2025-09-18 18.58.00.excalidraw]]

**Sintaxis:**

```sql
SELECT columnas
FROM tabla1
LEFT JOIN tabla2
ON tabla1.columna = tabla2.columna;
```

**Ejemplo:**

```sql
SELECT Customers.CustomerName, Orders.OrderID
FROM Customers
LEFT JOIN Orders ON Customers.CustomerID = Orders.CustomerID
ORDER BY Customers.CustomerName;
```

## MySQL Right Join

Selecciona los registros de la tabla de la derecha y los que coinciden con los de la izquierda, si existen.

![[Drawing 2025-09-18 19.01.15.excalidraw]]

**Sintaxis:**

```sql
SELECT columnas
FROM tabla1
RIGHT JOIN tabla2
ON tabla1.columna = tabla2.columna;
```

**Ejemplo:**

```sql
SELECT Orders.OrderID, Employees.LastName, Employees.FirstName
FROM Orders
RIGHT JOIN Employees ON Orders.EmployeeID = Employees.EmployeeID
ORDER BY Orders.OrderID;
```

## MySQL Cross Join
(ver [[Clase 03 - Sentencias SQL en MySQL#MySQL ``CROSS JOIN``|MySQL CROSS JOIN]])

Selecciona todos los registros de dos tablas.

![[Drawing 2025-09-18 20.39.05.excalidraw]]

**Sintaxis:**

```sql
SELECT columnas
FROM tabla1
CROSS JOIN tabla2;
```

**Ejemplo:**

```sql
SELECT Customers.CustomerName, Orders.OrderID
FROM Customers
CROSS JOIN Orders;
```

## MySQL Self Join
(ver [[Clase 03 - Sentencias SQL en MySQL#MySQL ``SELF JOIN``|MySQL SELF JOIN]])

Es un join normal lo único es que es sobre la misma tabla.

**Sintaxis:**

```sql
SELECT columnas
FROM tabla1 T1, tabla1 T2
WHERE condición;
```

**Ejemplo:**

```sql
SELECT A.CustomerName AS CustomerName1, B.CustomerName AS CustomerName2, A.City
FROM Customers A, Customers B
WHERE A.CustomerID <> B.CustomerID
AND A.City = B.City
ORDER BY A.City;
```
