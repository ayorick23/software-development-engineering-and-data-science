# PROGRAMACIÓN I: clase 7

Fecha de creación: 8 de marzo de 2025 14:05
Clase: PROGRAMACIÓN I
Fecha de la clase: 8 de marzo de 2025

# Iterando información en listas

## Lista

Es una colección ordenada y mutable que permite almacenar múltiples elementos.

```python
numeros = [1, 2, 3, 4, 5]
palabras = ["hola", "mundo"]
mezclado = [1, "texto", 3.5]
```

### Principales operaciones con listas

- Acceder a elementos: `lista[indice]`
- Modificar elementos: `lista[indice] = nuevoValor`
- Agregar elementos: `append()`, `insert()`, `extend()`
- Eliminar elementos: `remove()`, `pop()`, `del()`
- Otras operaciones: `len()`, `index()`, `count()`, `sort()`, `reverse()`, `copy()`

### Conceptos fundamentales de listas

### Indexación y Slicing

```python
lista = [10, 20, 30, 40, 50]
print(lista[0]) #primer elemento
print(lista[-1]) #ultimo elemento
print(lista[1:4]) #desde el indice 1 hasta el 3
```

### Que es slicing?

Es una técnica para acceder a un subconjunto de elementos de una secuencia, como una lista, cadena de texto o tupla, utilizando una notación especial con corchetes[].

La sintaxis básica es: `secuencia[inicio:fin:paso]`

- inicio: Índice donde comienza el subarreglo (incluido).
- fin: Índice donde termina el subarreglo (no incluido).
- paso(Opcional): El salto entre elementos.

### Listas anidadas (matrices 2D)

```python
matriz = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print(matriz[1][2]) #accede al numero 6
```

### Comprensión de listas (`List Comprehension`)

```python
cuadrados = [x**2 for x in range(5)]
print(cuadrados) #[0, 1, 4, 9, 16]
```

## Collections y deque

El modulo `collections`: Proporciona estructuras de datos avanzadas mas eficientes.

`deque` (Double-ended queue):

- Mas rápido que una lista para inserciones y eliminaciones en los extremos.
- Soporta operaciones de cola y pila de forma optima.

```python
from collections import deque

dq = deque([1, 2, 3])
dq.append(4)      #agrega al final
dq.appendleft(0)  #agrega al inicio
print(dq)         #deque([0, 1, 2, 3, 4])

dq.pop()          #elimina del final
dq.popleft()      #elimina del inicio
print(dq)         #deque([1, 2, 3])
```

Ejercicio:

```python
import random

numMagic = random.randint(1, 100)

```

<aside>
📝

AGREGAR DO WHILE

Y MAS FUNCIONES DE DEQUE

</aside>