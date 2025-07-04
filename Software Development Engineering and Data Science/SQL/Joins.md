# Joins

En SQL, un `JOIN` es una operación que combina filas de dos o más tablas en una base de datos, basándose en una columna relacionada entre ellas. Permite crear un único conjunto de resultados a partir de diferentes tablas, lo que facilita la recuperación de datos que están distribuidos en varias tablas.

## 1. `INNER JOIN` (Unión Interna)

El `INNER JOIN` devuelve solo las filas cuando hay coincidencias en ambas tablas. Es el tipo de `JOIN` más común y, si no especificas un tipo de `JOIN`, la mayoría de los sistemas lo interpretarán como un `INNER JOIN`.

```sql
SELECT columnas
FROM TablaA
INNER JOIN TablaB ON TablaA.columna_comun = TablaB.columna_comun;
```

## 2. `LEFT JOIN` (Unión Izquierda)

El `LEFT JOIN` (o `LEFT OUTER JOIN`) devuelve todas las filas de la tabla de la izquierda (la primera tabla mencionada), y las filas coincidentes de la tabla de la derecha. Si no hay una coincidencia en la tabla de la derecha, los valores para las columnas de la tabla de la derecha serán `NULL`.

```sql
SELECT columnas
FROM TablaA
LEFT JOIN TablaB ON TablaA.columna_comun = TablaB.columna_comun;
```

## 3. `RIGHT JOIN` (Unión Derecha)

El `RIGHT JOIN` (o `RIGHT OUTER JOIN`) devuelve todas las filas de la tabla de la derecha (la segunda tabla mencionada), y las filas coincidentes de la tabla de la izquierda. Si no hay una coincidencia en la tabla de la izquierda, los valores para las columnas de la tabla de la izquierda serán `NULL`.

```sql
SELECT columnas
FROM TablaA
RIGHT JOIN TablaB ON TablaA.columna_comun = TablaB.columna_comun;
```

## 4. `FULL OUTER JOIN` (Unión Externa Completa)

El `FULL OUTER JOIN` (o `OUTER JOIN`) devuelve todas las filas cuando hay una coincidencia en una de las tablas. Si no hay una coincidencia, se devuelven los valores NULL para las columnas de la tabla sin coincidencia. Combina los resultados de `LEFT JOIN` y `RIGHT JOIN`.

**NOTA:** No todos los sistemas de bases de datos soportan `FULL OUTER JOIN` (por ejemplo, MySQL no lo tiene directamente, pero se puede simular con `UNION` de `LEFT JOIN` y `RIGHT JOIN`). PostgreSQL, SQL Server y Oracle sí lo soportan.

```sql
SELECT columnas
FROM TablaA
FULL OUTER JOIN TablaB ON TablaA.columna_comun = TablaB.columna_comun;
```

## 5. `CROSS JOIN` (Producto Cartesiano)

El `CROSS JOIN` devuelve el producto cartesiano de las dos tablas, es decir, cada fila de la primera tabla se combina con cada fila de la segunda tabla. El número de filas en el resultado es el producto del número de filas de ambas tablas. No se utiliza una cláusula `ON` para este tipo de `JOIN`.

```sql
SELECT columnas
FROM TablaA
CROSS JOIN TablaB;
```

### IMPORTANTE

1. **Rendimiento:** Un uso ineficiente de los `JOIN`s, especialmente en tablas muy grandes, puede afectar seriamente el rendimiento de la consulta. Asegúrate de tener índices en las columnas utilizadas en la cláusula `ON`.
2. **Alias de Tablas:** Es una buena práctica usar alias de tablas (ej. `E` para `Empleados`, `D` para `Departamentos`) para hacer las consultas más cortas y legibles, especialmente cuando se unen varias tablas o cuando hay columnas con el mismo nombre en diferentes tablas.
3. **Columnas Ambigüas:** Si dos tablas unidas tienen columnas con el mismo nombre, debes calificarlas con el nombre de la tabla o su alias (ej., `Empleados.id` o `E.id`).
