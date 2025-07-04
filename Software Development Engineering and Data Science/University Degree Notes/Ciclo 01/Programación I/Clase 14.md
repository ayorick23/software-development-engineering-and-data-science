# PROGRAMACIÓN I: clase 14

Fecha de creación: 26 de abril de 2025 19:00
Clase: PROGRAMACIÓN I
Fecha de la clase: 26 de abril de 2025

# Archivos en Python

Verificar si un archivos existe es una práctica fundamental para evitar errores al intentar leer, modificar o eliminar archivos que podrían no estar presentes.

Python nos brinda el módulo estándar `os`, el cual permite interactuar con el sistema operativo, proporciona funciones para manejar rutas, directorios, archivos, procesos, permisos y más. Dentro de este módulo, la función `os.path.isfile(ruta)` sirve para determinar si una ruta corresponde a un archivo existente.

```python
import os

ruta_archivo = 'dato/venta.csv'

if os.path.isfile(ruta_archivo):
	print("El archivo existe y está listo para usarse.")
else:
	print("El archivo no existe. Verifique la ruta.")
```

## Directorios (Carpetas) en Python

Un directorio (también llamado carpeta) es una ubicación dentro del sistema de archivos donde se pueden agrupar archivos y otros subdirectorios. Verificar la existencia de un directorio es esencial cuando un programa necesita guardar archivos, leer contenido de carpetas específicas o recorrer.

El módulo estándar `os`, que permite interactuar con el sistema operativo. Dentro de este módulo, la función `os.pathh.isdir(ruta)` es la encargada de verificar si una ruta específica corresponde a un directorio existente.

Si la carpeta no existe, el programa podría crearla usando `os.mkdir()` o `os.makedirs()` antes de continuar. De esta forma, se evitan errores como `FileNotFoundError` o `OSError`.