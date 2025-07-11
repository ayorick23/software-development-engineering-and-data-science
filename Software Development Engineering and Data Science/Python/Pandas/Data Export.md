# Data Export


```python
import pandas as pd
```

## Create a example DataFrame


```python
#Crear un DataFrame de ejemplo
data = {
    "Name": ["Dereck", "Ale", "Cris", "Eli"],
    "Age": [25, 35, 55, 45],
    "Salary": [3000, 4000, 5000, 6000]
}
df = pd.DataFrame(data)

print(df)
```

         Name  Age  Salary
    0  Dereck   25    3000
    1     Ale   35    4000
    2    Cris   55    5000
    3     Eli   45    6000
    

## Export to CSV


```python
#Exportar a CSV
df.to_csv("data2.csv", index=False)
```

## Export to Excel


```python
#Exportar a Excel
df.to_excel("data2.xlsx", index=False)
```

## Export to JSON


```python
#Exportar a JSON
df.to_json("data2.json", orient="records", indent=4)
```

## Export to parquet format (for large volumes of data)


```python
#Exportar a format parquet (para grandes volúmenes de datos)
df.to_parquet("data2.parquet", engine="pyarrow")
```

## Export to SQLite


```python
import sqlite3

# Conectar a la base de datos (creará el archivo si no existe)
conn = sqlite3.connect("datos2.db")

# Exportar el DataFrame a SQLite
df.to_sql("empleados", conn, if_exists="replace", index=False)

# Cerrar la conexión
conn.close()
```

## Export to MySQL


```python
from sqlalchemy import create_engine

# Crear la conexión (reemplaza los valores según tu configuración)
engine = create_engine("mysql+pymysql://usuario:contraseña@localhost/nombre_base_datos")

# Exportar el DataFrame a MySQL
df.to_sql("empleados", con=engine, if_exists="replace", index=False)
```
