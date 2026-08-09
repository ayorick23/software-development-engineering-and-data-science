# Análisis de Datos: Validación de Hipótesis
(aplica en un caso concreto el [[Diseño de Proyectos de PA|Diseño de Proyectos]] anterior: plantear y probar una hipótesis específica)

[[Diseño de Proyectos de PA|← Anterior]] | [[Integración y Análisis de Encuestas de Satisfacción|Siguiente →]]

## ¿Cómo validar una hipótesis sobre la rotación de personal utilizando análisis de datos?

La práctica del análisis de datos se ha convertido en una herramienta esencial para validar hipótesis y tomar decisiones informadas en las organizaciones. En este segmento, vamos a explorar cómo un gerente podría usar datos para investigar si la distancia al trabajo influye en la rotación de empleados. Este enfoque no solo ilumina el camino hacia una comprensión más profunda de los factores de rotación, sino que también ofrece la oportunidad de utilizar herramientas avanzadas como la inteligencia artificial para mejorar la productividad en análisis complejos.

### ¿Cuál es la hipótesis planteada?

La hipótesis en cuestión sugiere que la distancia del domicilio al lugar de trabajo puede estar impactando negativamente en la rotación de personal. Según el gerente de la empresa, es posible que los empleados que viven más lejos tiendan a abandonar la organización con mayor frecuencia. Para validar esta idea, se propone investigar si debería priorizarse la contratación de personas que vivan cerca de la oficina.

### ¿Cómo se calculan las distancias al trabajo?

Para determinar si la distancia realmente afecta la rotación, el primer paso es calcular las trayectorias desde los domicilios de los empleados hasta su oficina. Aunque a primera vista esto pueda parecer una tarea laboriosa -- especialmente con bases de datos grandes --, existe una solución eficiente gracias a la inteligencia artificial, específicamente utilizando **ChatGPT**.

1. **Obtener direcciones**: comienza recopilando las direcciones de los empleados de la base de datos.
2. **Utilizar inteligencia artificial** (ver [[10-IA-Generativa-LLMs-y-Agentes#Prompt Engineering|Prompt Engineering]]): mediante un prompt en ChatGPT, solicita el cálculo experimental de las distancias en kilómetros entre dos puntos de interés, por ejemplo, en Bogotá, Colombia.
3. **Automatización en masa**: para un análisis más ágil y rápido, formula otro prompt para calcular las distancias para todos los registros de empleados y presentarlas en una tabla de Excel.

### ¿Qué tipo de análisis se puede realizar con estos datos?

Una vez obtenidas las distancias, se pueden realizar varios tipos de análisis para profundizar la comprensión sobre la relación entre la lejanía y la rotación:

#### Análisis exploratorio con gráficos

**Gráfico de dispersión**: utiliza este tipo de gráfico para visualizar las relaciones entre las variables cuantitativas, como la antigüedad y la distancia al trabajo. Este gráfico ayuda a identificar tendencias o patrones que apoyen, o refuten, la hipótesis planteada.

#### Análisis estadístico

1. **Correlación**: calcula el coeficiente de correlación para evaluar el grado de asociación entre la distancia al trabajo y otros factores como la antigüedad. Recuerda que una fuerte correlación no implica causalidad, pero puede ser un paso preliminar que guíe investigaciones más profundas.
2. **Comparación con datos históricos**: evalúa las direcciones de empleados que han dejado la empresa previamente para entender si la lejanía fue un factor común en sus decisiones de salida.

### ¿Qué recomendaciones se pueden extraer de los análisis?

- **Contrataciones cercanas**: aunque no se encontró una correlación fuerte, aún es sensato considerar el lugar de residencia al contratar, particularmente si subir costos de desplazamiento podrían afectar el bienestar del empleado.
- **Factores adicionales**: profundizar en otras variables que podrían estar influyendo en la decisión de abandonar la empresa, como satisfacción laboral o beneficios.

Este método de análisis, que combina hojas de cálculo, inteligencia artificial, y herramientas gráficas, ofrece una manera ágil y guiada de abordar y validar hipótesis organizacionales, enriqueciendo el proceso de toma de decisiones con datos duros y capaces de transformarse en insights valiosos.
