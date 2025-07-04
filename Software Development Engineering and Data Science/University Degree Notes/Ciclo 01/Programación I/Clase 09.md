# PROGRAMACIÓN I: clase 9

Fecha de creación: 22 de marzo de 2025 13:57
Clase: PROGRAMACIÓN I
Fecha de la clase: 22 de marzo de 2025

# Manejo de archivos

El manejo de archivos en Python permite interactuar con datos almacenados en archivos externos, como texto o binarios. Python proporciona funciones y métodos integrados para abrir, leer, escribir, cerrar y manipular archivos de manera eficiente.

## ¿Qué es una archivo?

Un archivo es una colección de datos almacenados en un dispositivo de almacenamiento (disco duro, SSD, memoria USB, etc.). Los archivos pueden ser de dos tipos principales:

- **Archivos de texto:** Contienen caracteres en un formato legible por humanos (.txt, .csv, .json).
- **Archivos binarios:** Contienen caracteres en un formato binario, no directamente legible (.jpg, .mp3, .exe).

## Apertura de archivos en Python

Python usa la función open() par abrir archivos. Su sintaxis es:

```python
archivo = open("nombre_archivo", "modo")
```

Donde:

- `nombre_archivo` es el nombre del archivo (incluyendo su ruta si no está en el mismo directorio).
- `modo` define la operación que se realizará con el archivo.

Ejemplo de apretura de un archivo de texto:

```python
archivo = open("datos.txt", "r")
```

Python también recomienda usar la declaración `with` para asegurarse de que el archivo se cierra automáticamente después de su uso:

```python
with open("datos.txt", "r") as archivo:
	contenido = archivo.read()
	print(contenido)
```

El with statement es una construcción en Python que simplifica la gestión de recursos que deben ser limpiados después de su uso. Por ejemplo archivos, conexiones de bases de datos o bloques de código que necesitan liberar recursos específicos.

## Modos de apertura de archivos

| Modo | Descripción |
| --- | --- |
| r | Abre el archivo en modo lectura. Falla si no existe |
| w | Abre el archivo en modo escritura. Si existe, lo sobrescribe, sino, lo crea |
| a | Abre el archivo en modo anexar. Si no existe, lo crea |
| r+ | Abre el archivo para lectura y escritura. Debe existir |
| w+ | Abre el archivo para lectura y escritura. Sobrescribe el archivo si existe |
| a+ | Abre el archivo para lectura y anexar contenido |
| rb | Abre el archivo en modo lectura binaria |
| wb | Abre el archivo en modo escritura binaria |
| ab | Abre el archivo en modo anexar binario |
| r+b | Abre el archivo para lectura y escritura en binario |
| w+b | Abre el archivo para escritura y lectura en binario (sobrescribe el archivo) |
| a+b | Abre el archivo para anexar y lectura en binario |

### Escritura de archivos

```python
with open("ejemplo.txt", "w") as archivo:
	archivo.write("Hola, este es un archivo de prueba.")
```

### Lectura de archivos

Para leer un archivo, se pueden usar los siguientes métodos:

`read()` - Leer todo el contenido del archivo

```python
with open("datos.txt", "r") as archivo:
	contenido = archivo.read()
	print(contenido)
```

`readline()` - Leer una línea del archivo

```python
with open("datos.txt", "r") as archivo:
	primera_linea = archivo.readline()
	print(primera_linea)
```

`readlines()` - Leer todas las líneas como una lista

```python
with open("datos.txt", "r") as archivo:
	lineas = archivo.readlines()
	for linea in lineas:
		print(linea.strip()) #.strip() elimina los saltos de linea
```

### Escritura de archivos

Para escribir en un archivo, se usa el método `write()`.

Escribir con `write()`

```python
with open("salid.txt", "w") as archivo:
	archivo.write("Este es un ejemplo de escritura en archivos.\n")
	archivo.write("Seguna linea de texto")
```

<aside>
<img src="notion://custom_emoji/b356a38d-82ed-44c1-bf18-c24d9d5e694a/1b4554ee-ae2e-80a3-bab8-007a8b65ff5e" alt="notion://custom_emoji/b356a38d-82ed-44c1-bf18-c24d9d5e694a/1b4554ee-ae2e-80a3-bab8-007a8b65ff5e" width="40px" />

Nota: el modo “w” sobrescribe el contenido del archivo si ya existe

</aside>

Agregar contenido con `a` (modo anexar)

```python
with open("salida.txt", "a") as archivo:
	archivo.write("\nEsta s una nueva linea agregada")
```

### Movimiento de archivos con `seek()` y `tell()`

Python permite mover el puntero del archivo con:

- `seek(posición)`: Mueve el cursos a una posición específica.
- `tell()`: Retorna la posición actual del cursor.

```python
with open("datos.txt", "r") as archivo:
	archivo.seek(10) #mueve el puntero al byte 10
	print(archivo.read(5)) #lee 5 caracteres desde la posición actual
	print(archivo.tell()) #muestra la posición actual del cursor
```

### Cierre de archivos

Es importante cerrar los archivos después de su uso para evitar pérdidas de datos o bloques de acceso. Se usa el método `close()`:

```python
archivo = open("datos.txt", "r")
contenido = archivo.read()
archivo.close() #cierra el archhivo manualmente
```

<aside>
<img src="notion://custom_emoji/b356a38d-82ed-44c1-bf18-c24d9d5e694a/1b4554ee-ae2e-80a3-bab8-007a8b65ff5e" alt="notion://custom_emoji/b356a38d-82ed-44c1-bf18-c24d9d5e694a/1b4554ee-ae2e-80a3-bab8-007a8b65ff5e" width="40px" />

Sin embargo, usar `with open()` es preferible porque cierra el archivo automáticamente.

</aside>