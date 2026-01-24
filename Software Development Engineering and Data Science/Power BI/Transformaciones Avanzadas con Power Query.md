# Transformaciones Avanzadas con Power Query

## ¿Qué es una transformación en Power Query?

Las transformaciones en [[Transformación de Datos con Power Query#¿Qué son Power Query y el proceso ETL?|Power Query]] son herramientas poderosas que permiten ajustar y modelar datos para satisfacer necesidades específicas. En su esencia, una transformación en Power Query puede realizar diversas tareas, como cambiar el tipo de dato, dividir columnas, reemplazar valores o filtrar datos. Estas acciones aseguran que la información sea eficazmente modelada para un análisis posterior. Además, Power Query ofrece la posibilidad de combinar consultas y anexar datos para obtener resultados más complejos y alineados con objetivos específicos.

## ¿Cómo se realiza un ejercicio de transformación en Power BI Desktop?

Trabajar con [[Business Intelligence#¿Cómo se realiza la reportería y visualización de datos?|Power BI Desktop]] implica un enfoque práctico para aplicar transformaciones. Un ejemplo práctico en el ámbito de los recursos humanos podría involucrar el registro de horas de entrada y salida de los trabajadores. A continuación, se describen los pasos principales de un ejercicio de transformación:

1. **Importación de datos**:
    - Navegar a la sección de obtener datos.
    - Seleccionar el tipo de archivo, como texto o CSV.
    - Identificar los archivos relevantes dentro de una carpeta específica.
2. **Limpieza inicial de datos**:
    - Eliminar columnas y filas innecesarias para simplificar la estructura de los datos.
    - Utilizar opciones como "Quitar filas superiores" para deshacerse de lo innecesario.
3. **Creación de encabezados**:
    - Utilizar la primera fila como encabezado, lo que permite organizar adecuadamente los datos y hacerlos más comprensibles.

## ¿Cómo se agregan columnas condicionales en Power Query?

Agregar columnas condicionales es esencial para enriquecer el conjunto de datos con informaciones derivadas de los valores originales.

1. **Agregar columna con lógica condicional**:
    - Elegir "Agregar columna" y luego "Columna condicional".
    - Definir una lógica clara, por ejemplo, si un campo comienza con una letra específica, asignar un valor correspondiente a la nueva columna.
    - Repetir el proceso para otras variables relevantes, como un identificador o clave.
2. **Revisar tipos de datos**:
    - Es necesario asignar el tipo de dato correcto a cada columna nueva. Los tipos de datos pueden ser texto, número entero, entre otros, y deben reflejar la naturaleza de los datos contenidos.

## ¿Cómo corregir y ajustar datos problemáticos?

Durante la transformación de datos, es común encontrar inconsistencias que deben ser solucionadas.

- **Reemplazo de valores no válidos**:
    - Detectar valores incorrectos o inconsistencia en los datos, como espacios indeseados.
    - Utilizar la función "Reemplazar valores" para corregir formatos. Por ejemplo, transformar "punto espacio m punto" en "AM" o "PM".
- **Filtrar filas y ajustar tipos de datos**:
    - Continuar eliminando filas innecesarias, ya sea al inicio o al final.
    - Asignar correcta y finalmente los tipos de datos apropiados a cada columna, asegurando así que las operaciones sucesivas sean válidas y precisas.

Transformar y limpiar datos en Power Query es fundamental para garantizar que el análisis y las decisiones basadas en datos sean acertadas. Conocer y aplicar estas técnicas permite trabajar con datos de manera más efectiva, extrayendo conocimientos significativos.
