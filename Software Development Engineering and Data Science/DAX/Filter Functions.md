# Funciones de Filtro

En **DAX**, las funciones de filtro permiten manipular y reducir los datos que se utilizan en los cálculos, ya sea para crear medidas, analizar subconjuntos de datos o manipular el contexto de los cálculos. Las funciones más comunes son `FILTER` y `CALCULATE`, que se utilizan para aplicar condiciones de filtrado a las tablas y expresiones.

## Funciones Principales

- `FILTER(<table>, <filter>)`: Devuelve una tabla que es un subconjunto de otra tabla o expresión.
- `CALCULATE(<expression>[, <filter1> [, <filter2> [, …]]])`: Evalúa una expresión en un contexto de filtro.
- `HASONEVALUE(<columnName>)`: Devuelve el valor TRUE cuando el contexto de columnName se ha filtrado a un solo valor distinto. De lo contrario, es FALSE.
- `ALLNOBLANKROW(<table> | <column>[, <column>[, <column>[,…]]])`: Devuelve una tabla que es un subconjunto de otra tabla o expresión.
- `ALL([<table> | <column>[, <column>[, <column>[,…]]]])`: Devuelve todas las filas de una tabla o todos los valores de una columna, ignorando cualquier filtro que se haya podido aplicar.
- `ALLEXCEPT(<table>, <column>[, <column>[,..]])`: Devuelve todas las filas de una tabla excepto aquellas filas que se ven afectadas por los filtros de columna especificados.
- `REMOVEFILTERS([<table> | <column>][, <column>[, <column>[,…]]]])`: Borrar todos los filtros de las tablas o columnas designadas.

## Consideraciones Importantes

- **Contexto de filtro:** Las funciones de filtro operan en el contexto de filtro, que es el conjunto de filtros aplicados a los datos antes de la evaluación de la expresión DAX.
- **Eficiencia:** En muchos casos, es más eficiente usar expresiones booleanas en lugar de la función `FILTER` dentro de `CALCULATE` para aplicar filtros.
- **Restricciones:** Las expresiones booleanas utilizadas como filtros en `CALCULATE` tienen algunas restricciones, como no poder hacer referencia a medidas o usar funciones anidadas `CALCULATE`.
