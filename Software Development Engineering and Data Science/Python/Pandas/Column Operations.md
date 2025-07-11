# Column Operations


```python
import pandas as pd
```

## Load a sample dataset


```python
#Cargar un dataset de ejemplo
df = pd.read_csv('https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv')
```

## Create a new column based on another


```python
#Crear una columna basada en otra
df["sepal_area"] = df ["sepal_length"] * df["sepal_width"]
print(df.head())
```

       sepal_length  sepal_width  petal_length  petal_width species  sepal_area
    0           5.1          3.5           1.4          0.2  setosa       17.85
    1           4.9          3.0           1.4          0.2  setosa       14.70
    2           4.7          3.2           1.3          0.2  setosa       15.04
    3           4.6          3.1           1.5          0.2  setosa       14.26
    4           5.0          3.6           1.4          0.2  setosa       18.00
    

## Modify values of an existing column


```python
#Modificar valores de una columna existente
df["sepal_length"] = df["sepal_length"] * 10
print(df.head())
```

       sepal_length  sepal_width  petal_length  petal_width species  sepal_area
    0          51.0          3.5           1.4          0.2  setosa       17.85
    1          49.0          3.0           1.4          0.2  setosa       14.70
    2          47.0          3.2           1.3          0.2  setosa       15.04
    3          46.0          3.1           1.5          0.2  setosa       14.26
    4          50.0          3.6           1.4          0.2  setosa       18.00
    

## Rename columns


```python
#Renombrar columnas
df.rename(columns={"sepal_length": "sepal_length_mm"}, inplace=True)
print(df.head())
```

       sepal_length_mm  sepal_width  petal_length  petal_width species  sepal_area
    0             51.0          3.5           1.4          0.2  setosa       17.85
    1             49.0          3.0           1.4          0.2  setosa       14.70
    2             47.0          3.2           1.3          0.2  setosa       15.04
    3             46.0          3.1           1.5          0.2  setosa       14.26
    4             50.0          3.6           1.4          0.2  setosa       18.00
    

## Delete columns


```python
#Eliminar columnas
df.drop(columns=["sepal_area"], inplace=True)
print(df.head())
```

       sepal_length_mm  sepal_width  petal_length  petal_width species
    0             51.0          3.5           1.4          0.2  setosa
    1             49.0          3.0           1.4          0.2  setosa
    2             47.0          3.2           1.3          0.2  setosa
    3             46.0          3.1           1.5          0.2  setosa
    4             50.0          3.6           1.4          0.2  setosa
    

## Reorder columns


```python
#Reordenar columnas
df = df[["species", "sepal_length_mm", "sepal_width", "petal_length", "petal_width"]]
print(df.head())
```

      species  sepal_length_mm  sepal_width  petal_length  petal_width
    0  setosa             51.0          3.5           1.4          0.2
    1  setosa             49.0          3.0           1.4          0.2
    2  setosa             47.0          3.2           1.3          0.2
    3  setosa             46.0          3.1           1.5          0.2
    4  setosa             50.0          3.6           1.4          0.2
    

## Applying functions to columns with apply()


```python
#Aplicando funciones a columnas con apply()
df["sepal_length_mm"] = df["sepal_length_mm"].apply(lambda x: x / 10)
print(df.head())
```

      species  sepal_length_mm  sepal_width  petal_length  petal_width
    0  setosa              5.1          3.5           1.4          0.2
    1  setosa              4.9          3.0           1.4          0.2
    2  setosa              4.7          3.2           1.3          0.2
    3  setosa              4.6          3.1           1.5          0.2
    4  setosa              5.0          3.6           1.4          0.2
    
