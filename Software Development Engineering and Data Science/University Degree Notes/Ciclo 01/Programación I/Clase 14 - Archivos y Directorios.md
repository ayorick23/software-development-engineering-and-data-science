# PROGRAMACIÓN I: clase 14

Fecha de creación: 26 de abril de 2025 19:00
Clase: PROGRAMACIÓN I
Fecha de la clase: 26 de abril de 2025

[[Clase 11 - Funciones|← Clase anterior]] | [[Clase 15 - Pruebas Unitarias y Módulos|Clase siguiente →]]

# Archivos en Python
(ver [[Files Handling]], continuación de [[Clase 09 - Manejo de Archivos|Clase 09]])

Verificar si un archivos existe es una práctica fundamental para evitar errores al intentar leer, modificar o eliminar archivos que podrían no estar presentes.

Python nos brinda el módulo estándar `os` (ver [[Files Handling#Módulo `os`|Módulo os]]), el cual permite interactuar con el sistema operativo, proporciona funciones para manejar rutas, directorios, archivos, procesos, permisos y más. Dentro de este módulo, la función `os.path.isfile(ruta)` sirve para determinar si una ruta corresponde a un archivo existente (ver [[Files Handling#Validez de Rutas de Archivos|Validez de Rutas de Archivos]]).

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

Si la carpeta no existe, el programa podría crearla usando `os.mkdir()` o `os.makedirs()` (ver [[Files Handling#Creación de Nuevas Carpetas|Creación de Nuevas Carpetas]]) antes de continuar. De esta forma, se evitan errores como `FileNotFoundError` o `OSError`.