# Transformación de Datos con Power Query

## ¿Qué son Power Query y el proceso ETL?

Power Query es una herramienta de Microsoft diseñada para llevar a cabo el proceso de Extracción, Transformación y Carga ([[Business Intelligence#¿Cómo ayuda el BI en la toma de decisiones empresariales?|ETL]]) de datos. Se integra en plataformas populares como Excel y Power BI para proporcionar automatización de ETL a usuarios de negocio.

### ¿Cuál es el ciclo ETL?

- **Extracción:** Se refiere a la conexión a fuentes de datos, que pueden ser bases de datos locales o en la nube, archivos, etc.
- **Transformación:** Consiste en agregar y estandarizar datos, asegurando su calidad y enriquecimiento.
- **Carga:** Finalmente, los datos transformados se cargan en plataformas para análisis, como Data Warehouses o Data Lakes.

## ¿Cómo trabaja Power Query en Power BI?

Power Query es un componente esencial que habilita conexiones con múltiples tipos de datos. Algunos ejemplos son archivos Excel, bases de datos SQL, páginas web y servicios en línea. A través de diferentes métodos de conexión, facilita la importación y actualización de datos para maximizar los beneficios analíticos.

### ¿Cuáles son los métodos de conexión en Power Query?

1. **Importación:**
    - Realiza una copia exacta de los datos en Power BI.
    - Optimiza las consultas pero no permite una conexión en tiempo real.
2. **DirectQuery:**
    - Permite una conexión en tiempo real, pero solo funciona para bases de datos.
    - Puede ser impactante sobre el rendimiento del servidor dependiendo de las especificaciones de hardware.
3. **Live Connection (conexión dinámica):**
    - Se conecta a conjuntos de datos ya existentes en Power BI.
    - Es útil para uso en servicios de Power BI.
4. **Modelos Compuestos:**
    - Mezcla los beneficios de los métodos de importación y DirectQuery.
    - Ideal para tratar con tablas estáticas y dinámicas.

## ¿Cómo podemos aplicar estos métodos en Power BI Desktop?

Para ejemplificar el uso de los métodos de conexión, se pueden seguir los pasos adecuados en Power BI Desktop para aplicar tanto importación como DirectQuery.

### ¿Qué pasos seguir para la importación de datos?

1. **Acceder a Obtener Datos:** Seleccionar desde donde deseas importar (p. ej., archivos Excel).
2. **Conectar y Cargar:** Seleccionar el archivo y cargar sus hojas en Power Query para su posterior uso en Power BI.
3. **Visualización de Resultados:** Puedes verificar que los datos copiados se visualizan correctamente en el panel lateral derecho.

### ¿Cómo funciona DirectQuery para bases de datos SQL?

1. **Configuración Inicial:** Asegúrate de permitir la conexión a la base de datos SQL deseada, eliminando configuraciones anteriores si es necesario.
2. **Acceder a Obtener Datos:** Selecciona bases de datos bajo SQL Server y elige DirectQuery como método.
3. **Configuración de Seguridad:** Inserta las credenciales correctas, como nombre de usuario y contraseña, para establecer la conexión segura.
4. **Carga de Datos:** Selecciona las tablas deseadas y permite que los datos se conecten en tiempo real, sin copias locales en Power BI.

### ¿Cómo identificar la diferencia entre los modelos de conexión?

- Observa la vista del modelo: Las conexiones DirectQuery se diferencian por el color diferente en su cabecera. No verás la raya azul, característica de la importación.

Estos procesos permiten la integración efectiva de datos en Power BI, facilitando la capacidad de llevar a cabo análisis avanzados sin comprometer el rendimiento operativo ni la integridad de los datos.
