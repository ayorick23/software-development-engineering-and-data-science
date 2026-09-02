# Docker for Data Sciencie and Machine Learning

Mientras que Docker es ampliamente conocido para el despliegue de aplicaciones web, su valor en el campo de la Ciencia de Datos (DS) y el Aprendizaje Automático (ML) es inmenso. Los científicos de datos a menudo se enfrentan a problemas de reproducibilidad, gestión de dependencias y consistencia de entornos que Docker puede resolver de manera eficiente.

## Problemas Clave en DS/ML que Docker Resuelve

1. **"Funciona en mi máquina":** Un modelo de Machine Learning desarrollado con éxito puede fallar al ser ejecutado en la máquina de otro colaborador o en un servidor de producción. Esto se debe a diferencias en las versiones de bibliotecas ([[07-Librerias-de-Data-Science-y-ML#TensorFlow|TensorFlow]], [[07-Librerias-de-Data-Science-y-ML#PyTorch|PyTorch]], [[07-Librerias-de-Data-Science-y-ML#Scikit-learn|Scikit-learn]]), el sistema operativo o las dependencias del sistema (como CUDA para GPUs).
2. **Gestión de dependencias:** Los proyectos de DS y ML suelen requerir decenas de bibliotecas, cada una con versiones específicas que pueden entrar en conflicto. Instalar y gestionar estas dependencias manualmente puede ser una tarea tediosa y propensa a errores.
3. **Reproducibilidad:** Publicar un trabajo de investigación o un modelo no es suficiente. Para que sea científicamente válido, otros investigadores deben poder reproducir tus resultados exactamente. Sin un entorno consistente, la reproducibilidad se vuelve casi imposible.
4. **Entornos de producción vs. desarrollo:** Un modelo entrenado localmente necesita un entorno de ejecución idéntico en un servidor en la nube para el serving (servir predicciones). Docker asegura que el entorno de entrenamiento sea el mismo que el entorno de inferencia.

## Flujo de Trabajo con Docker en un Proyecto de DS/ML

El uso de Docker en un proyecto de ciencia de datos sigue una lógica clara y ordenada:

1. **Definir el Entorno:** En lugar de instalar dependencias globalmente, las defines en un Dockerfile. Especificas la imagen base (ej. `python:3.9`), instalas las bibliotecas necesarias ([[Python/Pandas/01 - Introducción y Arquitectura Interna|pandas]], [[Python/NumPy/01 - Introducción y Arquitectura Interna|numpy]], `scikit-learn`) y otras herramientas (como Jupyter).
2. **Construir la Imagen:** Se construye una [[Images#Construyendo Imágenes con Dockerfiles|imagen]] de Docker a partir de tu Dockerfile. Esta imagen es un paquete auto-contenido que contiene todo lo necesario para ejecutar tu código de análisis o tu modelo.
3. **Ejecutar el Contenedor:** Inicias un [[Containers#Ciclo de Vida del Contenedor|contenedor]] a partir de la imagen. Puedes ejecutar scripts de análisis, entrenar modelos o incluso lanzar un servidor de Jupyter Notebook dentro del contenedor.
4. **Compartir el Entorno:** La imagen de Docker se puede compartir fácilmente. Puedes subirla a [[Introduction to Docker#Docker Hub|Docker Hub]] o a un registro privado para que otros colaboradores la descarguen y la usen. Esto garantiza que todos trabajen en el mismo entorno.

## Ejemplos de Uso Práctico

### Entorno de Jupyter Notebooks

Uno de los usos más comunes es crear un entorno de trabajo con Jupyter. En lugar de instalar todo el ecosistema de Python localmente, lo encapsulas en un contenedor.

- **Dockerfile:** Define una imagen base de Python, instala jupyter, pandas, [[Python/Matplotlib/01 - Introducción y Arquitectura|matplotlib]], etc., y define el puerto y el comando para lanzar Jupyter Notebook.
- **Comando:** `docker run -it -p 8888:8888 -v $(pwd):/app mi-imagen-ds jupyter notebook --ip=0.0.0.0 --allow-root`
- **Beneficio:** Cualquier colaborador solo necesita Docker. Al ejecutar el comando, tendrá un Jupyter Notebook listo con todas las dependencias correctas, sin afectar su sistema local.

### Entrenamiento de Modelos de Machine Learning (MLOps)

En un proyecto de MLOps (_Machine Learning Operations_), Docker es fundamental para el entrenamiento y despliegue de modelos.

- **Dockerfile:** Construyes una imagen que contenga tu modelo, el código de entrenamiento y todas las bibliotecas de ML (`tensorflow`, `pytorch`).
- **Comando de entrenamiento:** Puedes lanzar el contenedor para ejecutar el script de entrenamiento del modelo.
- **Servicio de inferencia:** Una vez entrenado el modelo, lo sirves (creas un servicio para que haga predicciones). Creas una nueva imagen que solo contenga el modelo serializado y un servidor web ligero (ej. [[APIs#Crear una API con Flask|Flask]]) para exponer un API de predicciones.

### Proyectos con GPU

Docker también puede aprovechar el hardware especializado, como las GPUs, lo cual es crítico en el aprendizaje profundo.

- **NVIDIA Container Toolkit:** Una extensión de Docker que permite a los contenedores acceder a las GPUs del host.
- **Imágenes base:** Se utilizan imágenes base especiales de `nvidia-cuda` que ya vienen con los controladores y bibliotecas necesarias.
- **Comando:** Al ejecutar el contenedor, se usa el flag `--gpus all` para darle acceso a todas las GPUs.
- **Beneficio:** Evitas el complejo y frágil proceso de instalar los controladores y las bibliotecas CUDA en tu sistema, encapsulando todo en una imagen.

## Buenas Prácticas y Consejos para DS/ML

- **Mantenlo Ligero:** Usa imágenes base pequeñas (`python:3.9-slim`, `miniconda3`) para reducir el tamaño final de la imagen.
- **Usa `requirements.txt`:** Genera un archivo `requirements.txt` con tus dependencias (`pip freeze > requirements.txt`) para instalar solo lo que necesitas (ver [[Package Managers#Gestionando Dependencias con `requirements.txt`|requirements.txt]]).
- **Versiones Claras:** En tu Dockerfile, especifica siempre las versiones exactas de las bibliotecas para asegurar la reproducibilidad (`pandas==1.3.3`).
- **Volúmenes para los datos:** No incluyas los datos en la imagen. Usa volúmenes para montar el directorio de datos desde tu máquina host al contenedor. Esto mantiene la imagen ligera y te permite trabajar con grandes conjuntos de datos sin necesidad de reconstruir la imagen.
- **Ignora los artefactos:** Usa un archivo `.dockerignore` para excluir archivos innecesarios como modelos entrenados, archivos de caché o conjuntos de datos que no deben ser parte de la imagen.
