# Lenguaje DAX
(ver [[DAX Statements]] para la sintaxis general de columnas, medidas y tablas calculadas)

[[Optimización de Modelos de Datos|← Anterior]] | [[Cálculo de Medidas y Contexto de Filtro|Siguiente →]]

## ¿Qué es DAX y por qué es relevante?

[[DAX/Aggregate Functions#Funciones de Agregación|DAX]], o Data Analysis Expression, es el lenguaje de expresiones analíticas en Power BI. Diseñado para usuarios de negocio, permite generar indicadores de gestión para responder preguntas empresariales. Se encuentra también en Microsoft Excel y SaaS Tabular. Su sintaxis es similar a las fórmulas de Excel, facilitando su uso para quienes ya dominan este programa. DAX es esencial para manipular modelos de datos tabulares, permitiendo la creación de fórmulas que mejoran el análisis y la toma de decisiones.

## ¿Cómo puedo utilizar DAX en Power BI?

DAX en Power BI sirve para crear tres tipos principales de fórmulas:

- **[[DAX Statements#Columnas Calculadas Nuevos Atributos en el Modelo|Columnas Calculadas]]:** Similares a las fórmulas de Excel, se calculan fila por fila y ocupan espacio en el modelo de datos.
- **Tablas calculadas:** Derivan de otras tablas y son útiles para integrar dimensiones como el tiempo.
- **[[DAX Statements#Medidas Cálculos Dinámicos|Medidas]]:** Son las más utilizadas gracias a su eficiencia y su capacidad para generar resultados óptimos sin ocupar espacio en disco, pues los cálculos se realizan en memoria.

### ¿Cuáles son las mejores prácticas al utilizar DAX?

Aquí algunos consejos para usar DAX de manera eficaz:

1. **Nomenclatura de tablas y campos:** Los nombres de tablas deben ir entre comillas simples y los campos entre corchetes.
2. **Relaciones y modelamiento:** Asegúrate de que las relaciones entre tablas estén correctamente establecidas para optimizar el modelo de datos.
3. **Formato de tiempo:** Mantén un control exhaustivo de las fechas, considerando siempre una dimensión de tiempo.

## ¿Cómo trabajar con DAX en Power BI Desktop?

Durante la parte práctica con Power BI Desktop, se realizó la creación de columnas y medidas. El ejemplo consistió en calcular el margen bruto y el margen comercial de ventas de vehículos. Se mostró cómo la creación de columnas y tablas calculadas se aplica para construir y optimizar modelos de datos.

### ¿Cómo se crean columnas y medidas?

Para crear columnas calculadas:

1. Ve a la vista de tabla.
2. Selecciona la tabla deseada.
3. Usa la opción "nueva columna" para insertar la fórmula DAX correspondiente, por ejemplo, para calcular el margen bruto.

Para crear medidas:

1. Crea una tabla vacía para almacenarlas.
2. Usa la opción "nueva medida" y formula la expresión DAX, como la suma total de ventas.

## ¿Qué ventajas ofrece el uso de medidas en comparación con columnas calculadas?

Las medidas son altamente ventajosas ya que:

- No ocupan espacio adicional en el disco.
- Realizan cálculos en memoria lo que acelera el proceso.
- Pueden reutilizarse, permitiendo construir encima de otras medidas previamente generadas.

Es fundamental en cualquier análisis de datos diseñado en Power BI priorizar el uso de medidas sobre columnas calculadas, especialmente cuando se trate de cálculos que no dependen directamente del contexto de cada fila.

## ¿Qué más podemos explorar en DAX?

El mundo de DAX es vasto y ofrece muchas oportunidades para profundizar en su estudio y aprovechar al máximo sus capacidades. Para los interesados en perfeccionar sus habilidades en este lenguaje, se recomienda considerar cursos más avanzados centrados en DAX para Power BI. Esto permitirá no solo afinar el uso de técnicas avanzadas sino también descubrir nuevas posibilidades analíticas dentro del ecosistema de Power BI.
