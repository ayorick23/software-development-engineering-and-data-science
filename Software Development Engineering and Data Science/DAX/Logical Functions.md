# Funciones Lógicas

En **DAX**, las funciones lógicas se utilizan para evaluar condiciones y devolver valores booleanos (VERDADERO o FALSO), permitiendo la creación de expresiones condicionales complejas para análisis de datos. Estas funciones son fundamentales para crear columnas calculadas y medidas que dependen de la lógica condicional, ayudando a filtrar datos, crear cálculos personalizados y ejecutar visualizaciones basadas en condiciones específicas. Similar a las [[Advanced Functions#4. Conditional Functions (_Funciones Condicionales_)|funciones condicionales de SQL]].

- `IF(<logical_test>, <value_if_true>[, <value_if_false>])`: Comprueba una condición y devuelve un valor determinado dependiendo de si es verdadera o falsa.
- `AND(<logical 1>, <logical 2>)`: Comprueba si ambos argumentos son TRUE, y devuelve TRUE si ambos argumentos son TRUE. En caso contrario, devuelve FALSE.
- `OR(<logical 1>, <logical 2>)`: Comprueba si uno de los argumentos debe devolver TRUE. La función devuelve FALSE si ambos argumentos son FALSE.
- `NOT(<logical>)`: Cambios TRUE a FALSE y viceversa.
- `SWITCH(<expression>, <value>, <result>[, <value>, <result>]…[, <else>])`: Evalúa una expresión contra una lista de valores y devuelve uno de los posibles resultados.
- `IFERROR(<value>, <value_if_error>)`: Devuelve value_if_error si la primera expresión es un error y el valor de la expresión en sí en caso contrario.
