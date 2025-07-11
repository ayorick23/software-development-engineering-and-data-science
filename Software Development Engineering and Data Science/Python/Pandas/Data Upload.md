# Data Upload

## Carga de datos

En Pandas, la carga de datos se realiza principalmente utilizando funciones como `read_csv()`, `read_excel()`, `read_json()`, entre otras, que permiten importar datos desde diferentes formatos de archivo a un DataFrame de Pandas. Estas funciones facilitan la manipulación y análisis de datos tabulares. 


```python
import pandas as pd
```

## Load data from a CSV file

La función `read_csv()` se utiliza para cargar datos desde archivos CSV (*valores separados por comas*).


```python
#Cargar datos desde un archivo CSV
df_csv = pd.read_csv('data.csv') #asegurarse de tener el archivo data.csv en la misma carpeta que el script
print(df_csv.head())
```

            Name  Age      Occupation
    0     Dereck   25        Engineer
    1  Alejandra   28  Data Scientist
    2    Charlie   35         Teacher
    

El parámetro `sep` se puede usar para especificar el delimitador si no es una coma.

## Load data from a Excel file

La función `read_excel` se utiliza para importar datos de archivos Excel (*.xlsx o .xls*) a un DataFrame. Esta función es versátil y permite leer hojas individuales o múltiples hojas, especificando su nombre o índice. 


```python
#Cargar datos desde un archivo Excel
df_excel = pd.read_excel('data.xlsx', sheet_name='Hoja1') #requiere openpyxl instalado
print(df_excel.head())
```

            Name  Age      Occupation
    0     Dereck   25       Engineeer
    1  Alejandra   28  Data Scientist
    2    Charlie   30         Teacher
    

Se puede especificar la hoja de cálculo a leer con el parámetro `sheet_name`.

## Loading data from a JSON file

La función `read_json()` se utiliza para leer datos desde un archivo JSON o una cadena JSON y convertirlos en un DataFrame. Esta función es útil para trabajar con datos estructurados en formato JSON, ya sea que estén almacenados en archivos locales o disponibles a través de URLs.


```python
#Cargar datos desde un archivo JSON
df_json = pd.read_json('data.json')
print(df_json.head())
```

                                                    data
    0  {'id': 1, 'name': 'Dereck Mendez', 'age': 25, ...
    1  {'id': 2, 'name': 'Alejandra Martinez', 'age':...
    2  {'id': 3, 'name': 'Cristian Rodriguez', 'age':...
    

## Loading data from a URL (example with inline CSV)

Para cargar datos desde una URL en Pandas, se utiliza la función `read_csv()` (o `read_json()`, `read_html()`, etc., según el formato de los datos) pasando la URL como argumento. Pandas se encargará de descargar y parsear los datos directamente en un DataFrame.


```python
#Cargar datos desde una URL (ejemplo con CSV en linea)
url = 'https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv'
df_url = pd.read_csv(url)
print(df_url.head())
```

       sepal_length  sepal_width  petal_length  petal_width species
    0           5.1          3.5           1.4          0.2  setosa
    1           4.9          3.0           1.4          0.2  setosa
    2           4.7          3.2           1.3          0.2  setosa
    3           4.6          3.1           1.5          0.2  setosa
    4           5.0          3.6           1.4          0.2  setosa
    

## Loading data from a SQL database (example with SQLite)

Pandas proporciona la función `read_sql` para ejecutar consultas SQL y cargar los resultados en un DataFrame. Esta función actúa como contenedor de `read_sql_query` y `read_sql_table`, seleccionando automáticamente la función adecuada según la entrada.

Para usar `read_sql`, necesitas una conexión a una base de datos SQL. Esto se puede lograr mediante bibliotecas como SQLAlchemy, compatible con varios sistemas de bases de datos. Como alternativa, puedes usar un objeto de conexión directa para bases de datos como SQLite.


```python
#Cargar datos desde un archivo SQL (ejemplo con SQLite)
import sqlite3

conn = sqlite3.connect('data.db') #asegurarse de tener el archivo data.db en la misma carpeta que el script
df_sql = pd.read_sql_query("SELECT * FROM data", conn)
print(df_sql.head())

conn.close()
```
