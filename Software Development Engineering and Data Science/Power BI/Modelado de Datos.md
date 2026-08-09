# Modelado de Datos

[[Automatización de Datos con Lenguaje M|← Anterior]] | [[Relaciones y Filtros|Siguiente →]]

## ¿Qué es el modelamiento de datos y por qué es importante en Business Intelligence?

El modelamiento de datos es la clave para construir soluciones efectivas en Business Intelligence (BI) y analítica, permitiéndonos realizar análisis complejos entre múltiples tablas interconectadas mediante campos comunes. Dentro de Power BI, esto se logra a través del esquema de modelo estrella, compuesto por una tabla de hechos central y varias tablas de dimensiones. ¿Por qué es vital este enfoque? Porque optimiza tanto el rendimiento como el tamaño del modelo de datos, eliminando redundancias y acelerando la toma de decisiones empresariales.

### ¿Cómo se construye un modelo de datos en Power BI?

1. **Identificación de componentes clave**: Inicie con la identificación de las tablas de hechos y dimensiones. La tabla de hechos debe contener la mayoría de los datos transaccionales, mientras que las tablas de dimensiones ayudan a optimizar y categorizar la información.
2. **Uso del componente VertiPack**: VertiPack es una herramienta crucial dentro de Power BI para el modelamiento de datos, que facilita la compresión y el rendimiento del modelo, y es fundamental para crear funciones DAX para generar indicadores de gestión.
3. **Optimización y reducción de redundancia**: Revisar los archivos de datos, identificar campos repetidos y optimizarlos mediante el uso de tablas de dimensiones que evitan la duplicación innecesaria de información.
4. **Conexión con Power Query**: Realice transformaciones y limpiezas en Power Query antes de importar los datos al entorno de Power BI. Esto puede incluir la eliminación de columnas vacías y el renombramiento de tablas y columnas para una mejor organización.

### ¿Cómo se relacionan las tablas para crear un modelo de datos eficaz?

Es esencial establecer relaciones robustas entre las tablas. Estas relaciones permiten responder a las preguntas del negocio a diferentes niveles de granularidad: diario, mensual, trimestral, etc., lo que facilita la creación de reportes visuales y narrativas de datos eficaces.

1. **Creación de relaciones**: Asocia las tablas mediante campos comunes para establecer relaciones, permitiéndole al modelo responder a preguntas complejas del negocio.
2. **Homologación de datos**: Mediante Power Query, es posible normalizar los datos para asegurar que cualquier variación en la forma (como el uso de mayúsculas) no afecte al identificar registros duplicados.
3. **Enriquecimiento de datos**: Incluya más allá de reducir, ya que gracias a procesos de agregación de columnas, o cálculos condicionales, se pueden añadir descripciones que enriquezcan la información analizada.

En resumen, crear un modelo de datos efectivo en Power BI implica más que solo importar datos: es un proceso estratégico que requiere análisis, limpieza y organización detallada de la información. Estos pasos no solo optimizan el rendimiento del sistema, sino que también aseguran que las preguntas empresariales sean respondidas con eficiencia.
