# <img width="35" height="35" src="https://img.icons8.com/?size=100&id=xSkewUSqtErH&format=png&color=000000" alt="pandas"> Introduction to Pandas

## Introducción a Pandas

**Pandas** es una de las librerías más utilizadas en Python para la manipulación y análisis de datos. Proporciona estructuras de datos rápidas, flexibles y expresivas diseñadas para trabajar con datos etiquetados (tablas, series temporales, etc.).


# Series and DataFrames

## Import Pandas


```python
import pandas as pd
```

## Check Pandas version

Antes de comenzar, es buena práctica verificar la versión instalada de `pandas`, especialmente para asegurar la compatibilidad de funciones:


```python
#Verificar la versión de Pandas
print("Version de Pandas:", pd.__version__)
```

    Version de Pandas: 2.2.3
    

## Creating a Series

Una Serie es una estructura unidimensional similar a un array o lista, pero con etiquetas (_índices_).


```python
#Creación de una Serie
data = [10, 20, 30, 40, 50]
serie = pd.Series(data)
print(serie)
```

    0    10
    1    20
    2    30
    3    40
    4    50
    dtype: int64
    

Esto crea una serie con índices predeterminados (0, 1, 2, ...).

## Creating a Series with custom indexes

Puedes definir tus propios índices para mayor control o legibilidad:


```python
#Creación de Series con índices personalizados
customSerie = pd.Series(data, index=['a', 'b', 'c', 'd', 'e'])
print(customSerie)
```

    a    10
    b    20
    c    30
    d    40
    e    50
    dtype: int64
    

Esto es útil cuando quieres que cada elemento tenga una etiqueta significativa.

## Accessing values of a Series

Puedes acceder a los valores utilizando el índice como si fuera un diccionario o una lista:


```python
#Acceder a valores de una Serie
print(f"Valor en índice 'c': {customSerie['c']}") # Acceso por etiqueta
print(f"Valor en posición 1 : {customSerie[1]}")  # Acceso por posición
```

    Valor en índice 'c': 30
    Valor en posición 1 : 20
    

    C:\Users\Dereck\AppData\Local\Temp\ipykernel_22128\4125903670.py:3: FutureWarning: Series.__getitem__ treating keys as positions is deprecated. In a future version, integer keys will always be treated as labels (consistent with DataFrame behavior). To access a value by position, use `ser.iloc[pos]`
      print(f"Valor en posición 1 : {customSerie[1]}")  # Acceso por posición
    

## Creating a DataFrame

Un DataFrame es una estructura bidimensional (*tabla*), similar a una hoja de cálculo o a una tabla SQL.


```python
#Creación de un DataFrame
datos = {
    'Nombre': ['Alejandra', 'Carlos', 'Henry', 'Cristian'],
    'Edad': [25, 30, 35, 40],
    'Ciudad': ['Barcelona', 'Madrid', 'Valencia', 'Sevilla']
}
df = pd.DataFrame(datos)
print(df)
```

          Nombre  Edad     Ciudad
    0  Alejandra    25  Barcelona
    1     Carlos    30     Madrid
    2      Henry    35   Valencia
    3   Cristian    40    Sevilla
    

## Access a DataFrame column


```python
#Acceder a una columna del DataFrame
print(df['Nombre'])         # Devuelve una Serie
print(df[['Nombre']])       # Devuelve un DataFrame
print(type(df['Nombre']))
print(type(df[['Nombre']]))
```

    0    Alejandra
    1       Carlos
    2        Henry
    3     Cristian
    Name: Nombre, dtype: object
          Nombre
    0  Alejandra
    1     Carlos
    2      Henry
    3   Cristian
    <class 'pandas.core.series.Series'>
    <class 'pandas.core.frame.DataFrame'>
    

## Access a DataFrame row

### Por índice entero con `iloc[]`


```python
#Acceder a una fila del DataFrame
print(df.iloc[1]) # Segunda fila
```

    Nombre    Carlos
    Edad          30
    Ciudad    Madrid
    Name: 1, dtype: object
    

### Por etiqueta con `loc[]`

Si defines un índice personalizado:


```python
df = pd.DataFrame(datos, index=['a', 'b', 'c', 'd'])
print(df.loc['b'])  # Fila con índice 'b'
```

    Nombre    Carlos
    Edad          30
    Ciudad    Madrid
    Name: b, dtype: object
    

## Access a specific cell

Combinando `loc[]` o `iloc[]` con nombres de columnas:


```python
#Acceder a una celda específica
print(df.loc['b', 'Edad'])  # Edad de la persona con índice 'b'
print(df.iloc[1, 1])        # Segunda fila, segunda columna
```

    30
    30
    

Utilizando `at[]` para acceder y modificar un único valor en un DataFrame utilizando etiquetas de fila y columna. Es similar a `loc[]`, pero está optimizado para acceder a un solo valor, lo que puede resultar en un mejor rendimiento en esos casos. 


```python
print(f"Edad de la tercera persona: {df.at['c', 'Edad']}")
```

    Edad de la tercera persona: 35
    
