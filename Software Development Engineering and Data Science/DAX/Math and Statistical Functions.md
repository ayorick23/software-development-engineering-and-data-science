# Funciones matemáticas y estadísticas

En **DAX**, las funciones matemáticas y estadísticas permiten realizar cálculos y análisis sobre los datos en tus modelos. DAX ofrece una variedad de funciones para trabajar con números, realizar agregaciones y obtener información estadística, muy similares a las [[Aggregate Functions|funciones de agregación de SQL]].

## Funciones Principales

- `SUM(<column>)`: Suma todos los números en una columna.
- `SUMX(<table>, <expression>)`: Devuelve la suma de una expresión evaluada para cada fila de una tabla.
- `AVERAGE(<column>)`: Devuelve el promedio (media aritmética) de todos los números en una columna.
- `AVERAGEX(<table>, <expression>)`: Calcula el promedio (media aritmética) de un conjunto de expresiones evaluadas sobre una tabla.
- `MEDIAN(<column>)`: Devuelve la mediana de una columna.
- `MEDIANX(<table>, <expression>)`: Calcula la mediana de un conjunto de expresiones evaluadas sobre una tabla.
- `GEOMEAN(<column>)`: Calcula la media geométrica de una columna.
- `GEOMEANX(<table>, <expression>)`: Calcula la media geométrica de un conjunto de expresiones evaluadas sobre una tabla.
- `COUNT(<column>)`: Devuelve el número de celdas en una columna que contiene valores que no están en blanco.
- `COUNTX(<table>, <expression>)`: Cuenta la cantidad de filas de una expresión que se evalúa como un valor que no está en blanco.
- `DIVIDE(<numerator>, <denominator> [,<alternateresult>])`: Realiza la división y devuelve un resultado alternativo o BLANK() la división por 0.
- `MIN(<column>)`: Devuelve un valor mínimo de una columna.
- `MAX(<column>)`: Devuelve un valor máximo de una columna.
- `COUNTROWS([<table>])`: Cuenta el número de filas en una tabla.
- `DISTINCTCOUNT(<column>)`: Cuenta el número de valores distintos en una columna.
- `RANKX(<table>, <expression>[, <value>[, <order>[, <ties>]]])`: Devuelve la clasificación de un número en una lista de números para cada fila en el argumento de la tabla.
