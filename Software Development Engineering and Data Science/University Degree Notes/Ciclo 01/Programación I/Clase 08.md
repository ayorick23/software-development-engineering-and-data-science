# PROGRAMACIÓN I: clase 8

Fecha de creación: 22 de marzo de 2025 15:16
Clase: PROGRAMACIÓN I
Fecha de la clase: 15 de marzo de 2025

# Arrays y Colecciones avanzadas

## Tuplas (`tuples`)

Una tupla es similar a una lista, pero con una diferencia importante: es inmutable, lo que significa que no puedes modificar sus elementos después de que se han creado. Esto lo hace útil cuando deseas asegurarte de que los datos no cambien.

### Sintaxis:

```python
tupla = (1, 2, 3)
```

### Características:

- Ordenada: Los elementos se mantiene en el orden en que fueron agregados.
- Inmutable: No se pueden modificar, agregar ni eliminar elementos.
- Pueden contener diferentes tipos de datos.

```python
tupla = (1, "hola", 3.14)
print(tupla[1]) #"hola"
```

Intentar modificar un elemento de la tula generar[a un error:

```python
tupla[1] = "nuevo" #Error: TypeError: 'tuple' object does not support item assigment
```

## Diccionarios (`dicts`)

Un diccionario es una colección no ordenada (en versiones anteriores a Pythhon 3.7), mutable que almacena pares de clave-valor. En lugar de acceder a los elementos por un índice, se accede mediante claves únicas.

### Sintaxis:

```python
diccionario = {"clave1:" "valor1", "clave2": "valor2"}
```

### Características:

- No ordenado: Aunque en versiones modernas de Python, los diccionarios mantienen el orden de inserción.
- Acceso rápido por clave.
- Las claves deben ser únicas.

```python
persona = {"nombre": "Juan", "edad": 30, "ciudad": "San Salvador"}
print(persona["nombre"]) #"Juan"
persona["edad"] = 31 #modificar un valor
```

## Diccionarios con valores por defecto

Por defecto, si intentamos acceder a una clave inexistente en un diccionario normal, obtenemos un error:

```python
mi_dicc = {"a": 1, "b": 2}
print(mi_dicc["c"]) #KeyError: "c"
```

Para evitar esto, podemos usar `dict.get()` o `collections.defaultdict`.

### Opción 1: Usar `get()`

Permite devolver un valor por defecto si la clave no existe.

```python
mi_dicc = {"a": 1, "b": 2}
print(mi_dicc.get("c", 0)) #0 (valor por defecto)
```

### Opción 2: Usar `defaultdict` de `collections`

Esto crea un diccionario donde las claves inexistentes obtienen un valor por defecto automáticamente.

```python
from collections import defaultdict

#Diccionario con valor por defecto 0 para claves inexistentes
mi_dicc = defaultdict(int) #'int' crea valores 0 por defecto
mi_dicc["x"] += 5 #no da error, x se inicializa en 0 y luego se suma 5
print(mi_dicc["x"]) #5
```

También podemos usar `list`, `set` o una función personaliza:

```python
mi_dicc = defaultdict(list) #claves inexistentes tendrán una lista vacía
mi_dicc["nombres"].append("Juan")
print(mi_dicc) #defaultdict(<class 'list'>, {'nombres': ['Juan']})
```

## Conjuntos (`sets`)

Un conjunto es una colección no ordenada de elementos únicos. No permiten elementos duplicados y no mantienen el orden de inserción.

### Sintaxis:

```python
conjunto = {1, 2, 3}
```

### Características:

- No ordenado: No se puede acceder a los elementos por índice.
- Sin elementos duplicados.

```python
conjuntos = {1, 2, 3, 3, 4} #el conjunto eliminará el valor duplicado 3
print(conjunto) #{1, 2, 3, 4}
```

## Arreglos (`arrays`)

En Python, normalmente usamos listas en lugar de arrays, ya que las listas son muy flexibles. Sin embargo, existe el módulo array que permite trabajar con arrays más eficientes en memoria cuando todos los elementos son del mismo tipo.

### ¿Cuándo usar array en lugar de listas?

Si necesitas eficiencia en memoria y rendimiento para datos numéricos de gran tamaño, array es mejor que una lista normal.

```python
import array

#Crear un array de enteros
arr = array.array('i', [1, 2, 3, 4, 5])

#Acceder a elementos
print(arr[0]) #1

#Agregar elementos
arr.append(6)
print(arr) #array('i', [1, 2, 3, 4, 5, 6])

#Eliminar elementos
arr.remove(3)
print(arr) #array('i', [1, 2, 4, 5, 6])
```

- `'i'` indica que el array contiene enteros con 4 bytes cada uno.
- `array('f', [1.2, 2.5, 3.8])` para un array de flotantes.