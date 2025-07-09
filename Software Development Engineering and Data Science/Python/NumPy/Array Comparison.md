# Array Comparison

## Comparación de Arrays

En **NumPy**, la comparación de arrays se realiza elemento por elemento utilizando operadores de comparación como ``==``, ``!=``, ``>``, ``<``, ``>=``, o ``<=``. Estas comparaciones devuelven un array booleano con el resultado de cada comparación individual. Además, existen funciones específicas como ``np.array_equal()`` para comparar la igualdad de arrays y ``np.allclose()`` para comparar con una tolerancia.


```python
import numpy as np
```

## Example arrays


```python
#Arrays de ejemplo
array1 = np.array([1, 2, 3])
array2 = np.array([1, 2, 3])
array3 = np.array([1, 4, 3])
```

## Element-by-element comparison

Al utilizar operadores de comparación directamente con arrays, la comparación se realiza entre elementos correspondientes de los arrays.


```python
#Comparación elemento por elemento
comparacion_igual = array1 == array3
comparacion_mayor = array1 > array3
print(comparacion_igual)
print(comparacion_mayor)
```

    [ True False  True]
    [False False False]
    

## Comparison functions

- ``np.array_equal(arr1, arr2)``:
Devuelve True si dos arrays tienen la misma forma y elementos, ``False`` en caso contrario. Considera ``NaN`` como iguales si ``equal_nan=True``.


```python
#Comparación de arrays completos
print(np.array_equal(array1, array2))
print(np.array_equal(array1, array3))
```

    True
    False
    

- ``np.all(comparacion)``: Devuelve ``True`` si todos los elementos de comparacion son ``True``, ``False`` en caso contrario.


```python
comparacion = array1 == array2
print(np.all(comparacion))
print(np.any(comparacion))
```

    True
    True
    

- ``np.any(comparacion)``: Devuelve ``True`` si al menos un elemento de comparacion es ``True``, ``False`` en caso contrario. 


```python
comparacion_distinta = array1 != array3
print(np.all(comparacion_distinta))
print(np.any(comparacion_distinta))
```

    False
    True
    
