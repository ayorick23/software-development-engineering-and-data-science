# Inspecting Arrays

## Inspeccionando Arrays

En **NumPy**, verificar la forma, dimensión y tipo de datos de un array es crucial para la manipulación y procesamiento eficiente de datos, especialmente en cálculos numéricos y análisis de datos. Estos atributos permiten entender la estructura del array, realizar operaciones de broadcasting y asegurar que las operaciones se realizan con tipos de datos compatibles, evitando errores y optimizando el rendimiento.


```python
import numpy as np
```

## Example array


```python
#Array de ejemplo
arr = np.array([1, 2, 3, 4, 5])
```

## Array dimensions

El atributo ``shape`` de un array devuelve una tupla que describe las dimensiones del array.


```python
#Dimensiones del array
print(arr.shape)
```

    (5,)
    

## Length of array

La función ``len()`` devuelve la longitud de la primera dimensión de un array NumPy, es decir, el número de elementos en esa dimensión.


```python
#Longitud del array
print(len(arr))
```

    5
    

## Number of array dimensions

``ndim`` es un atributo de los arreglos (*objetos ndarray*) que indica el número de dimensiones de dicho arreglo. Es decir, te dice cuántos ejes o dimensiones tiene la matriz.


```python
#Número de dimensiones del array
print(arr.ndim)
```

    1
    

## Number of array elements

``size`` en un array se refiere al número total de elementos presentes en el array. Se puede determinar mediante la función ``np.size()``  o el atributo ``ndarray.size``.


```python
#Número de elementos del array
print(arr.size)
print(np.size(arr))
```

    5
    5
    

## Data type of array elements

``dtype`` se refiere al tipo de datos de los elementos dentro de un array. Describe cómo deben interpretarse los bytes en la memoria del array.


```python
#Tipo de datos de los elementos del array
print(arr.dtype)
```

    int64
    

## Name of data type

``dtype.name`` es un atributo de un objeto de tipo de dato (``dtype``) que devuelve una cadena que representa el nombre del tipo de dato. Este nombre proporciona un identificador legible para el tipo de dato almacenado en una matriz de NumPy.


```python
#Nombre del tipo de dato
print(arr.dtype.name)
```

    int64
    

## Convert an array to a different type

El método ``astype()`` se utiliza para convertir un array a un tipo de datos diferente. Permite cambiar el tipo de datos de los elementos de un array, como de enteros a flotantes, o de flotantes a cadenas, asegurando que los datos estén en el formato deseado. ``astype()`` siempre devuelve una copia del array con el nuevo tipo de datos, incluso si el tipo de datos original y el nuevo son iguales.


```python
#Convertir un array a un tipo de dato diferente
arr_float = arr.astype(np.float64)
print(arr_float)
print(arr_float.dtype)
```

    [1. 2. 3. 4. 5.]
    float64
    
