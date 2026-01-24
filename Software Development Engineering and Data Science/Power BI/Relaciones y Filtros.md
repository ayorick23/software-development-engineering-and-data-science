# Relaciones y Filtros

El entendimiento de las relaciones y filtros en Power BI es esencial para quien busca optimizar modelos de datos robustos y precisos. Aunque a menudo se ignora, este aspecto puede decidir el éxito de un análisis de datos. Al familiarizarnos con conceptos como las llaves primarias y foráneas y al comprender la importancia de las relaciones entre tablas, estamos un paso más cerca de desarrollar modelos de datos eficientes.

## ¿Qué son las llaves en un modelo de datos?

Las llaves en las tablas son elementos esenciales que definen la relación y unicidad de datos en un modelo. Primero, encontramos la [[Constraints#`PRIMARY KEY` - Clave Primaria|llave primaria]], única y no repetitiva, representativa de las tablas de dimensión. Por ejemplo, en una tabla de países, la llave primaria sería única y no debería tener valores nulos, a menos que se trate de una tabla aún en construcción. Luego, tenemos la [[Constraints#`FOREIGN KEY` - Clave Foránea|llave foránea]], que puede repetirse y se encuentra naturalmente en las tablas de hechos. En un escenario de compras, el código de un producto puede repetirse tantas veces como compras registradas, mientras mantiene su unicidad en la dimensión del producto.

### ¿Cuáles son los tipos de relaciones más comunes?

Al analizar datos con Power BI, existen varios tipos de relaciones, siendo la relación de uno a muchos la más eficaz para el modelo estrella. Este tipo de relación conecta una llave primaria de una tabla de dimensión con una llave foránea de una tabla de hechos. Sin embargo, se debe evitar la relación de muchos a muchos, puesto que puede resultar en datos incorrectos o atípicos que distorsionan el análisis.

## ¿Cómo se modelan las relaciones en Power BI?

Power BI permite la creación de un modelo de datos optimizado mediante la técnica de modelo estrella, en la cual una tabla de hechos central está rodeada de diversas tablas de dimensión conectadas. Al crear estas relaciones, el procedimiento implica identificar campos en común entre tablas para establecer conexiones de uno a muchos. De este modo, el filtrado se realiza desde la tabla de dimensión hacia la tabla de hechos, asegurando datos relevantes y precisos en nuestras consultas.

### ¿Qué pasos seguir en la vista de modelo?

1. **Creación de dimensiones**: Iniciar con la duplicación de una tabla existente para crear la dimensión necesaria, eliminando columnas no requeridas y duplicados.
2. **Formato de datos**: Aplicar transformaciones de formato, como la conversión a mayúsculas, en los campos pertinentes.
3. **Relaciones**: En la vista de modelo, se deben arrastrar y soltar los campos entre las tablas para establecer las relaciones deseadas.

## ¿Cómo optimizar la integración de tablas en Power BI?

Al integrar múltiples tablas, puedes crear una vista personalizada que solo incluya tablas relevantes para un análisis específico. Generar una nueva vista implicaría seleccionar cuáles tablas y relaciones enfocarse, favoreciendo un enfoque más sencillo y directo al análisis de datos. Se aconseja generar todas las posibles relaciones de uno a muchos entre llaves primarias y foráneas para mantener un modelo factible, confiable y eficiente.

Al final del proceso, puedes administrar todas las conexiones en el administrador de relaciones, donde se podrán adecuar las referencias correspondientes, asegurando que tu modelo de datos en Power BI esté perfectamente estructurado para el análisis detallado. La dedicación a estos pasos te permitirá dominar el arte del modelamiento de datos, llevando tu capacidad de análisis al siguiente nivel.
