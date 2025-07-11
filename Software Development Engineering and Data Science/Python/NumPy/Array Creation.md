# <img width="35" height="35" src="https://img.icons8.com/?size=100&id=aR9CXyMagKIS&format=png&color=000000" alt="numpy"> Introduction to NumPy

## Introducción a NumPy

**NumPy** es una biblioteca de [[Introduction to Python#Introduction to Python|Python]] fundamental para la computación científica y el análisis de datos. Su principal función es proporcionar objetos de matriz multidimensionales (*llamados ndarray*) que permiten almacenar y manipular grandes conjuntos de datos de manera eficiente, así como una amplia gama de funciones matemáticas para operar sobre estas matrices.

# Array Creation

## Creación de Arrays

En **NumPy**, la creación de arrays se puede lograr de varias formas, siendo la más común la función `np.array()`. Esta función convierte una secuencia de Python (como una [[Lists and Tuples#Listas|lista]] o [[Lists and Tuples#Tuplas|tupla]]) en un objeto **ndarray**, que es la estructura de datos fundamental de NumPy. Además, existen funciones específicas como `np.zeros()`, `np.ones()`, `np.empty()`, `np.arange()`, y `np.linspace()` que facilitan la creación de arrays con patrones predefinidos o rangos de valores.

## Import NumPy

```python
import numpy as np
```

## Check NumPy version

Antes de comenzar, es buena práctica verificar la versión instalada de `numpy`, especialmente para asegurar la compatibilidad de funciones:

```python
#Verificar la versión de NumPy
print("Version de NumPy:", np.__version__)
```

    Version de NumPy: 2.2.3

## Create an array

La función `np.array()` es la más versátil para crear [[Clase 04 - Listas, Arreglos, Matrices y Estructuras Condicionales#Arreglos (`Array`)|arrays]] a partir de datos existentes. Recibe una secuencia (*lista, tupla, etc.*) como argumento y devuelve un array de NumPy con la misma estructura. Los ***ndarray*** son un objeto que representa una matriz o arreglo de datos homogéneos (todos del mismo [[Data Types#Tipos de Datos|tipo]]) indexados por una tupla de enteros no negativos.

```python
#Crear un array de NumPy
arr = np.array([10, 20, 30, 40, 50])
```

## Create an array from a list

Para crear un array a partir de una lista se utiliza la función ``np.array()``. Esta función toma la lista como argumento y devuelve un nuevo array de NumPy con los mismos elementos.

```python
#Crear un array a partir de una lista
lista = [1, 2, 3, 4, 5]
arr1 = np.array(lista)
print(arr1)
```

    [1 2 3 4 5]

## Create an array with specific data types

La creación de arrays con tipos de datos específicos se realiza principalmente utilizando la función ``np.array()`` y especificando el tipo deseado con el argumento [[Inspecting Arrays#Data type of array elements|dtype]].

```python
#Crear un array con un tipo de dato específico
arr2 = np.array([1.5, 2.5, 3.5, 4.5, 5.5], dtype=float)
print(arr2)
```

    [1.5 2.5 3.5 4.5 5.5]

## Create an array with a range of values

La función ``np.arange()`` crea arrays con una secuencia de números dentro de un rango especificado, similar a la función [[Built-in Functions#`range()`|range()]] de Python, pero devuelve un arreglo NumPy en lugar de una lista. Puedes definir el inicio, el fin (*exclusivo*) y el paso (*incremento*) de la secuencia.

```python
#Crear un array con un rango de valores
arr2 = np.arange(0, 10, 2) #valores de 0 a 10 (exclusivo) con paso de 2
print(f"Array con np.arange: {arr2}")
```

    Array con np.arange: [0 2 4 6 8]

## Create an array with intervals

La función ``np.linspace`` se utiliza para crear una [[Clase 04 - Listas, Arreglos, Matrices y Estructuras Condicionales#Matrices|matriz]] de números espaciados uniformemente en un intervalo específico.

```python
#Crear un array con intervalos
arr3 = np.linspace(0, 1, 5) #5 valores entre 0 y 1 (incluidos)
print(f"Array con np.linspace: {arr3}")
```

    Array con np.linspace: [0.   0.25 0.5  0.75 1.  ]

## Create an array of zeros

La función ``np.zeros()`` crea una matriz llena de ceros con la forma y tipo de datos especificados. Si no se especifica, el tipo por defecto es [[Data Types#Floats|float 64]].

```python
#Crear un array de ceros
zeros = np.zeros((3, 3))

print(f"Array de ceros:\n{zeros}")
```

    Array de ceros:
    [[0. 0. 0.]
     [0. 0. 0.]
     [0. 0. 0.]]

## Create an array of ones

La función ``np.ones()`` genera una matriz de una forma y un tipo de datos especificados, donde todos los elementos se inicializan en 1.

```python
#Crear un array de unos
ones = np.ones((2, 4))

print(f"Array de unos:\n{ones}")
```

    Array de unos:
    [[1. 1. 1. 1.]
     [1. 1. 1. 1.]]

## Create an array of constants

La función ``np.full()`` se utiliza para crear una nueva matriz de una forma y un tipo de datos específicos, rellena con un valor dado. Pide como argumentos una tupla con la forma y el valor con el que se rellenará la matriz.

```python
constants = np.full((2, 3), 5)
print(f"Array de constantes:\n{constants}")
```

    Array de constantes:
    [[5 5 5]
     [5 5 5]]

## Create an identity matrix

La matriz identidad se puede crear usando la función ``np.identity()`` o ``np.eye()``. Ambas funciones generan una matriz cuadrada donde la diagonal principal está compuesta por unos y el resto de los elementos son ceros. La diferencia principal radica en que ``np.eye()`` permite especificar un desplazamiento para la diagonal, mientras que ``np.identity()`` solo crea la diagonal principal.

```python
#Crear una matriz identidad con identity()
identity = np.identity(3)
print(f"Matriz identidad con identity():\n{identity}")
```

    Matriz identidad con identity():
    [[1. 0. 0.]
     [0. 1. 0.]
     [0. 0. 1.]]

```python
#Crear una matriz identidad con eye()
identity2 = np.eye(3)
print(f"Matriz identidad con eye():\n{identity2}")

displaced_identity = np.eye(3, k=1) # k especifica el desplazamiento
print(f"Matriz identidad desplazada:\n{displaced_identity}")
```

    Matriz identidad con eye():
    [[1. 0. 0.]
     [0. 1. 0.]
     [0. 0. 1.]]
    Matriz identidad desplazada:
    [[0. 1. 0.]
     [0. 0. 1.]
     [0. 0. 0.]]

## Create an array of random numbers betwen 0 and 1

- ``np.random.rand(d0, d1, ..., dn)``: Genera una matriz con la forma especificada (``d0, d1, ..., dn``) llena de números aleatorios distribuidos uniformemente entre 0 y 1.

```python
#Crear un array de números aleatorios
randomArr = np.random.rand(3, 3) #matriz 3x3 con valores aleatorios entre 0 y 1
print(f"Array aleatorio con np.random.rand:\n{randomArr}")
```

    Array aleatorio con np.random.rand:
    [[0.19790156 0.79390578 0.93733923]
     [0.83541731 0.87589121 0.3873544 ]
     [0.55185233 0.87390292 0.16781115]]

## Create an array of random integers

- ``np.random.randint(low, high, (size))``: Genera una matriz con números enteros aleatorios en el rango (``low, high``). ``size`` define la forma de la matriz.

```python
#Crear un array de números aleatorios enteros
randomIntArr = np.random.randint(0, 10, (3, 3)) #matriz 3x3 con enteros entre 0 y 10
print(f"Array de enteros aleatorios con np.random.randint:\n{randomIntArr}")
```

    Array de enteros aleatorios con np.random.randint:
    [[0 1 1]
     [4 3 5]
     [6 3 3]]

## Create an array of random numbers with normal distribution

- ``np.random.normal(loc, scale, (size))``: Genera una matriz con números aleatorios distribuidos normalmente, con media ``loc`` y desviación estándar ``scale``. ``size`` define la forma de la matriz.

```python
#Crear un array de números aleatorios con distribución normal
normalArr = np.random.normal(0, 1, (3, 3)) #matriz 3x3 con distribución normal (media=0, desviación estándar=1)
print(f"Array con distribución normal con np.random.normal:\n{normalArr}")
```

    Array con distribución normal con np.random.normal:
    [[ 0.06254963 -0.88010308  1.09985282]
     [-0.3366856  -1.13185179  1.93241312]
     [ 0.60933206 -0.88857287  0.21925543]]

## Create an empty array

Una matriz vacía se crea utilizando la función ``np.empty()``. Esta función crea una matriz con la forma especificada, pero sin inicializar sus elementos con ningún valor específico. Los valores iniciales serán basura (valores que ya estaban en memoria en esas posiciones).

```python
#Crear un array vacío
emptyArr = np.empty((2, 2)) #matriz 2x2 sin inicializar
print(f"Array vacío con np.empty:\n{emptyArr}")
```

    Array vacío con np.empty:
    [[0.25 0.5 ]
     [0.75 1.  ]]
