# Integración con Python

## ¿Cómo se integra Python con Power BI?

La integración de [[Introduction to Python#Introduction to Python|Python]] en Power BI es una poderosa funcionalidad que permite a los analistas de datos ampliar las capacidades de visualización y análisis de esta herramienta. Gracias a Python, puedes limpiar y transformar datos, así como crear visualizaciones avanzadas utilizando diversas bibliotecas. A continuación, exploraremos el proceso para implementar Python dentro de Power BI Desktop.

### ¿Qué necesitas para empezar con la integración?

Para comenzar con la integración de Python en Power BI, primero debes descargar un archivo CSV relacionado con tarjetas de crédito. Este archivo proporcionará datos sobre el comportamiento de líneas de crédito de varios clientes de un banco. Una vez que tengas el archivo, sigue estos pasos:

1. **Cargar los datos en Power BI:**
    - Dentro de Power BI Desktop, selecciona "Obtener datos" y carga el archivo CSV ubicado en la carpeta designada para integraciones avanzadas.
2. **Configuraciones iniciales:**
    - Crea segmentaciones de datos basadas en variables como zona, tipo de tarjeta y estado civil.
    - Personaliza el lienzo del reporte añadiendo fondos e imágenes, como el logo del banco, para mejorar la presentación visual.

### ¿Cómo instalar y configurar Anaconda para Python?

Anaconda es un entorno de desarrollo integrado (IDE) muy eficaz para trabajar con Python, y es esencial para configurar Python en Power BI. Sigue esta guía para instalarlo y configurarlo:

1. **Descarga e instalación de Anaconda:**
    - Visita [anaconda.com/download](https://www.anaconda.com/download) y descarga el instalador.
    - Ejecuta el instalador, selecciona "Para todos los usuarios" y sigue las instrucciones para la instalación completa.
2. **Configuración en Anaconda Navigator:**
    - Abre Anaconda Navigator y crea un nuevo entorno que nombrarás como "Power BI Platzi" (o similar según tu preferencia).
    - Dentro del nuevo entorno, instala bibliotecas esenciales de Python como [[Series and Dataframes#Introducción a Pandas|Pandas]], [[Array Creation#Introducción a NumPy|NumPy]] y [[Introduction to Matplotlib#Introducción a Matplotlib|Matplotlib]]. Estas son fundamentales para la manipulación y visualización avanzada de datos.
3. **Configuración en Power BI Desktop:**
    - Dirígete a las opciones de Power BI Desktop y selecciona la ruta raíz de Python creada en Anaconda. Asegúrate de establecer la configuración correcta para scripts de Python.

### ¿Cómo crear visualizaciones de Python en Power BI?

Ahora que ya tienes Python configurado dentro de Power BI, puedes comenzar a crear visualizaciones personalizadas:

1. **Crear un objeto visual de Python:**
    - En Power BI, ve a la sección de "Compilar" y selecciona "Objeto visual de Python". Habilita la edición de scripts cuando se te solicite.
2. **Escribir y pegar código de Python:**
    - Utiliza scripts de Python para generar visualizaciones adecuadas. Por ejemplo, puedes crear gráficos de dispersión utilizando las bibliotecas Pandas y Matplotlib, tal como se muestra en un archivo de recursos compartido.
    - Selecciona las variables adecuadas y configura la visualización para que coincida con tus necesidades analíticas.
3. **Ejecutar y ajustar la visualización:**
    - Después de configurar el script, presiona "Play" para ejecutar el código y ver la visualización generada. Puedes utilizar diferentes scripts proporcionados para crear otras visualizaciones más complejas.

La integración de Python con Power BI abre un mundo de posibilidades, permitiendo a los analistas crear informes más robustos y visualizaciones personalizadas que van más allá de las capacidades estándar de Power BI.
