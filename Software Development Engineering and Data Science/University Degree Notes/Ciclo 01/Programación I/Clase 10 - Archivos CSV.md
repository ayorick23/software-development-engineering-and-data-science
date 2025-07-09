# PROGRAMACIÓN I: clase 10

Fecha de creación: 29 de marzo de 2025 13:51
Clase: PROGRAMACIÓN I
Fecha de la clase: 29 de marzo de 2025

# Archivos CSV en Python

Los archivos CSV (Comma-Separated Values) son un formato de almacenamiento de datos estructurado en texto plano. Son ampliamente utilizados para la transferencia de datos entre diferentes sistemas, debido a su simplicidad y compatibilidad con múltiples plataformas y programas.

En Python, existen diferentes métodos y librerías para manejar archivos CSV, desde el módulo estándar csv hasta herramientas más avanzadas como Pandas. A continuación, exploraremos conceptos teóricos clave sobre la manipulación de estos archivos.

## Características de los archivos CSV

- Estructura tabular: Similar a una hoja de cálculo, con filas y columnas separadas por comas (,), aunque otros delimitadores como ; o \t también pueden usarse.
- Formato ligero: Son archivos de texto plano, lo que los hace más livianos en comparación con formatos binarios como Excel.
- Interoperabilidad: Se pueden abrir y modificar en programas como Microsoft Excel, Google Sheets y bases de datos SQL.
- No soporta tipos de datos: Todo se almacena como texto; si se necesitan números o fechas, es necesario convertirlos al tipo de dato adecuado al procesarlos en Python.

### Módulo `csv`: Manejo de archivos CSV en Python

Python incluye el módulo estándar csv que proporciona herramientas específicas para manipular archivos CSV de forma eficiente.

### `csv.reader`: Lectura de archivos CSV

El objeto `csv.reader` permite leer un archivo CSV y procesar cada línea como una lista de valores.

```python
import csv

with open("archivo.csv", mode="r", encoding="utf-8") as file:
	reader = csv.reader(file)
	for row in reader:
		print(row) #cada fila es una lista
```

- `csv.reader()` interpreta automáticamente los delimitadores.
- Se puede especificar un delimitador diferente usando `delimiter=’;’` si el archivo no usa comas.

### `csv.writer`: Escritura de archivos CSV

Para escribir en un archivo CSV, se usa `csv.writer`.

```python
import csv

data = [
	["ID", "Nombre", "Edad"],
	[1, "Juan", 30],
	[2, "Maria", 25]
]

with open("archivo.csv", mode="w", newline="", encoding="utf-8") as file:
	writer = csv.writer(file)
	writer.writerows(data) #escribe varias filas
```

- `newline=""` evita la inclusión de líneas en blanco adicionales.
- `writerow()` escribe una única fila.
- `writerows()` escribe múltiples filas a la vez.

## Manipulación avanzada con Pandas

Para grandes volúmenes de datos, la librería Pandas proporciona métodos optimizados para manipular CSVs.

Pandas no viene incluido en la biblioteca estándar de Python, por lo que necesitas instalarlo antes de usarlo. Puedes instalarlo con el siguiente comando.

```bash
pip install pandas
```

### Lectura con `pandas.read_csv()`

```python
import pandas as pd

df = pd.read_csv("archivo.csv")
print(df.head()) #muestra las primeras 5 filas
```

### Escritura con `to_csv()`

```python
import pandas as pd

#Crear un DataFrame con datos ficticios
data = {
	"ID": [1, 2, 3, 4, 5],
	"Nombre": ["Ana", "Carlos", "Beatriz", "David", "Elena"],
	"Edad": [23, 30, 25, 35, 28],
	"Salario": [1500.5, 2200.75, 1800.0, 2500.3, 2000.0]
}

df = pd.DataFrame(data)

#Guardar el DataFrame en un archivo CSV
df.to_csv("empleados.csv", index=False, encoding="utf-8")

print("Archivo 'empleados.csv' guardado correctamente")
```