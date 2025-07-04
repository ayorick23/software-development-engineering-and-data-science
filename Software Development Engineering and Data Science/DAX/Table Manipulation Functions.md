# Funciones de Manipulación de Tablas

En DAX, las funciones de manipulación de tablas permiten crear nuevas tablas o modificar las existentes para realizar análisis más detallados. Estas funciones son esenciales para transformar datos y obtener información valiosa. Similares a algunas funciones [[DQL (Data Query Language)|DQL]] y [[Joins|JOIN's de SQL]].

## Funciones Principales

- `SUMMARIZE(<table>, <groupBy_columnName>[, <groupBy_columnName>]…[, <name>, <expression>]…)`: Devuelve una tabla de resumen de los totales solicitados para un conjunto de grupos.
- `DISTINCT(<table>)`: Devuelve una tabla eliminando filas duplicadas de otra tabla o expresión.
- `ADDCOLUMNS(<table>, <name>, <expression>[, <name>, <expression>]…)`: Agrega columnas calculadas a la tabla o expresión de tabla dada.
- `SELECTCOLUMNS(<table>, <name>, <expression>[, <name>, <expression>]…)`: Selecciona columnas calculadas de la tabla o expresión de tabla dada.
- `GROUPBY(<table> [, <groupBy_columnName>[, [<column_name>] [<expression>]]…)`: Cree un resumen de la tabla de entrada agrupada por columnas específicas.
- `INTERSECT(<left_table>, <right_table>)`: Devuelve las filas de la tabla del lado izquierdo que aparecen en la tabla del lado derecho.
- `NATURALINNERJOIN(<left_table>, <right_table>)`: Une dos tablas mediante una unión interna.
- `NATURALLEFTOUTERJOIN(<left_table>, <right_table>)`: Une dos tablas mediante una unión externa izquierda.
- `UNION(<table>, <table>[, <table> [,…]])`: Devuelve la unión de tablas con columnas coincidentes.
