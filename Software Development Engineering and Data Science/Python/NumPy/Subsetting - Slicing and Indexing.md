# Subsetting: Slicing and Indexing

## Subconjuntos: Segmentación e Indexación

## Subsetting

En **NumPy**, la creación de subconjuntos de matrices se realiza utilizando la indexación y el slicing. La indexación permite acceder a elementos individuales, mientras que el slicing permite extraer rangos de elementos de forma continua. Se pueden utilizar corchetes ``[]`` para especificar los índices o rangos de índices que se desean seleccionar, separando los índices de diferentes dimensiones con comas para arrays multidimensionales.


```python
import numpy as np
```

### Indexing

- **Indexación simple:** Se accede a elementos individuales usando su posición dentro del array. La numeración en Python, y por ende en NumPy, comienza desde 0.


```python
arr = np.array([10, 20, 30, 40, 50])
print(arr[1])  # Imprime 20 (el segundo elemento)
```

    20
    

- **Indexación multidimensional:** Para arrays con más de una dimensión, se utilizan múltiples índices separados por comas.


```python
arr_2d = np.array([[1, 2, 3], [4, 5, 6]])
print(arr_2d[0, 1])  # Imprime 2 (fila 0, columna 1)
```

    2
    

### Slicing

- **Slicing básico:** Se utiliza el operador ``:`` para indicar un rango de elementos.


```python
print(arr[1:4])  # Elementos desde el índice 1 hasta el 3
```

    [20 30 40]
    

- **Slicing con inicio y fin:** Se puede especificar el índice de inicio y fin (*exclusivo*) del rango.


```python
print(arr[:3])   # Desde el principio hasta el índice 2
print(arr[2:])   # Desde el índice 2 hasta el final)
```

    [10 20 30]
    [30 40 50]
    

- **Slicing con paso:** Se puede especificar un paso para saltar elementos dentro del rango.


```python
print(arr[0:5:2])  # Imprime cada dos elementos
```

    [10 30 50]
    

- **Slicing en arrays multidimensionales:** Se puede aplicar slicing a cada dimensión por separado.


```python
arr_3d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print(arr_3d[1:, 0:2])  # Imprime todas las filas desde la 1 y columnas 0 y 1
```

    [[4 5]
     [7 8]]
    

### Boolean Indexing

- Se puede utilizar un array booleano como índice para seleccionar elementos basados en una condición.


```python
indices = arr > 20
print(arr[indices])
```

    [30 40 50]
    

- La función ``np.where()`` también puede ser útil para encontrar los índices de elementos que cumplen una condición.


```python
indices_where = np.where(arr > 20)
print(indices_where)  # Imprime los índices donde la condición es verdadera
print(arr[indices_where])
```

    (array([2, 3, 4]),)
    [30 40 50]
    
