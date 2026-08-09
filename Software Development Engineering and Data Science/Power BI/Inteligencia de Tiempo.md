# Inteligencia de Tiempo

[[Funciones Iterativas|← Anterior]] | [[Creación de KPI's con Variables|Siguiente →]]

## ¿Qué es la inteligencia de tiempo?

La inteligencia de tiempo es una técnica poderosa que implica el uso de estrategias para analizar y monitorear medidas en un periodo prolongado. En Power BI, una herramienta de Business Intelligence, el concepto de tiempo es esencial. A través de la inteligencia de tiempo, se puede evaluar la evolución de datos, supervisar crecimientos y prever tendencias futuras, todo de manera detallada.

### ¿Por qué es fundamental una tabla de fechas?

Para que la inteligencia de tiempo funcione correctamente en Power BI, es crucial tener una [[Date and Time Functions#Creación de Tablas de Fecha|tabla de fechas]]. Esta tabla, también conocida como tabla calendario, es una dimensión dentro de los modelos de datos. Su propósito principal es asegurar que no haya "saltos" en los periodos de tiempo, es decir, asegurar un período contiguo de fechas. Por ejemplo, si una empresa no trabaja fines de semana y no hay transacciones en esos días, esas fechas no podrían existir sin esta tabla, lo que afectaría el análisis temporal.

### ¿Cómo se integra la tabla de fechas en Power BI?

La integración de una tabla de fechas se realiza de forma obligatoria. Dentro de Power BI, incluso es posible crear una tabla calendario utilizando el lenguaje M:

1. **Obtener datos:** Se inicia añadiendo una consulta en blanco en Power Query.
2. **Añadir código M:** Se pega el código correspondiente para auto-generar una tabla calendario.
3. **Espacios temporales:** Especifica el periodo — como del 1 de enero de 2015 al 31 de diciembre de 2017.

Esta tabla creada se llena con campos descriptivos como días de la semana o meses, ofreciendo una base sólida para análisis más profundos y detallados.

### ¿Qué son las funciones DAX de inteligencia de tiempo?
(ver [[Time Intelligence Functions]])

Una vez se ha integrado la tabla de fechas, Power BI permite utilizar funciones DAX específicas de inteligencia de tiempo. Algunas de las más utilizadas son:

- **``PARALLELPERIOD``:** Para comparar periodos paralelos, por ejemplo, el mes anterior.
- **``DATEADD``:** Para agregar un intervalo de fechas a una fecha determinada.
- **``YEAR-TO-DATE``, ``MONTH-TO-DATE``, ``QUARTER-TO-DATE``:** Para acumular o resumir datos a lo largo del año, mes o trimestre.

Estas funciones permiten realizar cálculos complejos y proyecciones sobre los datos a lo largo del tiempo.

## ¿Cómo se configura una tabla de fechas en Power BI?

Una vez que la tabla de fechas está en el modelo, es necesario configurarla adecuadamente para su funcionalidad en Power BI:

1. **Configurar tabla de fechas:** Selecciona la tabla de fechas y márcala como tal en el modelo de datos para que Power BI la reconozca.
2. **Validación y guardado:** Verifica que todos los ajustes sean correctos.

### ¿Cómo se realizan agregaciones de tiempo inteligente?

Después de establecer la tabla de fechas, se pueden implementar agregaciones inteligentes:

- **Fórmulas personalizadas:** Crear medidas DAX como "ventas del mes anterior" usando la función ``PARALLELPERIOD``, lo cual exige el campo de fecha y el intervalo deseado (por ejemplo, -1 mes).

### ¿Cómo mejorar la visualización de los datos?

Una presentación clara y ordenada es fundamental. Los pasos incluyen:

- **Ordenar datos:** Seleccionar un orden lógico, como ordenar los meses por el número del mes.
- **Visualización:** Crear gráficos y tablas que comparen datos actuales con periodos anteriores, clarificando las diferencias y permitiendo análisis más profundos.

Al seguir estos pasos, los datos cobran vida, dando acceso a una comprensión más profunda y permitiendo tomar decisiones informadas basadas en la historia que esos datos cuentan.
