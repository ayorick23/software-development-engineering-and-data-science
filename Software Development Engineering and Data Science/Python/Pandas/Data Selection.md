# Data Selection

## Selección de Datos

En **Pandas**, la selección de datos se refiere a la extracción de subconjuntos específicos de datos de un DataFrame. Esto se puede hacer utilizando etiquetas, posiciones o condiciones. Pandas ofrece varios métodos para la selección, incluyendo el uso de corchetes `[]`, `loc`, `iloc`, `at` y `iat`.


```python
import pandas as pd
```

## Creating a sample dataset


```python
#Creando un dataset de ejemplo
df = pd.DataFrame({
    'nombre': ['Dereck', 'Carlos', 'Alejandra', 'Cristina', 'Elizabeth', 'Henry', 'Sofía', 'Steve'],
    'edad': [25, 30, 22, 35, 28, 40, 26, 32],
    'ciudad': ['Madrid', 'Barcelona', 'Valencia', 'Sevilla', 'Bilbao', 'Madrid', 'Barcelona', 'Valencia'],
    'salario': [35000, 45000, 28000, 55000, 38000, 60000, 32000, 48000],
    'departamento': ['IT', 'Ventas', 'Marketing', 'IT', 'RRHH', 'Ventas', 'Marketing', 'IT']
})

#Establecer índice personalizado para mejor demostraciones con loc
df.index = ['emp_001', 'emp_002', 'emp_003', 'emp_004', 'emp_005', 'emp_006', 'emp_007', 'emp_008']
```

## Select a column

Aquí el método `head()` muestra las primeras filas de la columna *"species"* en el DataFrame.


```python
#Seleccionar una columna
print(df['nombre'].head())
```

    emp_001       Dereck
    emp_002       Carlos
    emp_003    Alejandra
    emp_004     Cristina
    emp_005    Elizabeth
    Name: nombre, dtype: object
    

## Select multiple columns


```python
#Seleccionar varias columnas
print(df[['nombre', 'departamento']].head())
```

                nombre departamento
    emp_001     Dereck           IT
    emp_002     Carlos       Ventas
    emp_003  Alejandra    Marketing
    emp_004   Cristina           IT
    emp_005  Elizabeth         RRHH
    

## `loc`

`loc` selecciona los datos utilizando nombres de filas y columnas (etiquetas).

### Select rows by index with `loc`

Para seleccionar una sola fila, utilizamos la etiqueta de la fila que queremos recuperar, en este caso es `'emp_004'`.


```python
#Seleccionar filas por índice con loc
print(df.loc['emp_004'])
```

    nombre          Cristina
    edad                  35
    ciudad           Sevilla
    salario            55000
    departamento          IT
    Name: emp_004, dtype: object
    

### Select multiple rows with `loc`

Si queremos seleccionar varias filas que no se siguen necesariamente en orden, tenemos que pasar una lista de sus etiquetas de fila como argumento con una coma entre cada una. Esto significa que tenemos que utilizar no uno, sino dos pares de corchetes: uno para la sintaxis normal de `loc` y otro para la lista de etiquetas.


```python
#Seleccionar varias filas con loc
print(df.loc[['emp_002', 'emp_008']])
```

             nombre  edad     ciudad  salario departamento
    emp_002  Carlos    30  Barcelona    45000       Ventas
    emp_008   Steve    32   Valencia    48000           IT
    

### Select a range of rows with `loc`

Podemos seleccionar un rango de filas pasando las etiquetas de la primera y la última fila con dos puntos entre ellas.


```python
#Seleccionar un rango de filas con loc
print(df.loc['emp_002':'emp_007'])
```

                nombre  edad     ciudad  salario departamento
    emp_002     Carlos    30  Barcelona    45000       Ventas
    emp_003  Alejandra    22   Valencia    28000    Marketing
    emp_004   Cristina    35    Sevilla    55000           IT
    emp_005  Elizabeth    28     Bilbao    38000         RRHH
    emp_006      Henry    40     Madrid    60000       Ventas
    emp_007      Sofía    26  Barcelona    32000    Marketing
    

### Select specific rows and columns with `loc`

Para hacer una selección combinada de filas y columnas se coloca primero el rango de filas seguido por el rango de columnas.


```python
#Seleccionar filas y columnas específicas con loc
print(df.loc['emp_002':'emp_005', 'nombre':'ciudad'])
```

                nombre  edad     ciudad
    emp_002     Carlos    30  Barcelona
    emp_003  Alejandra    22   Valencia
    emp_004   Cristina    35    Sevilla
    emp_005  Elizabeth    28     Bilbao
    

## `iloc`

`iloc` selecciona los datos por posición (*índice entero*) en lugar de por etiqueta.

### Select rows by index with `iloc`

Se puede seleccionar una fila utilizando el número entero que representa el número de índice de la fila. No necesitamos comillas ya que estamos introduciendo un número entero y no una cadena de etiquetas como hicimos con `loc`.


```python
#Seleccionar filas por posición con iloc
print(df.iloc[0])
```

    nombre          Dereck
    edad                25
    ciudad          Madrid
    salario          35000
    departamento        IT
    Name: emp_001, dtype: object
    

### Select multiple rows with `iloc`

La selección de varias filas funciona en `iloc` como en `.loc`; introducimos los números enteros del índice de fila en una lista con corchetes.


```python
#Seleccionar varias filas con iloc
print(df.iloc[[1, 3, 5]])
```

               nombre  edad     ciudad  salario departamento
    emp_002    Carlos    30  Barcelona    45000       Ventas
    emp_004  Cristina    35    Sevilla    55000           IT
    emp_006     Henry    40     Madrid    60000       Ventas
    

### Select a range of rows with `iloc`

Para seleccionar un rango de filas, utilizamos dos puntos entre dos enteros de índice de fila especificados.


```python
#Seleccionar un rango de filas con iloc
print(df.iloc[1:4])
```

                nombre  edad     ciudad  salario departamento
    emp_002     Carlos    30  Barcelona    45000       Ventas
    emp_003  Alejandra    22   Valencia    28000    Marketing
    emp_004   Cristina    35    Sevilla    55000           IT
    

**NOTA:** `iloc` no es inclusivo para la selección de trozos, nuestra salida incluirá todas las filas hasta la última antes de ésta. Por lo tanto, devuelve solo la segunda, tercer y cuarta fila.

### Select specific rows and columns with `iloc`

Se maneja igual que con `loc`. Se colocan los rangos de las filas y luego los rangos de las columnas.


```python
#Seleccionar filas y columnas con iloc
print(df.iloc[0:5, 1:-1]) #primeras 5 filas, desde la segunda hasta la penúltima columna
```

             edad     ciudad  salario
    emp_001    25     Madrid    35000
    emp_002    30  Barcelona    45000
    emp_003    22   Valencia    28000
    emp_004    35    Sevilla    55000
    emp_005    28     Bilbao    38000
    

## `at`

`at` se utiliza para acceder a un único valor en un DataFrame utilizando etiquetas de fila y columna. Es similar a `loc`, pero está optimizado para acceder a un solo valor y, por lo tanto, es más rápido.


```python
#Seleccionar un valor específico con at (mas rápido para un solo valor)
print(df.at['emp_001', 'nombre'])
```

    Dereck
    

## `iat`

`iat` es un atributo que se utiliza para acceder a un valor escalar en un DataFrame por su posición entera (*índice*). Es similar a `iloc`, pero `iat` está optimizado para acceder a un solo valor y es más rápido que `iloc` en esos casos.


```python
#Seleccionar un valor específico con iat (mas rápido para un solo valor)
print(df.iat[0, -1])
```

    IT
    
