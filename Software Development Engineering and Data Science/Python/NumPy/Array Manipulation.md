# Array Manipulation

## Manipulación de Arrays

En **NumPy**, la manipulación de arrays es fundamental para el análisis numérico y el procesamiento de datos. Permite crear, acceder, modificar y transformar arrays de manera eficiente. Las operaciones comunes incluyen cambiar la forma del array, combinar y dividir arrays, e invertir o rotar elementos, entre otros.


```python
import numpy as np
```

## Example arrays


```python
#Datos de ejemplo
arr1 = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
arr2 = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
print(f"Array 1 original:\n{arr1}")
print(f"Array 2 original:\n{arr2}")
```

    Array 1 original:
    [[1 2 3]
     [4 5 6]
     [7 8 9]]
    Array 2 original:
    [[[1 2]
      [3 4]]
    
     [[5 6]
      [7 8]]]
    

## Transpose an array

Para transponer matrices en NumPy, están disponibles los siguientes métodos:

### ``T``

Esta es la forma más común y práctica de transponer una matriz 2D. Devuelve directamente una vista transpuesta de la matriz.


```python
#Transponer un array con .T
transpuesta = arr1.T
print(f"Transpuesta:\n{transpuesta}")
```

    Transpuesta:
    [[1 4 7]
     [2 5 8]
     [3 6 9]]
    

### ``transpose()``

Esta función proporciona más control, especialmente para matrices de mayor dimensión, al permitirle especificar el orden de los ejes.


```python
#Transponer un array con np.transpose
transpuesta_por_defecto = np.transpose(arr2)
print("Transpuesta:\n", transpuesta_por_defecto)

#Transponer especificando el orden de los ejes
transpuesta_ejes = np.transpose(arr2, axes=(2, 1, 0))
print("Tranpuesta con ejes específicos:\n", transpuesta_ejes)
```

    Transpuesta:
     [[[1 5]
      [3 7]]
    
     [[2 6]
      [4 8]]]
    Tranpuesta con ejes específicos:
     [[[1 5]
      [3 7]]
    
     [[2 6]
      [4 8]]]
    

## Reshape an array

Para cambiar la forma de un array se utiliza la función ``reshape()``. Esta función permite reorganizar los elementos del array en una nueva forma sin alterar los datos originales. Es importante que la nueva forma tenga el mismo número total de elementos que la original.


```python
#Cambiar la forma de un array
arr_reshape = arr1.reshape(1, 9) #convertir en un array de 1 fila y 9 columnas
print(f"Array después del reshape:\n{arr_reshape}")
```

    Array después del reshape:
    [[1 2 3 4 5 6 7 8 9]]
    

## Flatten an array

Aplanar un array significa convertir un array multidimensional en un array unidimensional. Existen dos métodos principales para hacer esto: ``ndarray.flatten()`` y ``np.ravel()``.

### `ndarray.flatten()`

El método ``flatten()`` crea una copia del array aplanado. Esto significa que cualquier cambio realizado en el array aplanado no afectará al array original.


```python
#Aplanar un array con flatten()
arr_flatten = arr1.flatten()
print(f"Array aplanado con flatten:\n{arr_flatten}")
```

    Array aplanado con flatten:
    [1 2 3 4 5 6 7 8 9]
    

### `np.ravel()`

El método ``ravel()`` intenta devolver una vista del array original, lo que significa que los cambios en el array aplanado se reflejarán en el array original. Si no es posible crear una vista (*por ejemplo, si los elementos no son contiguos en memoria*), ``ravel()`` creará una copia.


```python
#Aplanar un array con np.ravel
arr_ravel = np.ravel(arr2)
print(f"Array aplanado con ravel:\n{arr_ravel}")
```

    Array aplanado con ravel:
    [1 2 3 4 5 6 7 8]
    

## Concatenate arrays

La combinación de arrays se realiza principalmente con las funciones `concatenate()`, `stack()`, `hstack()`, `vstack()`, `dstack()`, y `column_stack()`. Estas funciones permiten unir arrays vertical, horizontal u otros ejes. Además, concatenate es versátil y puede manejar matrices de diferentes dimensiones y unirlas a lo largo de diferentes ejes.

### `concatenate()`

Esta función une los arrays a lo largo de un eje especificado. El eje predeterminado es `0`, que une los arrays verticalmente (*filas*). Para unir arrays horizontalmente (*columnas*), se especifica `axis=1`.


```python
#Concatenar arrays
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6]])

arr_concatenate_filas = np.concatenate((a, b), axis=0)  # Concatenar por filas
print(f"Arrays concatenados por filas:\n{arr_concatenate_filas}")

arr_concatenate_columnas = np.concatenate((a, b.T), axis=1)  # Concatenar por columnas
print(f"Arrays concatenados por columnas:\n{arr_concatenate_columnas}")
```

    Arrays concatenados por filas:
    [[1 2]
     [3 4]
     [5 6]]
    Arrays concatenados por columnas:
    [[1 2 5]
     [3 4 6]]
    

### `stack()`

Esta función apila los arrays a lo largo de un nuevo eje. Por defecto, crea un nuevo eje en la primera posición (``axis=0``).


```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

#Apilar verticalmente
arr_stack_vertical = np.stack((a, b), axis=0)
print(f"Apliado verticalmente:\n{arr_stack_vertical}")

#Apilar horizontalmente
arr_stack_horizontal = np.stack((a, b), axis=1)
print(f"Apliado horizontalmente:\n{arr_stack_horizontal}")
```

    Apliado verticalmente:
    [[1 2 3]
     [4 5 6]]
    Apliado horizontalmente:
    [[1 4]
     [2 5]
     [3 6]]
    

### `hstack()`

Es una función corta para ``np.concatenate`` con ``axis=1``, apilando los arrays horizontalmente.


```python
arr_hstack = np.hstack((a, b))
print(f"Apilado horizontal con hstack:\n{arr_hstack}")
```

    Apilado horizontal con hstack:
    [1 2 3 4 5 6]
    

### `vstack()`

Es una función corta para `np.concatenate` con `axis=0`, apilando los arrays verticalmente.


```python
arr_vstack = np.vstack((a, b))
print(f"Apilado vertical con vstack:\n{arr_vstack}")
```

    Apilado vertical con vstack:
    [[1 2 3]
     [4 5 6]]
    

### `dstack()`

Apila los arrays a lo largo del tercer eje (*z*), creando arrays 3D.


```python

arr_dstack = np.dstack((a, b))
print(f"Apilado en profundidad con dstack:\n{arr_dstack}")
```

    Apilado en profundidad con dstack:
    [[[1 4]
      [2 5]
      [3 6]]]
    

### `column_stack()`

Apila los arrays como columnas. Para arrays 1D, crea una matriz 2D donde cada array 1D es una columna. Para arrays 2D, actúa como `hstack`.


```python
arr3 = np.array([7, 8])

arr_columnstack = np.column_stack((arr3, arr3*2))
print("\nApilamiento por columnas:\n", arr_columnstack)
```

    
    Apilamiento por columnas:
     [[ 7 14]
     [ 8 16]]
    

### `row_stack()`

Apila los arrays como filas. Para arrays 1D, crea una matriz 2D donde cada array 1D es una fila. Para arrays 2D, actúa como `vstack`.


```python
arr_rowstack = np.row_stack((arr3, arr3*2))
print("\nApilamiento por filas:\n", arr_rowstack)
```

    
    Apilamiento por filas:
     [[ 7  8]
     [14 16]]
    

    C:\Users\Dereck\AppData\Local\Temp\ipykernel_12680\1956380926.py:1: DeprecationWarning: `row_stack` alias is deprecated. Use `np.vstack` directly.
      arr_rowstack = np.row_stack((arr3, arr3*2))
    

## Split an array

La división de arrays en sub-arrays se realiza principalmente con las funciones ``split()``, ``array_split()``, ``hsplit()``, ``vsplit()`` y ``dsplit()``.

### `split()`

Divide un array en sub-arrays de igual tamaño, este requiere que el número de divisiones sea compatible con la longitud del array. Devuelve una lista de sub-arrays.


```python
#Dividir un array
arr_split = np.split(arr1, 3) #Divide el array en 3 partes
print(arr_split)
```

    [array([[1, 2, 3]]), array([[4, 5, 6]]), array([[7, 8, 9]])]
    

### `array_split()`

Similar a ``split()``, pero permite divisiones desiguales. Si la longitud del array no es divisible por el número de divisiones, crea sub-arrays con tamaños ligeramente diferentes.


```python
arr_array_split = np.array_split(arr1, 2) #Divide el array en 2 partes, no necesariamente iguales
print(arr_array_split)
```

    [array([[1, 2, 3],
           [4, 5, 6]]), array([[7, 8, 9]])]
    

### `hsplit()`

Divide un array en sub-arrays horizontalmente (*por columnas*). Equivalente a ``split(arr, indices_or_sections, axis=1)``.


```python
arr_hsplit = np.hsplit(arr1, 3) #Divide el array horizontalmente en 3 partes
print(arr_hsplit)
```

    [array([[1],
           [4],
           [7]]), array([[2],
           [5],
           [8]]), array([[3],
           [6],
           [9]])]
    

### `vsplit()`

Divide un array en sub-arrays verticalmente (*por filas*). Equivalente a ``split(arr, indices_or_sections, axis=0)``.


```python
arr_vsplit = np.vsplit(arr1, 3) #Divide el array verticalmente en 3 partes
print(arr_vsplit)
```

    [array([[1, 2, 3]]), array([[4, 5, 6]]), array([[7, 8, 9]])]
    

### `dsplit()`

Divide un array en sub-arrays a lo largo del tercer eje (*profundidad*). Solo aplicable a arrays de tres o más dimensiones.


```python
arr_dsplit = np.dsplit(arr2, 2) #Divide el array tridimensional en 2 partes
print(arr_dsplit)
```

    [array([[[1],
            [3]],
    
           [[5],
            [7]]]), array([[[2],
            [4]],
    
           [[6],
            [8]]])]
    

## Adding and removing elements in an array

Agregar y eliminar elementos de un array se realiza principalmente a través de las funciones `append()`, `insert()` y `delete()`. `append()` añade elementos al final del array, `insert()` agrega elementos en una posición específica y `delete()` elimina elementos basados en sus índices. Es importante recordar que estas funciones suelen devolver una nueva matriz, dejando la original intacta.

### `append()`

Añade elementos al final del array.


```python
arr = np.array([1, 2, 3])
arr_append = np.append(arr, [4, 5])
print(arr_append)
```

    [1 2 3 4 5]
    

### `insert()`

Inserta elementos en una posición específica dentro del array. Requiere especificar el índice y el valor a insertar.


```python
arr_insert = np.insert(arr, 1, [7, 8]) # Inserta 7 y 8 en la posición 1
print(arr_insert)
```

    [1 7 8 2 3]
    

### `delete()`

Elimina elementos del array según su índice. Se le pasa el array y una lista de índices a eliminar.


```python
arr_delete = np.delete(arr_insert, [0, 2])  # Elimina elementos en los índices 0 y 2
print(arr_delete)
```

    [7 2 3]
    
