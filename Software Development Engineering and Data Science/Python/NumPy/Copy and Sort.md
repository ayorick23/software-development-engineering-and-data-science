# Copying and Sorting

## Copiar y Ordenar

## Copying Arrays

Existen diferentes tipos de copias en **NumPy**, la más común es la copia superficial (*shallow copy*) y la copia profunda (*deep copy*).

- **Copia superficial (*shallow copy*)**:
En este caso, se crea una nueva matriz, pero los elementos son referencias a los objetos de la matriz original. Si los elementos son mutables (por ejemplo, listas), las modificaciones en esos elementos afectarán a ambas matrices.

- **Copia profunda (*deep copy*)**:
En este caso, se crea una nueva matriz y se copian los datos de la matriz original, creando nuevos objetos para cada elemento. Las modificaciones en la copia profunda no afectarán a la matriz original, ni viceversa.


```python
import numpy as np
```

### Shallow Copy

Una copia superficial de un array, también conocida como *vista*, comparte los datos subyacentes con el array original. Esto significa que cualquier cambio realizado en la copia superficial también se reflejará en el array original, y viceversa. Se crea una copia superficial utilizando la función ``view()`` o asignando el array a una nueva variable.


```python
#Copia superficial
arr1 = np.array([1, 2, 3])
copia_superficial = arr1.view()

copia_superficial[0] = 5  # Modifica la vista
print(arr1)               # El array original también cambia
```

    [5 2 3]
    

### Deep Copy

Una copia profunda de un array se crea utilizando la función ``np.copy()``. Esta función crea una nueva matriz con datos independientes, lo que significa que las modificaciones en la copia no afectarán a la matriz original, y viceversa.


```python
#Copia profunda
arr2 = np.array([1, 2, 3, 4, 5])
copia_profunda = np.copy(arr2)

copia_profunda[0] = 10  # Modifica la copia profunda
print(arr2)             # El array original no cambia
```

    [1 2 3 4 5]
    

## Sorting Arrays

El ordenamiento de arrays se realiza principalmente con la función ``np.sort()``. Esta función permite ordenar arrays en orden ascendente o descendente, y puede aplicarse a arrays unidimensionales y multidimensionales, especificando el eje de ordenación.

**NOTA:** Devuelve una nueva matriz ordenada, dejando la matriz original sin cambios y requiere más memoria porque crea una copia de la matriz ordenada.

### Sorting one-dimensional arrays

La función ``np.sort()`` ordena los elementos de un array unidimensional en orden ascendente por defecto.


```python
#Ordenamiento de arrays unidimensionales
arr3 = np.array([3, 1, 4, 1, 5, 9, 2, 6])
ordenado = np.sort(arr3)
print(ordenado)
```

    [1 1 2 3 4 5 6 9]
    


```python
#Orden descendente
ordenado_descendente = ordenado[::-1]
print(ordenado_descendente)
print(arr3) # El array original no cambia
```

    [9 6 5 4 3 2 1 1]
    [3 1 4 1 5 9 2 6]
    

### Sorting multidimensional arrays

``np.sort()`` también puede ordenar arrays multidimensionales. El parámetro ``axis`` especifica el eje a lo largo del cual se realizará la ordenación.


```python
#Ordenamiento de arrays bidimensionales
#Ordenamiento por columnas
arr4 = np.array([[3, 2], [1, 4]])
ordenado_axis0 = np.sort(arr4, axis=0)
print(ordenado_axis0)
```

    [[1 2]
     [3 4]]
    


```python
#Ordenamiento por filas
ordenado_axis1 = np.sort(arr4, axis=1)
print(ordenado_axis1)
```

    [[2 3]
     [1 4]]
    

### ``np.ndarray.sort()``

Es un método que se llama directamente sobre un objeto ``ndarray``. Realiza la ordenación en el lugar, modificando la matriz original y no devuelve ningún valor (devuelve ``None``). Es más eficiente en términos de memoria cuando se trabaja con matrices grandes, ya que no crea una copia.


```python
#Ordenamiento usando np.ndarray.sort()
arr5 = np.array([3, 1, 4, 1, 5, 9, 2, 6])
arr5.sort()  # Modifica arr directamente
print(arr5)
```

    [1 1 2 3 4 5 6 9]
    
