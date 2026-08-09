# Automatización y Limpieza de Datos

[[Arquitecturas de Power BI|← Anterior]] | [[Transformación de Datos con Power Query|Siguiente →]]

## ¿Qué es Power Query y por qué se le considera una "magia"?

Power Query es una herramienta esencial dentro de Power BI, encargada del proceso de [[Transformación de Datos con Power Query#¿Cuál es el ciclo ETL?|ETL (extracción, transformación y carga)]] de datos. No obstante, es importante comprender que Power Query no es una herramienta para el análisis directo de datos, sino para crear procesos de automatización de calidad, homologación y más. Una de sus características más destacadas es la capacidad de retroceder, avanzar o eliminar un paso dentro del proceso, similar a desarrollar macros pero en un entorno de código bajo.

### ¿Cómo se accede y se navega en Power Query?

Para usar Power Query, en Power BI buscamos la opción "transformar datos" en la cinta de opciones. Una vez dentro del editor de consultas, encontramos varias secciones: la cinta de opciones con pestañas como inicio, transformar, agregar columna, entre otras; la sección de consultas a la izquierda, donde se cargan todas las tablas; y la barra de fórmulas con el [[Automatización de Datos con Lenguaje M#¿Qué es el lenguaje M en Power BI y cómo funciona?|lenguaje M]], vital para la programación en Power Query.

### ¿Cuál es la funcionalidad de la estructura de datos dentro de Power Query?

Power Query permite cargar y transformar tablas desde fuentes diversas. Por ejemplo, al trabajar con una tabla de matriz, podemos transformarla en una estructura tabular más común dentro de Power BI. Esto implica eliminar columnas innecesarias, usar la primera fila como encabezado y renombrar columnas. Esto no solo optimiza la visualización de los datos sino también su análisis en Power BI.

## ¿Cómo realizar transformaciones básicas en Power Query?

Realizar transformaciones en Power Query es como maniobrar la "magia", permitiendo revertir cambios y editar pasos aplicados. Para ilustrar esto, considera un archivo Excel sobre gastos por departamento. Luego de cargar los datos, podemos:

1. Quitar columnas innecesarias.
2. Promover la primera fila como encabezado.
3. Transformar una matriz en una estructura tabular, útil para generar análisis y visualizaciones más precisas.

### ¿Cómo gustan los valores y formatos de datos en Power Query?

El manejo y adaptación de valores en Power Query es fundamental. Por ejemplo, al trabajar con valores monetarios, se reemplazan símbolos no reconocidos, como el del dólar, para evitar errores. Además, los tipos de datos deben ser adecuados, como un número decimal o entero, según corresponda, asegurando así un análisis correcto.

### ¿Cómo se manejan errores y tipos de datos regionales?

Al trabajar con datos de diversas regiones, como un archivo desde Estados Unidos, podemos enfrentar configuraciones no locales. Power Query permite ajustar formatos de fecha y números según la configuración regional adecuada, por ejemplo, en inglés de EE. UU. También es crucial eliminar filas problemáticas para evitar errores, usando herramientas como ordenar columnas y agregar índices.

## ¿Cuáles son las ventajas de una estructura tabular en Power BI?

Trabajar con datos en formato tabular en lugar de matrices ofrece significativas ventajas en Power BI:

1. **Flexibilidad de Filtrado:** Permite filtrar por períodos o categorías fácilmente.
2. **Análisis Coherente:** Evita duplicidades como totalizaciones erróneas, optimizando los resultados.
3. **Visualización Dinámica:** Favorece la generación de gráficos y reportes más claros y precisos sin datos replicados innecesariamente.

Power Query es una herramienta poderosa que transforma cómo interactuamos con los datos dentro de Power BI. Al entender y aplicar sus características, optimizamos procesos y creamos análisis mucho más potentes.
