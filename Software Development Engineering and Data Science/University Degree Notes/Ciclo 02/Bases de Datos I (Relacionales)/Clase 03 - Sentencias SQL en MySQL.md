---
Fecha de creación: 2025-07-25T18:05:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-07-25
---
# ¿Qué es MySQL?

MySQL es un sistema manejador de bases de datos relacionales.
- Es de código abierto
- Es gratis
- Es ideal para pequeñas y grandes aplicaciones
- Es muy rápida, confiable, escalable y fácil de usar
- Es multi-plataforma
- Cumple con el estándar ANSI SQL
- Fue lanzado en 1995
- Es desarrollado, distribuido y apoyado por la corporación Oracle
- Es usado por websites como Facebook, Twitter, Airbnb, etc., también por Wordpress y por supuesto por un largo número de desarrolladores web alrededor del mundo.
- MySQL fue adquirido por Oracle Corporation en el 2008.

## Sentencias SQL en MySQL

Dado que MySQL cumple con el estándar ANSI SQL lo aprendido en el contenido anterior aplica muy bien con las sentencias `SELECT`.

### MySQL `LIMIT`
(Misma teoría de [[DQL (Data Query Language)#7. `LIMIT` / `TOP` / `FETCH NEXT` (_Limitar Resultados_)|LIMIT / TOP / FETCH NEXT]] en SQL Server)

Es una variante del estándar SQL para especificar el número de registros que retornará una consulta.

```sql
SELECT columnas
FROM tabla
WHERE condición
LIMIT número;
```

### MySQL Wildcards

El comodín es utilizado para sustituir uno o más caracteres dentro de un string, es utilizado con el operador ``LIKE`` en una cláusula [[DQL (Data Query Language)#3. `WHERE` (_Filtrado de Filas_)|WHERE]] para especificar un patrón en una columna de búsqueda.

| Símbolo | Descripción                   | Ejemplo                                   |
| ------- | ----------------------------- | ----------------------------------------- |
| ``%``   | Representa 0 a más caracteres | ``bl%`` encontraría bl, black, blue, blob |
| ``-``   | Representa un solo caracter   | ``h_t`` encontraría hot, hat o hit        |

### MySQL ``IN``

Permite especificar múltiples valores en una cláusula ``WHERE``, es una alternativa a tener múltiples condiciones ``OR``.

```sql
SELECT columnas
FROM tabla
WHERE columna IN (valor1, valor2, ...);

-- O

SELECT columnas
FROM tabla
WHERE columna IN (SELECT …);
```

### MySQL `BETWEEN`

Selecciona valores dentro de un rango, pueden ser números, textos o fechas. Es inclusivo, quiere decir que los valores iniciales y finales están incluidos.

```sql
SELECT columna
FROM tabla
WHERE columna BETWEEN valor1 AND valor2;
```

### MySQL Aliases

Los "_alias_" son nombres que se pueden dar a una tabla, columna como un nombre temporal. Se utilizan para que sean más entendibles. El alias existe solo para la consulta donde se está utilizando.

```sql
SELECT columna AS alias
FROM tabla;

SELECT columnas
FROM tabla AS alias;
```

### MySQL ``INNER JOIN``
(Misma teoría de [[Joins#1. `INNER JOIN` (Unión Interna)|INNER JOIN]] en SQL Server)

Selecciona los registros que coinciden entre dos tablas.

```sql
SELECT columnas
FROM tabla1
INNER JOIN tabla2
ON tabla1.columna = tabla2.columna;
```

### MySQL ``CROSS JOIN``
(Misma teoría de [[Joins#5. `CROSS JOIN` (Producto Cartesiano)|CROSS JOIN]] en SQL Server)

Retorna todos los registros de 2 tablas.

```sql
SELECT columnas
FROM tabla1
CROSS JOIN tabla2;
```

### MySQL ``SELF JOIN``

Es un join regular pero lo particular es que lo hace con la misma tabla.

```sql
SELECT columnas
FROM tabla1 T1, tabla2 T2
WHERE condition;
```

### MySQL ``UNION`` Operator

Se usa para combinar el resultado de 2 o más sentencias ``SELECT``.

```sql
SELECT columnas FROM tabla1
UNION
SELECT columnas FROM tabla2;
```

### MySQL ``GROUP BY``
(Misma teoría de [[DQL (Data Query Language)#4. `GROUP BY` (_Agrupación de Filas_)|GROUP BY]] en SQL Server)

Agrupa los registros que tienen el mismo valor dentro de registros sumarizados.

Se utiliza con [[Aggregate Functions|funciones de agregación]] como son: ``COUNT()``, ``MAX()``, ``MIN()``, ``SUM()``, ``AVG()``.

```sql
SELECT columnas
FROM tabla
WHERE condición
GROUP BY columnas
ORDER BY columnas;
```

### MySQL ``HAVING``
(Misma teoría de [[DQL (Data Query Language)#5. `HAVING` (_Filtrado de Grupos_)|HAVING]] en SQL Server)

Se usa en sustitución de ``WHERE`` para condicionar la sentencia ``SELECT`` que llevan ``GROUP BY`` con funciones de agregación.

```sql
SELECT columnas
FROM tabla
WHERE condición
GROUP BY columnas
HAVING condición (con funciones de agregación)
ORDER BY columnas;
```

### MySQL ``EXISTS``

Se usa para comprobar si existe un registro en un sub-query.

```sql
SELECT columnas
FROM tabla
WHERE EXISTS
(SELECT columna FROM tabla WHERE condición);
```

### MySQL ``ANY`` & ``ALL``

**ANY:** devuelve un resultado verdadero si alguno de los valores del subquery cumple con la condición establecida.

```sql
SELECT columnas
FROM tabla
WHERE columna operador ANY
  (SELECT columna
  FROM tabla
  WHERE condición);
```

**ALL:** devuelve un resultado verdadero si todos los valores del subquery cumplen con la condición establecida.

```sql
FROM tabla
WHERE columna operador ALL
  (SELECT columna
  FROM tabla
  WHERE condición);
```

### MySQL ``CASE``
(Similar a las [[Advanced Functions#4. Conditional Functions (_Funciones Condicionales_)|funciones condicionales]] de SQL Server)

Se utiliza para establecer distintas condiciones en una consulta, cuando encuentra la primera que se cumpla devuelve el resultado.

```sql
CASE
    WHEN condición1 THEN resultado1
    WHEN condición2 THEN resultado2
    WHEN condiciónN THEN resultadoN
    ELSE resultado
END;
```

## Manipulación de Datos con MySQL
(Misma teoría de [[DML (Data Manipulation Language)]])

### MySQL ``INSERT INTO``

Es utilizada para insertar nuevos registros en una tabla.

```sql
INSERT INTO tabla (columna1, columna2, columna3, ...)
VALUES (valor1, valor2, valor3, ...);
```

### MySQL ``UPDATE``

Es utilizada para modificar registros existentes en una tabla.

```sql
UPDATE tabla
SET columna1 = valor1, columna2 = valor2, ...
WHERE condición;
```

### MySQL ``DELETE``

Es utilizada para eliminar registros existentes en una tabla.

```sql
DELETE FROM tabla WHERE condición;
```