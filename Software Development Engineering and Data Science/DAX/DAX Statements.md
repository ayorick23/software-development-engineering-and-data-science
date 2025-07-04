# Declaraciones DAX

En DAX, una declaración se refiere a una expresión que define o calcula un valor. Estas declaraciones pueden ser medidas, columnas calculadas, o incluso variables dentro de una fórmula más grande. DAX utiliza declaraciones para crear lógica personalizada en Power BI y otros modelos de datos de Microsoft.

## Tipos de Declaraciones en DAX

### Medidas

Son cálculos definidos por el usuario que se evalúan en el contexto de la segmentación de datos y filas de una tabla. Se utilizan para agregar, resumir o analizar datos.

### Columnas Calculadas

Son columnas agregadas a una tabla existente con valores calculados por una expresión DAX. La expresión se evalúa para cada fila de la tabla.

- `COLUMN(<table>[<column>] = <expression>)`: Almacena el resultado de una expresión como una columna en una tabla.
- `ORDER BY(<table>[<column>])`: Define el orden de una columna. Cada columna puede ordenarse en orden ascendente (`ASC`) o descendente (`DESC`).

### Variables

Se utilizan para almacenar resultados de expresiones intermedias dentro de una medida o columna calculada. Esto ayuda a mejorar la legibilidad y el rendimiento de las expresiones complejas.

- `VAR(<name> = <expression>)`: Almacena el resultado de una expresión como una variable con nombre. Para devolver la variable, utilice `RETURN` después de definirla.
