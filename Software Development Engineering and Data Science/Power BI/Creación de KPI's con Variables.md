# Creación de KPI's con Variables

## ¿Cómo crear KPIs en Power BI usando variables?

Los Key Performance Indicators (KPIs) son vitales en cualquier solución de analítica de datos, sirviendo como métricas estratégicas para seguir el rendimiento de una organización. Power BI, popular por su capacidad de transformar datos en informes visuales fáciles de interpretar, permite la creación de KPIs mediante el uso de variables. Veamos cómo lograrlo y optimizar la creación de medidas con [[Lenguaje DAX#¿Qué es DAX y por qué es relevante?|DAX]].

### ¿Cómo reutilizar un modelo de datos comercial en Power BI?

Para comenzar, es fundamental reutilizar el modelo de datos comercial existente. Primero, dirígete a la vista de tabla y analiza la tabla de presupuestos. Observa el presupuesto por sede y a nivel de mes y año. Una relación importante es la conexión entre las tablas de presupuesto y otras tablas relevantes, como la de ventas.

1. **Ajustar tablas**: Revisa la relación entre la tabla de presupuestos y las otras tablas para identificar cualquier falta en la dimensión tiempo o calendario.
2. **Transformar datos**: Corrige cualquier error separando mes y período.
3. **Combinar columnas**: Selecciona y combina las columnas correspondientes para crear la “fecha presupuesto”.
4. **Configurar el tipo de dato**: Asegúrate de que el tipo de dato sea 'fecha' para evitar errores. Por ejemplo, el caso de septiembre que no fue reconocido debido a la configuración regional del computador.

### ¿Cómo conectar tablas utilizando la dimensión calendario?

La conexión entre la tabla de ventas y la tabla de presupuestos mediante la fecha es esencial para una correcta correlación de datos a través del tiempo.

- **Agregar dimensión calendario**: Conecta estas tablas a través de la fecha con la lógica: "fecha" en la tabla de ventas con "fecha presupuesto" en la tabla de presupuestos.
- **Verificar granularidad**: Confirma que la información ahora conversa con un nivel de granularidad mensualmente mínimo para evitar cualquier discrepancia temporal.

## ¿Cómo generar KPIs y utilizar variables en Power BI?

Una vez establecidas las conexiones, podemos comenzar a crear los KPIs y utilizar variables para facilitar estas métricas complejas.

### ¿Qué es una variable en Power BI y cómo se usan en DAX?

Las variables son fórmulas dentro de fórmulas que permiten optimizar la creación de medidas con DAX.

- **Ejemplo de medida KPI**: Puedes empezar generando una medida para el presupuesto como una suma de las columnas pertinentes.
- **Crear métricas porcentuales**: Utiliza la función `DIVIDE` para dividir el total de ventas entre el presupuesto (o target).

### ¿Cómo implementar Quarter over Quarter y Year over Year?

Para el análisis temporal, herramientas como Quarter over Quarter (QoQ) y Year over Year (YoY) son indispensables.

1. **Quarter over Quarter (QoQ)**:
    
    - Define una variable con la sintaxis `var`.
    - Utiliza las funciones `CALCULATE` y `DATEADD` para aislar el último trimestre.
    - Realiza una división para determinar la variación trimestral.
2. **Year over Year (YoY)**:
    
    - Similar a QoQ, pero ajustando la resta a períodos anuales.
    - Implementa `SAMEPERIODLASTYEAR` para lograr esta comparación.

### ¿Qué hacer con los resultados en blanco en YoY?

Los datos en blanco pueden ser confusos para los gerentes. Es posible utilizar una alternativa:

- **Variable alternativa**: Emplea la función `COALESCE` para reemplazar valores nulos por cero, mejorando la presentación de los resultados.

## Conclusión

Ahora tienes las habilidades para integrar y optimizar indicadores de gestión en tu modelo de datos de Power BI eficazmente. Anímate a seguir explorando y perfeccionando tus habilidades con Power BI, y prepárate para el siguiente módulo donde profundizaremos en la utilización de estos indicadores mediante objetos visuales y aplicando buenas prácticas de narración de datos. ¡La práctica constante potenciará tu competencia analítica!
