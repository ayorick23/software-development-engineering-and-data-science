# Statistical Clustering


```python
import pandas as pd
```

## Load a sample dataset


```python
#Cargar un dataset de ejemplo
df = pd.read_csv('https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv')
```

## Obtain general descriptive statistics


```python
#Obtener estadísticas descriptivas generales
print(df.describe())
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
    

# Count unique values per category


```python
#Contar valores únicos por categoría
print(df['species'].value_counts())
```

    species
    setosa        50
    versicolor    50
    virginica     50
    Name: count, dtype: int64
    

## Group by a column and get the mean


```python
#Agrupar por una columna y obtener la media
print(df.groupby("species")["sepal_length"].mean())
```

    species
    setosa        5.006
    versicolor    5.936
    virginica     6.588
    Name: sepal_length, dtype: float64
    

## Group and get multiple statistics


```python
#Agrupar y obtener múltiples estadísticas
print(df.groupby("species")["sepal_length"].agg(["mean", "max", "min", "std"]))
```

                 mean  max  min       std
    species                              
    setosa      5.006  5.8  4.3  0.352490
    versicolor  5.936  7.0  4.9  0.516171
    virginica   6.588  7.9  4.9  0.635880
    

## Group by multiple columns


```python
#Agrupar por múltiples columnas
print(df.groupby(["species", "sepal_width"])["sepal_length"].mean())
```

    species     sepal_width
    setosa      2.3            4.500000
                2.9            4.400000
                3.0            4.700000
                3.1            4.800000
                3.2            4.680000
                3.3            5.050000
                3.4            5.033333
                3.5            5.150000
                3.6            4.833333
                3.7            5.266667
                3.8            5.250000
                3.9            5.400000
                4.0            5.800000
                4.1            5.200000
                4.2            5.500000
                4.4            5.700000
    versicolor  2.0            5.000000
                2.2            6.100000
                2.3            5.600000
                2.4            5.300000
                2.5            5.625000
                2.6            5.666667
                2.7            5.680000
                2.8            6.150000
                2.9            6.085714
                3.0            5.950000
                3.1            6.766667
                3.2            6.433333
                3.3            6.300000
                3.4            6.000000
    virginica   2.2            6.000000
                2.5            5.900000
                2.6            6.900000
                2.7            6.075000
                2.8            6.475000
                2.9            6.800000
                3.0            6.716667
                3.1            6.725000
                3.2            6.760000
                3.3            6.566667
                3.4            6.250000
                3.6            7.200000
                3.8            7.800000
    Name: sepal_length, dtype: float64
    

## Applying custom functions with groupby


```python
#Aplicar funciones personalizadas con groupby
def rango(x):
    return x.max() - x.min()
print(df.groupby("species")["sepal_length"].agg(rango))
```

    species
    setosa        1.5
    versicolor    2.1
    virginica     3.0
    Name: sepal_length, dtype: float64
    
