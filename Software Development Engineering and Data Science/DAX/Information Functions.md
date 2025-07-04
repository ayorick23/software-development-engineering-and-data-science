# Funciones de Información

En DAX, las funciones de información se utilizan para obtener información sobre los datos o el contexto en el que se evalúa una fórmula. Estas funciones no realizan cálculos directamente, sino que proporcionan detalles sobre el tipo de dato, errores, relaciones, o el contexto de la evaluación.

## Funciones Principales

- `COLUMNSTATISTICS()`: Devuelve estadísticas de cada columna de cada tabla. Esta función no tiene argumentos.
- `NAMEOF(<value>)`: Devuelve el nombre de la columna o medida de un valor.
- `ISBLANK(<value>) // ISERROR(<value>)`: Devuelve si el valor está en blanco // un error.
- `ISLOGICAL(<value>)`: Comprueba si un valor es lógico o no.
- `ISNUMBER(<value>)`: Comprueba si un valor es un número o no.
- `ISFILTERED(<table> | <column>)`: Devuelve verdadero cuando hay filtros directos en una columna.
- `ISCROSSFILTERED(<table> | <column>)`: Devuelve verdadero cuando hay filtros cruzados en una columna.
- `USERPRINCIPALNAME()`: Devuelve el nombre principal o la dirección de correo electrónico del usuario. Esta función no tiene argumentos.
