# Funciones Iterativas

[[Cálculo de Medidas y Contexto de Filtro|← Anterior]] | [[Inteligencia de Tiempo|Siguiente →]]

## ¿Cómo funcionan las funciones iterativas en Power BI?

Las [[DAX/Aggregate Functions#Funciones de Iteración (Las que terminan en `X`)|funciones iterativas]], también conocidas como funciones X, son esenciales para manejar cálculos avanzados en Power BI. Estas funciones permiten iterar fila por fila dentro de una tabla, lo que es útil para generar cálculos complejos que no se pueden lograr con métodos tradicionales de Excel, como la simple referencia a filas o rangos de celdas.

### ¿Cuáles son las funciones X más comunes y para qué se utilizan?

Las funciones iterativas más frecuentes incluyen:

- **[[DAX/Aggregate Functions#SUMX()|SUMX]]**: Utilizada para sumas iterativas.
- **[[DAX/Aggregate Functions#AVERAGEX()|AVERAGEX]]**: Calcula promedios iterativos.
- **MINX**: Encuentra valores mínimos iterativos.

Estas funciones son ideales para aplicaciones como la creación de un diagrama de Pareto.

## ¿Cómo se desarrolla un diagrama de Pareto con funciones X en Power BI?

Para crear un diagrama de Pareto, se deben seguir varios pasos que involucran el uso de funciones iterativas en Power BI:

### ¿Cuáles son los pasos para crear un ranking de marcas?

1. **Creación de la tabla**: Se inicia con una tabla que incluye las ventas totales y la variable "marca de vehículo".
2. **Generación del ranking**: Para crear un RANKX que itere sobre cada fila, se utiliza la función [[DAX/Aggregate Functions#RANKX()|RANKX]] acompañada de [[Filter Functions#ALL()|ALL]] para evitar interacción con el campo "marca".

El objetivo es clasificar por ventas cada marca, proporcionando un ranking desde Toyota en la primera posición en adelante.

### ¿Cómo se realiza la suma iterativa en Power BI?

La suma iterativa se logra con los siguientes pasos:

1. **Nueva medida para el valor de marca**: Se crea una medida utilizando la función `SUMX` para sumar fila por fila en orden descendente tomando el ranking de ventas.
2. **Función TOP N**: Se usa para restringir la suma al top de marcas por ventas, ordenando desde la más alta.

Con esto, se obtiene una suma acumulativa de ventas, alcanzando el total de ingresos deseados.

### ¿Cómo se implementa la división y el formato del diagrama de Pareto?

1. **Creación de la medida "Pareto marca"**: Esta medida se genera mediante una división (`DIVIDE`) del valor de marca respecto al total de ventas por marca, permitiendo incluir un resultado alternativo en caso de división por cero.
2. **Configuración del formato**: Se aplica un formato de porcentaje al resultado para plasmar claramente el valor de Pareto.

Finalmente, se guarda el modelo de datos después de cerrar cualquier bug del sistema. Un gráfico de columnas agrupadas y líneas permite visualizar claramente el Pareto, revelando que aproximadamente el 80% de las ventas provienen de un pequeño porcentaje de las marcas.

## Recomendaciones y consejos para el uso de funciones iterativas

- **Practica constantemente**: La práctica con estas funciones es esencial para dominar su uso y optimizar tus modelos de Power BI.
- **Pruébalas en diversos contextos**: No te limites a un solo tipo de análisis; explora cómo las funciones iterativas pueden aplicarse en otras áreas de tus datos.
- **Mantén tu modelo bien documentado**: Esto facilitará futuras modificaciones y el entendimiento del proceso de creación a terceros.
