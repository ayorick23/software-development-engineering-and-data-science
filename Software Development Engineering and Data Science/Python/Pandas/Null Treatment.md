# Null Treatment


```python
import pandas as pd
import numpy as np
```

## Create a DataFrame with null values


```python
#Crear un DataFrame con valores nulos
data = {
    "Name": ["Dereck", "Ale", "Cris", "Eli", np.nan],
    "Age": [25, np.nan, 35, 55, 45],
    "Salary": [3000, 4000, np.nan, 5000, 6000]
}
df = pd.DataFrame(data)

print(df)
```

         Name   Age  Salary
    0  Dereck  25.0  3000.0
    1     Ale   NaN  4000.0
    2    Cris  35.0     NaN
    3     Eli  55.0  5000.0
    4     NaN  45.0  6000.0
    

## Identify null values


```python
#Identficar valores nulos
print(df.isnull())
```

        Name    Age  Salary
    0  False  False   False
    1  False   True   False
    2  False  False    True
    3  False  False   False
    4   True  False   False
    

## Count null values per column


```python
#Contar valores nulos por columna
print(df.isnull().sum())
```

    Name      1
    Age       1
    Salary    1
    dtype: int64
    

## Delete rows with null values


```python
#Eliminar filas con valores nulos
print(df.dropna())
```

         Name   Age  Salary
    0  Dereck  25.0  3000.0
    3     Eli  55.0  5000.0
    

## Delete columns with null values


```python
#Eliminar columnas con valores nulos
print(df.dropna(axis=1))
```

    Empty DataFrame
    Columns: []
    Index: [0, 1, 2, 3, 4]
    

## Fill null values withh a specific value


```python
#Rellenar valores nulos con un valor específico
print(df.fillna(0))
```

         Name   Age  Salary
    0  Dereck  25.0  3000.0
    1     Ale   0.0  4000.0
    2    Cris  35.0     0.0
    3     Eli  55.0  5000.0
    4       0  45.0  6000.0
    

## Fill null values with the column mean


```python
#Rellenar valores nulos con la media de la columna
df["Age"].fillna(df["Age"].mean(), inplace=True)
print(df)
```

         Name   Age  Salary
    0  Dereck  25.0  3000.0
    1     Ale  40.0  4000.0
    2    Cris  35.0     NaN
    3     Eli  55.0  5000.0
    4     NaN  45.0  6000.0
    

    C:\Users\Dereck\AppData\Local\Temp\ipykernel_19004\3509231545.py:2: FutureWarning: A value is trying to be set on a copy of a DataFrame or Series through chained assignment using an inplace method.
    The behavior will change in pandas 3.0. This inplace method will never work because the intermediate object on which we are setting values always behaves as a copy.
    
    For example, when doing 'df[col].method(value, inplace=True)', try using 'df.method({col: value}, inplace=True)' or df[col] = df[col].method(value) instead, to perform the operation inplace on the original object.
    
    
      df["Age"].fillna(df["Age"].mean(), inplace=True)
    

## Fill null values with forward fill (ffill)


```python
#Rellenar valores nulos con forward fill (ffill)
print(df.fillna(method="ffill"))
```

         Name   Age  Salary
    0  Dereck  25.0  3000.0
    1     Ale  40.0  4000.0
    2    Cris  35.0  4000.0
    3     Eli  55.0  5000.0
    4     Eli  45.0  6000.0
    

    C:\Users\Dereck\AppData\Local\Temp\ipykernel_19004\1278551488.py:2: FutureWarning: DataFrame.fillna with 'method' is deprecated and will raise in a future version. Use obj.ffill() or obj.bfill() instead.
      print(df.fillna(method="ffill"))
    

## Fill null values with backward fill (bfill)


```python
#Rellenar valores nulos con backward fill (bfill)
print(df.fillna(method="bfill"))
```

         Name   Age  Salary
    0  Dereck  25.0  3000.0
    1     Ale  40.0  4000.0
    2    Cris  35.0  5000.0
    3     Eli  55.0  5000.0
    4     NaN  45.0  6000.0
    

    C:\Users\Dereck\AppData\Local\Temp\ipykernel_19004\3595070711.py:2: FutureWarning: DataFrame.fillna with 'method' is deprecated and will raise in a future version. Use obj.ffill() or obj.bfill() instead.
      print(df.fillna(method="bfill"))
    
