# Filters and Conditions

## Filtros y Condiciones

En **Pandas**, los filtros y condiciones se utilizan para seleccionar filas de un DataFrame basadas en ciertos criterios. Existen varias formas de lograr esto, incluyendo indexación booleana, el método `query()`, y funciones como `loc`. Se pueden combinar múltiples condiciones utilizando operadores lógicos como `&` (*AND*), `|` (*OR*), y `~` (*NOT*).


```python
import pandas as pd
```

## Load a sample dataset


```python
#Cargar dataset de ejemplo
df = pd.read_csv("https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv")
```

## Filter rows based on a condition

La forma más básica de filtrar es usando una máscara booleana. Se crea una serie booleana que indica si cada fila cumple la condición.


```python
#Filtrar filas según una condición
print(df[df["species"] == "setosa"].head())
```

       sepal_length  sepal_width  petal_length  petal_width species
    0           5.1          3.5           1.4          0.2  setosa
    1           4.9          3.0           1.4          0.2  setosa
    2           4.7          3.2           1.3          0.2  setosa
    3           4.6          3.1           1.5          0.2  setosa
    4           5.0          3.6           1.4          0.2  setosa
    

## Filter by multiple conditions (`&`)

El operador `&` requiere que ambas condiciones sean verdaderas para que la fila sea seleccionada.


```python
#Filtrar por múltiples condiciones (AND)
print(df[(df["species"] == "setosa") & (df["sepal_length"] > 5)].head())
```

        sepal_length  sepal_width  petal_length  petal_width species
    0            5.1          3.5           1.4          0.2  setosa
    5            5.4          3.9           1.7          0.4  setosa
    10           5.4          3.7           1.5          0.2  setosa
    14           5.8          4.0           1.2          0.2  setosa
    15           5.7          4.4           1.5          0.4  setosa
    

## Filter by multiple conditions (`|`)

`|` requiere que al menos una de las condiciones sea verdadera.


```python
#Filtrar por múltiples condiciones (OR)
print(df[(df["species"] == "setosa") | (df["species"] == "versicolor")].head())
```

       sepal_length  sepal_width  petal_length  petal_width species
    0           5.1          3.5           1.4          0.2  setosa
    1           4.9          3.0           1.4          0.2  setosa
    2           4.7          3.2           1.3          0.2  setosa
    3           4.6          3.1           1.5          0.2  setosa
    4           5.0          3.6           1.4          0.2  setosa
    

## Filter with denial (`~`)

Niega la expresión booleana. Es decir, si la expresión original es verdadera, `~` la convierte en falsa, y viceversa.


```python
#Filtrar con negación (~)
print(df[~(df["species"] == "setosa")].head())
```

        sepal_length  sepal_width  petal_length  petal_width     species
    50           7.0          3.2           4.7          1.4  versicolor
    51           6.4          3.2           4.5          1.5  versicolor
    52           6.9          3.1           4.9          1.5  versicolor
    53           5.5          2.3           4.0          1.3  versicolor
    54           6.5          2.8           4.6          1.5  versicolor
    

## Filter with `loc`

Devuelve filas basándose en una expresión condicional. Filtra todas las filas según cumplan o no una determinada condición y mostrar sólo las que la cumplan.


```python
#Filtrar con loc
print(df.loc[df["species"] == "setosa"].head())
```

       sepal_length  sepal_width  petal_length  petal_width species
    0           5.1          3.5           1.4          0.2  setosa
    1           4.9          3.0           1.4          0.2  setosa
    2           4.7          3.2           1.3          0.2  setosa
    3           4.6          3.1           1.5          0.2  setosa
    4           5.0          3.6           1.4          0.2  setosa
    

## Filter with `isin()`

Filtra filas donde un valor está en una lista dada.


```python
#Filtrar con isin
print(df[df["species"].isin(["setosa", "virginica"])].head())
```

       sepal_length  sepal_width  petal_length  petal_width species
    0           5.1          3.5           1.4          0.2  setosa
    1           4.9          3.0           1.4          0.2  setosa
    2           4.7          3.2           1.3          0.2  setosa
    3           4.6          3.1           1.5          0.2  setosa
    4           5.0          3.6           1.4          0.2  setosa
    

## Filter with `query()`

La función `query()` permite filtrar filas de un DataFrame utilizando una expresión de cadena como consulta. Esta función es una forma más legible y concisa de realizar filtrado basado en condiciones, similar a como se escribe una consulta SQL.


```python
#Filtrar con query()
print(df.query("species == 'setosa' and sepal_length > 5"))
```

        sepal_length  sepal_width  petal_length  petal_width species
    0            5.1          3.5           1.4          0.2  setosa
    5            5.4          3.9           1.7          0.4  setosa
    10           5.4          3.7           1.5          0.2  setosa
    14           5.8          4.0           1.2          0.2  setosa
    15           5.7          4.4           1.5          0.4  setosa
    16           5.4          3.9           1.3          0.4  setosa
    17           5.1          3.5           1.4          0.3  setosa
    18           5.7          3.8           1.7          0.3  setosa
    19           5.1          3.8           1.5          0.3  setosa
    20           5.4          3.4           1.7          0.2  setosa
    21           5.1          3.7           1.5          0.4  setosa
    23           5.1          3.3           1.7          0.5  setosa
    27           5.2          3.5           1.5          0.2  setosa
    28           5.2          3.4           1.4          0.2  setosa
    31           5.4          3.4           1.5          0.4  setosa
    32           5.2          4.1           1.5          0.1  setosa
    33           5.5          4.2           1.4          0.2  setosa
    36           5.5          3.5           1.3          0.2  setosa
    39           5.1          3.4           1.5          0.2  setosa
    44           5.1          3.8           1.9          0.4  setosa
    46           5.1          3.8           1.6          0.2  setosa
    48           5.3          3.7           1.5          0.2  setosa
    
