# Input and Output (I/0)

## Entrada y Salida (E/S)

En **NumPy**, la entrada y salida (*E/S*) se refiere a la forma en que se cargan datos en las matrices (*input*) y se guardan o presentan los resultados (*output*). NumPy ofrece funciones para trabajar con archivos de texto y binarios, así como para formatear la salida de las matrices en diferentes formatos, incluyendo texto y representaciones binarias.


```python
import numpy as np
```

## Example arrays


```python
#Arrays de ejemplo
arr = np.array([1, 2, 3, 4, 5])
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
```

## Saving and Loading on disk

Los objetos ndarray se pueden guardar y cargar desde el disco con funciones que gestionan archivos binarios de NumPy con la extensión ``.npy`` como `np.save()` o ``.npz`` con la función `np.savez()`.

Los archivos ``.npy`` y ``.npz`` almacenan datos, forma, tipo de datos y otra información necesaria para reconstruir el ndarray de manera que permita recuperar la matriz correctamente, incluso cuando el archivo está en otra máquina con una arquitectura diferente.

### `np.save`

La función ``np.save()`` guarda una matriz de NumPy en un archivo binario con extensión `.npy`. Se utiliza para almacenar una sola matriz.


```python
#Guarda el array con el nombre 'my_array.npy'
np.save('mi_array.npy', arr)
```

### `np.savez`

La función ``np.savez()`` permite guardar múltiples arrays de NumPy en un solo archivo con extensión ``.npz``. Este archivo es un tipo de archivo comprimido que contiene los arrays guardados. Al guardar con ``np.savez()``, los arrays se guardan con nombres específicos, ya sea por defecto o usando nombres de palabras clave.


```python
#Guardar los arrays en un archivo llamado 'mis_datos.npz'
np.savez('mis_datos.npz', primero = arr1, segundo = arr2)
```

### `np.load`

La función ``np.load()`` se utiliza para cargar matrices u objetos encurtidos desde ``.npy``, ``.npz`` o archivos encurtidos.


```python
#Carga un archivo .npy
array = np.load('mi_array.npy')
print(array)

#Carga un archivo .npz
data = np.load('mis_datos.npz')
array1 = data['primero']
array2 = data['segundo']
print(array1)
print(array2)
```

    [1 2 3 4 5]
    [1 2 3]
    [4 5 6]
    

## Saving and Loading text files

Para guardar y cargar archivos de texto en NumPy, se utilizan las funciones ``np.savetxt()`` y ``np.loadtxt()`` respectivamente. ``savetxt()`` permite guardar un array de NumPy en un archivo de texto, mientras que ``loadtxt()`` carga datos de un archivo de texto en un array de NumPy.

### `np.savetxt`

La función ``np.savetxt()`` se utiliza para guardar un array de NumPy en un archivo de texto.


```python
#Guarda un array en un archivo de texto
np.savetxt("mi_archivo.txt", arr, delimiter=',')
```

### `np.loadtxt`

La función ``np.loadtxt()`` se utiliza para leer datos de un archivo de texto y cargarlos en un array de NumPy.


```python
#Carga un archivo de texto
data = np.loadtxt("mi_archivo.txt", delimiter=',')
print(data)
```

    [1. 2. 3. 4. 5.]
    

### `np.genfromtext`

Esta función se utiliza para cargar datos de un archivo de texto a un array de NumPy. Es más flexible que ``np.loadtxt()`` ya que admite valores faltantes y diferentes tipos de datos.


```python
#Datos de ejemplo de formato string
data = """
# Este es un comentario
1,2,3
4,,6
7,8,9
"""

#Carga de datos usando genfromtxt()
array = np.genfromtxt(data.splitlines(), delimiter=',', comments='#')
print(array)
```

    [[ 1.  2.  3.]
     [ 4. nan  6.]
     [ 7.  8.  9.]]
    
