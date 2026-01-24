# Combinación de Tablas y Consultas con Power Query

## ¿Cómo funcionan las combinaciones en Power Query?

Entender las funcionalidades de [[Transformación de Datos con Power Query#¿Qué son Power Query y el proceso ETL?|Power Query]] es esencial para optimizar la integración y gestión de datos desde diferentes fuentes. Este potente recurso nos permite, entre otras cosas, unificar tablas o consultas, lo que resulta invaluable en numerosos escenarios de negocio. Examinemos cómo dichas combinaciones pueden ejecutarse eficazmente, mejorando nuestra capacidad de gestión de datos.

### ¿Qué es anexar en Power Query?

Uno de los métodos más comunes para combinar tablas en Power Query es `anexar`. Este proceso nos permite unir dos o más tablas en una sola. La condición ideal para este tipo de combinación es que todas las tablas tengan la misma estructura. En situaciones contrarias, las nuevas columnas serán rellenadas con valores nulos, similar al uso del comando [[Clase 03 - Sentencias SQL en MySQL#MySQL ``UNION`` Operator|UNION]] en [[Clase 01 - Definición de Base de Datos#Conceptos Claves|SQL]].

- **Ejemplo de uso**: Si tenemos tablas de ventas que cubren diferentes años, podemos anexar esas tablas para consolidar toda la información de ventas en un solo lugar. Esto se logra a través del editor de Power Query, seleccionando las tablas a anexar y validando la integración final mediante visualizaciones en Power BI.

### ¿Cómo se combina mediante un campo en común?

El proceso de combinación implica la unión de tablas o consultas basándose en un campo común, similar a un [[Joins#Joins|JOIN]] en SQL.

Existen varias formas de combinación:

- Externa izquierda
- Externa derecha
- Interna
- Anti izquierda
- Anti derecha
- Externa completa
- **Ejemplo práctico**: Supongamos que queremos unificar una tabla de materiales con otra de sucursales. Utilizando Power Query, podemos seleccionar un campo común, aunque los nombres de las columnas sean diferentes, siempre que contengan valores equivalentes. Esta combinación se puede ajustar mediante diferentes tipos, como la combinación externa izquierda si deseamos mantener todos los registros de la primera tabla.

### ¿Cómo se utiliza la combinación de binarios?

La **combinación de binarios** es ideal para automatizar la unión de archivos desde una misma carpeta. Supone una manera eficiente de centralizar y automatizar el proceso de integración sin la necesidad de cargar archivos uno por uno.

- **Ejemplo eficiente**: Supongamos que tenemos múltiples archivos de ventas de distintos años en una carpeta. Power Query permite combinarlos basándose en un parámetro común, algo esencial para gestionar grandes volúmenes de datos de forma rápida y eficiente.

### ¿Cuáles son las mejores prácticas al usar Power Query?

Implementar prácticas estratégicas durante el uso de Power Query garantiza una gestión de datos más ordenada y eficiente:

1. **Organización**: Clasificar y agrupar consultas relacionadas para facilitar la comprensión y la revisión de archivos.
2. **Uso de nombres descriptivos**: Asignar nombres significativos a tablas y consultas para un acceso rápido y claro.
3. **Orden y limpieza de datos**: Asegurarse de que todos los datos estén bien estructurados y sin errores antes de cualquier combinación.

Estas prácticas no solo facilitan el manejo de datos, sino que también optimizan el rendimiento global en aplicaciones como Power BI. Así, podemos garantizar un flujo de trabajo ágil, fomentar un análisis más efectivo, y capacitarse para usar herramientas potentes de forma eficiente.
