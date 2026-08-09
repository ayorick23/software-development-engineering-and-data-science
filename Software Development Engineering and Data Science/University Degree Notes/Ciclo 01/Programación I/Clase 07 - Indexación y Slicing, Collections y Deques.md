# PROGRAMACIÓN I: clase 7

Fecha de creación: 8 de marzo de 2025 14:05
Clase: PROGRAMACIÓN I
Fecha de la clase: 8 de marzo de 2025

[[Clase 05 - Operador Ternario, Bucles For y While|← Clase anterior]] | [[Clase 08 - Tuplas, Diccionarios, Sets y Arrays|Clase siguiente →]]

# Iterando información en listas

## Lista
(ver [[Lists and Tuples#Listas|Listas]])

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
(ver [[Lists and Tuples#Índices y Slicing (_Rebanado_)|Indexación y Slicing]])

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
(ver [[Control Flow#Listas por Comprensión|Listas por Comprensión]])

```python
cuadrados = [x**2 for x in range(5)]
print(cuadrados) #[0, 1, 4, 9, 16]
```

## Collections y deque
(ver [[Data Structures and Algorithms#Queue (_Cola_)|Queue (Cola)]])

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

### Más funciones de `deque`

```python
dq = deque([1, 2, 3])

dq.extend([4, 5])       #agrega varios elementos al final -> deque([1, 2, 3, 4, 5])
dq.extendleft([0, -1])  #agrega varios al inicio, en orden inverso -> deque([-1, 0, 1, 2, 3, 4, 5])
dq.rotate(1)             #rota los elementos 1 posición a la derecha
dq.rotate(-1)            #rota los elementos 1 posición a la izquierda
dq.count(2)               #cuenta cuantas veces aparece un valor
dq.remove(2)               #elimina la primera aparición de un valor
dq.clear()                   #vacía el deque
```

<aside>
📝

`deque(maxlen=n)`: se le puede pasar un tamaño máximo; al llenarse, cada elemento nuevo descarta automáticamente el más antiguo del otro extremo. Útil para cosas como un historial reciente o un buffer circular.

</aside>

## Bucle `do`-`while`
(ver [[Control Flow#Bucle `do`-`while`|Bucle do-while]])

Python no tiene una sentencia `do`-`while` nativa como otros lenguajes (que garantiza que el bloque se ejecute al menos una vez antes de evaluar la condición). Se emula combinando `while True` con un `if` y `break`:

```python
while True:
    numero = int(input("Ingrese un número (0 para salir): "))
    print(f"Ingresaste: {numero}")
    if numero == 0:
        break
```

Ejercicio:

```python
import random

numMagic = random.randint(1, 100)

```