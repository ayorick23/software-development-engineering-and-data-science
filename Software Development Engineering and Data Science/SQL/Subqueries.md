# Subqueries

Las subconsultas (también conocidas como consultas anidadas o `INNER QUERIES`) son sentencias `SELECT` que están incrustadas dentro de otra sentencia SQL (que puede ser [[DML (Data Manipulation Language)#`SELECT`|SELECT]], [[DML (Data Manipulation Language)#`INSERT`|INSERT]], [[DML (Data Manipulation Language)#`UPDATE`|UPDATE]], [[DML (Data Manipulation Language)#`DELETE`|DELETE]], [[DQL (Data Query Language)#2. `FROM` (_Origen de los Datos_)|FROM]] o [[DQL (Data Query Language)#5. `HAVING` (_Filtrado de Grupos_)|HAVING]]). Una subconsulta se ejecuta primero y el resultado se utiliza como entrada para la consulta externa. Son una herramienta poderosa para resolver problemas complejos que no pueden ser resueltos con un solo `SELECT` directo o un `JOIN` simple.

## Tipos de Subconsultas

Las subconsultas se pueden clasificar de varias maneras, pero una distinción clave es si son **correlacionadas** o **no correlacionadas**.

### Subconsultas No Correlacionadas

Una **subconsulta no correlacionada** es independiente de la consulta externa. Se ejecuta una sola vez y su resultado se pasa a la consulta externa. Esto significa que la subconsulta no hace referencia a ninguna columna de la consulta externa.

1. **Subconsultas en la cláusula [[DQL (Data Query Language)#3. `WHERE` (_Filtrado de Filas_)|WHERE]] (_para filtrar filas_)**

   - Con operadores de comparación (`=`, `>`, `<`, `>=`, `<=`, `!=`, `<>`) cuando la subconsulta devuelve un solo valor.
   - Con operadores `IN`, `NOT IN` cuando la subconsulta devuelve una lista de valores.
   - Con operadores `ANY`, `ALL` (raramente usados, `IN` es más común para "cualquiera").
   - Con operadores `EXISTS`, `NOT EXISTS` (para verificar la existencia de filas).

```sql
SELECT columna1, columna2, ...
FROM tabla1
WHERE columna_comparacion OPERADOR (
    SELECT columna
    FROM tabla2
    WHERE condición
);
```

2. **Subconsultas en la cláusula [[DQL (Data Query Language)#2. `FROM` (_Origen de los Datos_)|FROM]] (_como tablas derivadas_)**

El resultado de la subconsulta se trata como una tabla temporal sobre la que puedes realizar más operaciones.

```sql
SELECT columnas
FROM (
    SELECT columnas_subconsulta
    FROM tabla_subconsulta
    WHERE condición_subconsulta
) AS alias_tabla_temporal
WHERE condición_principal;
```

3. **Subconsultas en la cláusula `SELECT` (_como columnas escalares_)**

La subconsulta debe devolver un solo valor (una sola fila y una sola columna) para cada fila de la consulta externa.

```sql
SELECT columna1, (SELECT otra_columna FROM tabla_interna WHERE condición) AS alias_subconsulta
FROM tabla_principal
WHERE condición_principal;
```

## Subconsultas Correlacionadas

Una **subconsulta correlacionada** depende de la consulta externa. Se ejecuta una vez por cada fila procesada por la consulta externa. La subconsulta hace referencia a una o más columnas de la consulta externa. Esto las hace más flexibles pero a menudo menos eficientes que las no correlacionadas para grandes volúmenes de datos.

1. **En la cláusula `WHERE` con `EXISTS` / `NOT EXISTS`**

Para verificar la existencia de filas relacionadas en otra tabla para cada fila de la consulta externa.

```sql
SELECT columna1
FROM tabla_principal
WHERE EXISTS (
    SELECT columna_subconsulta
    FROM tabla_subconsulta
    WHERE tabla_subconsulta.columna_subconsulta = tabla_principal.columna1
);
```

2. **Para encontrar filas donde una columna cumple una condición basada en un grupo relacionado**

```sql
SELECT columna1, columna2, ...
FROM tabla_principal
WHERE condicion_principal
AND columna_tabla_principal IN (
    SELECT columna_subconsulta
    FROM tabla_subconsulta
    WHERE condicion_subconsulta
);
```

## Cuándo Usar Subconsultas

- Cuando necesitas usar el resultado de una consulta como entrada para otra.
- Cuando los [[Joins]] no son suficientes o hacen la consulta demasiado compleja (aunque a menudo un `JOIN` es más eficiente).
- Para filtrar filas basándose en la existencia de datos en otra tabla (`EXISTS`/`NOT EXISTS`).
- Para realizar cálculos agregados basados en subconjuntos de datos específicos.
- Para obtener un valor escalar que se usará por cada fila de la consulta externa.

## Alternativas a las Subconsultas

A menudo, los `JOIN`s o las [[Common Table Expressions]] (_CTEs_) pueden ofrecer soluciones más legibles y, en algunos casos, más eficientes que las subconsultas, especialmente las correlacionadas. Es importante evaluar cuál es la mejor herramienta para cada problema.

Por ejemplo, la primera consulta de `EXISTS` (Obtener los clientes que han realizado al menos un pedido) podría reescribirse con un [[Joins#1. `INNER JOIN` (Unión Interna)|INNER JOIN]] de la siguiente manera:

```sql
-- Usando INNER JOIN como alternativa
SELECT DISTINCT c.cliente_id, c.nombre, c.apellido
FROM Clientes c
INNER JOIN Pedidos p ON c.cliente_id = p.cliente_id;
```
