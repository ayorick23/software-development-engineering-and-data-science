# Data Manipulation Language (DML)

En **SQL**, DML (_Data Manipulation Language o Lenguaje de Manipulación de Datos_) se refiere al conjunto de comandos que se utilizan para modificar los datos dentro de una base de datos. Estos comandos permiten **insertar**, **actualizar**, **eliminar** y **consultar** datos almacenados en tablas o vistas. En resumen, DML se centra en la manipulación de los datos en sí, a diferencia de **DDL** (_Data Definition Language_) [[DDL (Data Definition Language)]] que se enfoca en la definición de la estructura de la base de datos.

## `INSERT`

`INSERT`: Este comando se utiliza para insertar nuevos registros en una tabla. Se especifica la tabla y los valores que se van a insertar en cada columna.

```sql
INSERT INTO nombre_tabla (columna1, columna2, columna3, ...)
VALUES (valor1, valor2, valor3, ...);
```

## `SELECT`

`SELECT`: Este comando se utiliza para consultar datos de una o más tablas. Permite seleccionar columnas específicas, filtrar filas basadas en condiciones, ordenar resultados, etc.

```sql
SELECT columna1, columna2, ...
FROM nombre_tabla
WHERE condicion
ORDER BY columna ASC/DESC
LIMIT numero; -- o TOP, FETCH NEXT en otros DBMS
```

## `UPDATE`

`UPDATE`: Este comando se utiliza para modificar los datos existentes en una tabla. Se especifica la tabla, las columnas a modificar y las condiciones para determinar qué filas se actualizarán.

```sql
UPDATE nombre_tabla
SET columna1 = nuevo_valor1, columna2 = nuevo_valor2, ...
WHERE condicion;
```

## `DELETE`

`DELETE`: Este comando se utiliza para eliminar registros de una tabla. Se especifica la tabla y las condiciones para determinar qué filas se eliminarán.

```sql
DELETE FROM nombre_tabla
WHERE condicion;
```

**¡Cuidado!** Si omites la cláusula [[DQL (Data Query Language)##3. WHERE (Filtrado de Filas)|WHERE]], el `DELETE` eliminará todas las filas de la tabla, vaciándola por completo.
