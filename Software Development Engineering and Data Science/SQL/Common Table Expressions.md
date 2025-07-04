# Common Table Expressions (CTEs)

Las **Common Table Expressions** (_CTEs_), también conocidas como expresiones de tabla comunes, son una característica de SQL que te permite definir un conjunto de resultados temporal y con nombre que puedes referenciar dentro de una única sentencia [[DML (Data Manipulation Language)#`SELECT`|SELECT]], [[DML (Data Manipulation Language)#`INSERT`|INSERT]], [[DML (Data Manipulation Language)#`UPDATE`|UPDATE]], [[DML (Data Manipulation Language)#`DELETE`|DELETE]] o [[Views#Creación de una Vista (`CREATE VIEW`)|CREATE VIEW]]. Piensa en una CTE como una "vista temporal" que solo existe durante la ejecución de la consulta.

Las CTEs no almacenan datos de forma persistente, son puramente lógicas. Son increíblemente útiles para:

1. **Mejorar la legibilidad de consultas complejas:** Permiten dividir una consulta compleja en bloques lógicos más pequeños y manejables, lo que facilita la lectura y el mantenimiento.
2. **Simplificar consultas anidadas (subconsultas):** A menudo pueden reemplazar subconsultas anidadas, haciendo el código más lineal y comprensible.
3. **Permitir el uso de la recursividad:** Son la base para escribir consultas recursivas en SQL.
4. **Reutilizar un mismo conjunto de resultados:** Puedes definir una CTE una vez y referenciarla múltiples veces dentro de la misma consulta principal.

## CTE Simple

Una CTE se define usando la cláusula `WITH`.

```sql
WITH nombre_cte AS (
    -- Tu consulta SELECT que define la CTE
    SELECT columna1, columna2
    FROM tabla
    WHERE condicion
)
-- La consulta principal que usa la CTE
SELECT *
FROM nombre_cte
WHERE otra_condicion;
```

## Múltiples CTEs en una Sola Consulta

Puedes definir varias CTEs en la misma cláusula `WITH`, separándolas con comas. Una CTE puede referenciar CTEs definidas anteriormente en la misma cláusula.

```sql
WITH nombre_cte1 AS (
    -- Tu consulta SELECT que define los CTEs
    SELECT columna1, columna2
    FROM tabla
    WHERE condicion
),
nombre_cte2 AS (
    SELECT columna1, columna2
    FROM tabla
    WHERE condicion
)
```

## CTEs con Funciones de Ventana (Window Functions)

Las CTEs son muy potentes cuando se combinan con [[Window Functions|funciones de ventana]], ya que permiten calcular métricas complejas en un primer paso y luego filtrar o agregar sobre esos resultados en un segundo paso más legible.

```sql
WITH nombre_cte1 AS (
    -- Tu consulta SELECT con función de ventana
    SELECT columna1,
    columna2,
    ROW_NUMBER() OVER (PARTITION BY columna1 ORDER BY columna2 DESC)
    FROM tabla
    WHERE condicion
)
```

## CTEs Recursivas (Recursive CTEs)

Una CTE recursiva es una CTE que se referencia a sí misma. Son fundamentales para procesar datos jerárquicos o de gráficos (como organigramas, estructuras de carpetas, listas de materiales, etc.). Una CTE recursiva tiene dos partes:

- **Miembro Ancla (Anchor Member):** La parte inicial de la CTE que establece la base de la recursión. Se ejecuta una sola vez.
- **Miembro Recursivo (Recursive Member):** La parte que se referencia a sí misma y se ejecuta repetidamente hasta que no se devuelven más filas. Debe contener una cláusula `UNION ALL`.

```sql
WITH RECURSIVE nombre_cte AS (
    -- Miembro Ancla (Consulta base)
    SELECT ...
    FROM ...
    WHERE ...

    UNION ALL

    -- Miembro Recursivo (Se refiere a nombre_cte)
    SELECT ...
    FROM ... JOIN nombre_cte ON ...
    WHERE ...
)
SELECT * FROM nombre_cte;
```

## Cuándo Usar CTEs

- **Refactorización de consultas complejas:** Cuando una consulta se vuelve demasiado larga o tiene muchas subconsultas anidadas.
- **Reutilización de lógica:** Si un subconjunto de datos o un cálculo intermedio se usa varias veces en la misma consulta.
- **Consultas recursivas:** Para manejar estructuras jerárquicas o gráficos.
- **Claridad del código:** Para hacer que tus sentencias SQL sean más legibles y fáciles de mantener, rompiendo la lógica en pasos pequeños y con nombre.
- **Debuggeo:** Puedes ejecutar cada CTE de forma independiente para verificar sus resultados antes de ejecutar la consulta completa.
