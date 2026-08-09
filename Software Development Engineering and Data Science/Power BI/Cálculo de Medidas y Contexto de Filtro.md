# Cálculo de Medidas y Contexto de Filtro

[[Lenguaje DAX|← Anterior]] | [[Funciones Iterativas|Siguiente →]]

## ¿Cómo utilizar DAX para crear fórmulas en Power BI?

Crear fórmulas en Power BI a través de [[Lenguaje DAX#¿Qué es DAX y por qué es relevante?|DAX (Data Analysis Expressions)]] puede ser un proceso transformador para los analistas de datos. DAX ofrece la oportunidad de optimizar cálculos y realizar análisis complejos con rapidez y precisión. La fórmula más empleada es [[Filter Functions#CALCULATE El motor de DAX|CALCULATE]], que permite agregar contextos de filtro a diversas operaciones.

### ¿Qué es ``CALCULATE`` y cómo se utiliza?

``CALCULATE`` es fundamental en la metodología DAX, ya que ofrece la capacidad de evaluar operaciones como la suma, añadiendo contextos de filtro según las necesidades del usuario. Por ejemplo, para crear un indicador que muestre solo las ventas de una marca específica, se puede implementar una medida usando ``CALCUALTE``. Este proceso consiste en:

- Crear una nueva medida en la tabla.
- Definir la expresión a evaluar, como el total de ventas.
- Establecer un contexto de filtro, por ejemplo, filtrando por la marca del vehículo ("Toyota").

Esta operación no es case sensitive, y se puede arrastrar el resultado para visualizar rápidamente el total de ventas específico para la marca.

### ¿Cómo calcular el porcentaje de participación?

Para determinar la participación de cada marca en las ventas totales, es necesario considerar tanto el total de ventas por marca como el total de ventas general. Dado que DAX no opera con celdas individuales como Excel, se deben emplear diferentes agregaciones para obtener el resultado deseado.

1. **Total de ventas por marca**: Puedes utilizar un ``CALCULATE`` incorporando [[Filter Functions#ALL()|ALL]] para eliminar cualquier interacción en la tabla respecto a la marca. Esto hará que se muestren los totales generales en cada ítem de la tabla.
2. **Calcular la participación**: Una vez obtenidos ambos totales, implementa una división ([[DAX/Aggregate Functions#DIVIDE()|DIVIDE]]) para calcular la participación relativa de las ventas de una marca sobre el total general.

Una vez realizadas estas operaciones, se puede dar formato a las columnas de datos para garantizar que se muestren correctamente, por ejemplo, en porcentajes o monedas, ajustando también el número de decimales según sea necesario.

### ¿Cómo crear una tarjeta de visualización de participación?

Para visualizar la participación de una marca en un dashboard, se puede crear una tarjeta que indique directamente el porcentaje específico de esa marca. La estrategia a seguir es la siguiente:

- Define una nueva medida utilizando ``CALCULATE`` para evaluar la participación de la marca, filtrando nuevamente por la marca específica ("Toyota").
- Asegúrate de incluir la expresión correcta y verifica que los paréntesis y las comas estén bien colocados.
- Arrastra la medida a la tarjeta de participación y aplica el formato de porcentaje.

De esta forma, puedes ver, por ejemplo, qué proporción del total de ventas corresponde a Toyota, ilustrada con un porcentaje claro y preciso. Así, DAX en Power BI no solo facilita cálculos complejos, sino que también permite visualizaciones claras y contextuales en tiempo real.
