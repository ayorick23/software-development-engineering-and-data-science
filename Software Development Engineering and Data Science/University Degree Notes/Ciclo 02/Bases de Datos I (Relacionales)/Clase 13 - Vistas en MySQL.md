---
Fecha de creación: 2025-10-10T17:43:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-10-10
---
# Vistas
(ver [[Views]])

En SQL, una **vista** es una tabla virtual basada en un conjunto de resultados de una sentencia SQL. Una vista contiene filas y columnas, tal como una tabla real. Los campos en una vista son campos de una o más tablas reales en una base de datos.

En una vista podemos añadir sentencias y funciones SQL y presentar la data tal como si estuviera siendo consultada una tabla.

## MySQL ``CREATE VIEW``

Esta sentencia es utilizada para crear una vista.

**Sintaxis:**

```sql
CREATE VIEW nombre_vista AS
SELECT columna1, columna2, ...
FROM nombre_tabla
WHERE condición;
```

**Ejemplo:**

```sql
CREATE VIEW [Products Above Average Price] AS
SELECT ProductName, Price
FROM Products
WHERE Price > (SELECT AVG(Price) FROM Products);
```

**Uso:**

```sql
SELECT * FROM [Products Above Average Price];
```

## MySQL ``CREATE OR REPLACE VIEW``

Esta sentencia es utilizada para actualizar una vista.

**Sintaxis:**

```sql
CREATE OR REPLACE VIEW nombre_vista AS
SELECT columna1, columna2, ...
FROM nombre_tabla
WHERE condición;
```

**Ejemplo:**

```sql
CREATE OR REPLACE VIEW [Brazil Customers] AS
SELECT CustomerName, ContactName, City
FROM Customers
WHERE Country = 'Brazil';
```

## MySQL ``DROP VIEW``

Esta sentencia es utilizada para eliminar una vista.

**Sintaxis:**

```sql
DROP VIEW nombre_vista;
```

**Ejemplo:**

```sql
DROP VIEW [Brazil Customers];
```

### Creando una vista para 2 tablas en MySQL

```sql
CREATE TABLE customers (
    id bigint primary key auto_increment,
    first_name varchar(100),
    last_name varchar(100),
    country_code varchar(2)
);

CREATE TABLE orders (
    order_id bigint primary key auto_increment,
    cust_id bigint,
    order_date datetime default now(),
    constraint foreign key (cust_id) references customers(id)
);

INSERT INTO customers (first_name, last_name, country_code) VALUES ("tim", "sehn", "ca");

INSERT INTO customers (first_name, last_name, country_code) VALUES ("aaron", "son", "us");

INSERT INTO customers (first_name, last_name, country_code) VALUES ("brian", "hendriks", "us");

-- Insertemos 2 órdenes por cada cliente

INSERT INTO orders (cust_id) VALUES (1), (1), (2), (2), (3), (3);

CREATE VIEW customer_orders
    (id, first_name, last_name, country_code, order_id, cust_id, order_date)
    AS SELECT id, first_name, last_name, country_code, order_id, cust_id, order_date
    FROM CUSTOMERS c JOIN orders o
    ON c.id = o.cust_id;
```

### Examinando Vistas en MySQL

```sql
SHOW FULL TABLES;
```

| Tables_in_rest  | Table_type |
| --------------- | ---------- |
| customer_orders | VIEW       |
| customers       | BASE TABLE |
| orders          | BASE TABLE |

Para ver la descripción de una vista:

```sql
DESC customer_orders;
```

| Field        | Type         | Null | Key | Default           | Extra             |
| ------------ | ------------ | ---- | --- | ----------------- | ----------------- |
| id           | bigint       | NO   |     | 0                 |                   |
| firt_name    | varchar(100) | YES  |     | NULL              |                   |
| last_name    | varchar(100) | YES  |     | NULL              |                   |
| country_code | varchar(2)   | YES  |     | NULL              |                   |
| order_id     | bigint       | NO   |     | 0                 |                   |
| cust_id      | bigint       | YES  |     | NULL              |                   |
| order_date   | datetime     | YES  |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |

Para obtener la definición completa de una vista:

```sql
SHOW CREATE VIEW customer_orders;
```

Para obtener más información sobre la vista definida:

```sql
SELECT *
FROM views
WHERE table_name = 'customer_orders'\G
```

Diversas utilizaciones de vistas:

```sql
SELECT *
FROM customer_orders
WHERE country_code = 'ca';

SELECT *
FROM orders o
JOIN customer_orders co ON o.order_id = co.cust_id
WHERE country_code = 'ca';

SELECT first_name, last_name
FROM customers
WHERE id NOT IN
	(SELECT cust_id
	FROM customer_orders
	WHERE country_code = 'us');
```

Vistas referenciadas por otras vistas:

```sql
CREATE VIEW us_customer_orders AS
SELECT *
FROM customer_orders
WHERE country_code = 'us';
```