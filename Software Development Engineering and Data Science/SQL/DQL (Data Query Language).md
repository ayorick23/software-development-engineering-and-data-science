# Data Query Language (DQL)

En **SQL**, DQL significa Data Query Language (_Lenguaje de Consulta de Datos_). Es la categoría de comandos SQL que se utiliza para recuperar datos de una base de datos. El comando principal dentro de DQL es `SELECT`.

La sentencia [[DML (Data Manipulation Language)#`SELECT`|SELECT]] es el corazón de las consultas SQL. Permite especificar qué columnas quieres ver, de qué tablas, bajo qué condiciones y cómo quieres que se agrupen u ordenen los resultados.

```sql
SELECT DISTINCT columna1, columna2, funcion_agregacion(columna) AS Alias
FROM nombre_tabla
[JOIN otra_tabla ON condicion_join]
WHERE condicion
GROUP BY columna_agrupacion1, columna_agrupacion2
HAVING condicion_grupo
ORDER BY columna_ordenamiento ASC/DESC
LIMIT numero_filas OFFSET inicio; -- o TOP / FETCH NEXT en otros DBMS
```

## Orden de los Comandos

### 1. `SELECT` (_Selección de Columnas_)

Especifica las columnas que deseas recuperar.

### 2. `FROM` (_Origen de los Datos_)

Indica la tabla (o tablas) de donde se obtendrán los datos.

### 3. `WHERE` (_Filtrado de Filas_)

Se utiliza para filtrar filas basándose en una o más condiciones. Solo las filas que cumplen la condición se incluirán en el resultado.

#### Operadores Comunes en `WHERE`

- Comparación: `=`, `!=` (o `<>`), `>`, `<`, `>=`, `<=`
- Lógicos: `AND`, `OR`, `NOT`
- Rango: `BETWEEN valor1 AND valor2`
- Listas: `IN (valor1, valor2, ...)`
- Patrones: `LIKE 'patrón'` (`%` para cero o más caracteres, `_` para un solo carácter)
- Valores Nulos: `IS NULL`, `IS NOT NULL`

### 4. `GROUP BY` (_Agrupación de Filas_)

Se usa para agrupar filas que tienen los mismos valores en una o más columnas en un conjunto de filas de resumen. Es crucial cuando se utilizan [[Aggregate Functions|funciones de agregación]] (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).

### 5. `HAVING` (_Filtrado de Grupos_)

`HAVING` es similar a `WHERE`, pero se aplica a los resultados de la agrupación (es decir, después de que `GROUP BY` haya combinado las filas). Se utiliza para filtrar grupos, no filas individuales. Siempre se usa después de `GROUP BY`.

### 6. `ORDER BY` (_Ordenación de Resultados_)

Se utiliza para ordenar el conjunto de resultados de una consulta en orden ascendente (`ASC`) o descendente (`DESC`) según una o más columnas. Si no se especifica, el orden por defecto es `ASC`.

### 7. `LIMIT` / `TOP` / `FETCH NEXT` (_Limitar Resultados_)

Estas cláusulas se usan para restringir el número de filas que se devuelven en el conjunto de resultados. La sintaxis varía según el sistema de gestión de bases de datos (DBMS).

- `LIMIT`: Usado comúnmente en MySQL, PostgreSQL, SQLite.
  - `LIMIT N`: Devuelve las primeras `N` filas.
  - `LIMIT N OFFSET M`: Devuelve `N` filas a partir de la fila `M`.
- `TOP`: Usado en SQL Server.
  - `SELECT TOP N ... FROM ...`
- `FETCH NEXT`: También en SQL Server (_desde SQL Server 2012_) y Oracle.
  - `OFFSET M ROWS FETCH NEXT N ROWS ONLY`
