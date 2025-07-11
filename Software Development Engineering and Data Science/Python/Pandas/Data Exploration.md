# Data Exploration

## Exploración de Datos

La exploración de datos en **Pandas** implica entender la estructura, contenido y características de un conjunto de datos utilizando las herramientas y funciones de la biblioteca. Esto incluye la importación de datos, la visualización inicial, la revisión de tipos de datos, la identificación de valores faltantes y la obtención de estadísticas descriptivas, entre otras acciones.


```python
import pandas as pd
```

## Load a sample dataset


```python
#Cargar un dataset de ejemplo
data = pd.read_csv('https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv')
```

## Display the first rows of the DataFrame

`head()` es un método que se utiliza para mostrar las primeras n filas de un DataFrame. Por defecto, si no se especifica un número, muestra las primeras 5 filas.


```python
#Mostrar las primeras 5 filas del dataset
print(data.head())
```

       sepal_length  sepal_width  petal_length  petal_width species
    0           5.1          3.5           1.4          0.2  setosa
    1           4.9          3.0           1.4          0.2  setosa
    2           4.7          3.2           1.3          0.2  setosa
    3           4.6          3.1           1.5          0.2  setosa
    4           5.0          3.6           1.4          0.2  setosa
    

## Display the last rows of the DataFrame

`tail()` se utiliza para mostrar las últimas filas de un DataFrame. Por defecto, muestra las últimas 5 filas, pero se puede especificar un número diferente de filas pasando un argumento entero al método.


```python
#Mostrar las últimas 5 filas del dataset
print(data.tail())
```

         sepal_length  sepal_width  petal_length  petal_width    species
    145           6.7          3.0           5.2          2.3  virginica
    146           6.3          2.5           5.0          1.9  virginica
    147           6.5          3.0           5.2          2.0  virginica
    148           6.2          3.4           5.4          2.3  virginica
    149           5.9          3.0           5.1          1.8  virginica
    

## Get general information about the DataFrame

`info()` es un método que proporciona un resumen conciso del DataFrame. Muestra información sobre el índice, las columnas, los tipos de datos, el número de valores no nulos y el uso de memoria del DataFrame.


```python
#Obtener información general del dataset
print(data.info())
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 150 entries, 0 to 149
    Data columns (total 5 columns):
     #   Column        Non-Null Count  Dtype  
    ---  ------        --------------  -----  
     0   sepal_length  150 non-null    float64
     1   sepal_width   150 non-null    float64
     2   petal_length  150 non-null    float64
     3   petal_width   150 non-null    float64
     4   species       150 non-null    object 
    dtypes: float64(4), object(1)
    memory usage: 6.0+ KB
    None
    

## Get descriptive statistics from the DataFrame

`describe()` proporciona estadísticas descriptivas para las columnas numéricas de un DataFrame. Devuelve un resumen de las estadísticas clave, como la *media*, la *desviación estándar*, los valores *mínimo* y *máximo*, los *cuartiles*, entre otros.

### ¿Qué información proporciona?
- **count:** El número de valores no nulos en la columna.
- **mean:** El promedio de los valores en la columna.
- **std:** La desviación estándar de los valores en la columna.
- **min:** El valor mínimo en la columna.
- **25% (Q1):** El primer cuartil, que representa el valor por debajo del cual se encuentra el 25% de los datos.
- **50% (Q2 o mediana):** El segundo cuartil, que representa el valor por debajo del cual se encuentra el 50% de los datos, también conocido como la mediana.
- **75% (Q3):** El tercer cuartil, que representa el valor por debajo del cual se encuentra el 75% de los datos.
- **max:** El valor máximo en la columna.


```python
#Obtener estadísticas descriptivas del DataFrame
print(data.describe())
```

           sepal_length  sepal_width  petal_length  petal_width
    count    150.000000   150.000000    150.000000   150.000000
    mean       5.843333     3.057333      3.758000     1.199333
    std        0.828066     0.435866      1.765298     0.762238
    min        4.300000     2.000000      1.000000     0.100000
    25%        5.100000     2.800000      1.600000     0.300000
    50%        5.800000     3.000000      4.350000     1.300000
    75%        6.400000     3.300000      5.100000     1.800000
    max        7.900000     4.400000      6.900000     2.500000
    

## View the number of rows and columns

`shape` es un atributo que te devuelve una tupla con el número de filas y columnas de un DataFrame.


```python
#Ver el número de filas y columnas
print(f"Filas y columnas: {data.shape}")
```

    Filas y columnas: (150, 5)
    

**NOTA:** `shape` es un atributo, no un método. Esto significa que no se usa con paréntesis como `data.shape()`. Se accede directamente como `data.shape`.

## Get the column names

`columns` es un atributo que devuelve los nombres de las columnas de un DataFrame como un objeto Index de Pandas. Permite acceder, manipular y trabajar con las etiquetas de las columnas.


```python
#Obtener los nombres de las columnas
print(data.columns)
```

    Index(['sepal_length', 'sepal_width', 'petal_length', 'petal_width',
           'species'],
          dtype='object')
    

## Count the unique values in each column

`nunique()` se utiliza para contar el número de valores únicos en una columna o filas de un DataFrame. Devuelve un escalar si se aplica a una serie, o una nueva serie con el recuento de valores únicos por columna/fila si se aplica a un DataFrame.


```python
#Contar los valores únicos de cada columna
print(data.nunique())
```

    sepal_length    35
    sepal_width     23
    petal_length    43
    petal_width     22
    species          3
    dtype: int64
    

## Count the unique values in each specific column

La función `value_counts()` se utiliza para contar la frecuencia de valores únicos en una Serie o columna de un DataFrame. Devuelve una nueva Serie con los valores únicos como índice y sus conteos como valores, ordenados de mayor a menor por defecto.


```python
#Contar los valores únicos de cada columna específica
print(data['species'].value_counts())
```

    species
    setosa        50
    versicolor    50
    virginica     50
    Name: count, dtype: int64
    

## View the data type of each column

dtypes se refiere a los tipos de datos de las columnas en un DataFrame. dtypes es una propiedad que devuelve una Serie con el tipo de dato de cada columna, donde el índice de la Serie son las columnas del DataFrame original.

### Ejemplos de tipos de datos en Pandas:
- **int64:** Enteros de 64 bits. Por ejemplo, números enteros como 1, 2, 100, etc.
- **float64:** Números de punto flotante (decimales). Por ejemplo, 3.14, -2.5, 0.0.
- **object:** Generalmente utilizado para cadenas de texto, pero también puede contener otros objetos Python, como listas o diccionarios.
- **bool:** Valores booleanos (verdadero o falso).
- **datetime64[ns]:** Fechas y horas.


```python
#Ver el tipo de datos de cada columna
print(data.dtypes)
```

    sepal_length    float64
    sepal_width     float64
    petal_length    float64
    petal_width     float64
    species          object
    dtype: object
    
