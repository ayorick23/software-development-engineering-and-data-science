# Mathematical Operations

## Operaciones Matemáticas

En **NumPy**, las operaciones matemáticas se pueden realizar de manera eficiente sobre arreglos ([[Array Creation#Creación de Arrays|arrays]]) a través de la vectorización. Esto permite realizar operaciones elemento por elemento sin necesidad de [[Control Flow|bucles]] explícitos, lo que resulta en un mejor rendimiento.

```python
import numpy as np
```

## Create arrays to perform operations

```python
#Crear arrays para realizar operaciones
arr1 = np.array([10, 20, 30, 40])
arr2 = np.array([1, 2, 3, 4])
```

## Arithmetic Operations

Las operaciones aritméticas se realizan de forma eficiente a nivel de elemento. Esto significa que las operaciones como suma, resta, multiplicación y división se aplican a cada elemento correspondiente de los arreglos o matrices. Además de los [[Introduction to Python#Operadores Básicos|operadores básicos]], NumPy ofrece funciones como ``add``, ``subtract``, ``multiply``, ``divide``, entre otras, para realizar estas operaciones.

### Sum of arrays

Dos arrays con la misma forma se pueden sumar directamente usando el operador `+`. Si los arrays tienen formas diferentes, se utiliza la transmisión (*broadcasting*) si es posible. La función `np.add()` realiza la misma operación de suma elemento a elemento, aceptando dos arrays como argumentos y devolviendo un nuevo array con la suma.

```python
#Suma de arrays
suma = arr1 + arr2
print(f"Suma de arrays: {suma}")
```

    Suma de arrays: [11 22 33 44]

```python
#Suma de arrays con add()
suma_add = np.add(arr1, arr2)
print(f"Suma de arrays con add(): {suma_add}")
```

    Suma de arrays con add(): [11 22 33 44]

### Array substraction

La resta de arrays se realiza elemento a elemento. Se puede utilizar el operador `-` o la función `np.subtract()` para realizar esta operación. Ambas opciones son válidas y devuelven un nuevo array con la diferencia entre los elementos de los arrays de entrada.

```python
#Resta de arrays
resta = arr1 - arr2
print(f"Resta de arrays: {resta}")
```

    Resta de arrays: [ 9 18 27 36]

```python
#Resta de arrays con subtract()
resta_substract = np.subtract(arr1, arr2)
print(f"Resta de arrays con subtract(): {resta_substract}")
```

    Resta de arrays con subtract(): [ 9 18 27 36]

### Element by element multiplication

La multiplicación de arrays puede realizarse de varias formas, dependiendo del tipo de multiplicación deseada. Para la multiplicación elemento a elemento, se usa el operador `*` o la función `np.multiply()`.

```python
#Multiplicación elemento por elemento
multiplicacion = arr1 * arr2
print(f"Multiplicación de arrays: {multiplicacion}")
```

    Multiplicación de arrays: [ 10  40  90 160]

```python
#Multiplcación elemento por elemento con multiply()
multiplicacion_multiply = np.multiply(arr1, arr2)
print(f"Multiplicación de arrays con multiply(): {multiplicacion_multiply}")
```

    Multiplicación de arrays con multiply(): [ 10  40  90 160]

### Element by element division

La división de arrays se realiza principalmente usando el operador `/` o la función `np.divide()`. Ambas opciones realizan la división elemento por elemento de dos arrays, devolviendo un nuevo array con los resultados.

```python
#División elemento por elemento
division = arr1 / arr2
print(f"División de arrays: {division}")
```

    División de arrays: [10. 10. 10. 10.]

```python
#División elemento por elemento con divide()
division_divide = np.divide(arr1, arr2)
print(f"División de arrays con divide(): {division_divide}")
```

    División de arrays con divide(): [10. 10. 10. 10.]

### Element by element power

la operación de potencia se realiza utilizando la función `np.power()` o el operador `**`. Ambas opciones elevan cada elemento de un array a la potencia especificada, ya sea un escalar o un array con la misma forma.

```python
#Potencia elemento por elemento
potencia = arr1 ** 2
print(f"Potencia de cada elemento de arr1: {potencia}")
```

    Potencia de cada elemento de arr1: [ 100  400  900 1600]

```python
#Potencia elemento por elemento con power()
potencia_power = np.power(arr1, 2)
print(f"Potencia de cada elemento de arr1 con power(): {potencia_power}")
```

    Potencia de cada elemento de arr1 con power(): [ 100  400  900 1600]

## Mathematical Functions

NumPy también ofrece funciones para operaciones más complejas, como:
- **Funciones exponenciales y logarítmicas:** `np.exp()`, `np.log()`, etc.
- **Funciones trigonométricas:** `np.sin()`, `np.cos()`, `np.tan()`, etc.

Estas funciones también se aplican de forma vectorizada a los elementos de los arreglos.

### Exponential and logarithmic functions

```python
#Funciones exponenciales y logarítmicas
raiz_cuadrada = np.sqrt(arr1)
print(f"Raíz cuadrada de arr1: {arr1}")

logaritmo = np.log(arr1)
print(f"Logaritmo natural de arr1: {logaritmo}")

logaritmoBase10 = np.log10(arr1)
print(f"Logaritmo base 10 de arr1: {logaritmoBase10}")

exponencial = np.exp(arr1)
print(f"Exponencial de arr1: {exponencial}")

valorAbsoluto = np.abs(arr1 - 25)
print(f"Valor absoluto de (arr1 - 25): {valorAbsoluto}")
```

    Raíz cuadrada de arr1: [10 20 30 40]
    Logaritmo natural de arr1: [2.30258509 2.99573227 3.40119738 3.68887945]
    Logaritmo base 10 de arr1: [1.         1.30103    1.47712125 1.60205999]
    Exponencial de arr1: [2.20264658e+04 4.85165195e+08 1.06864746e+13 2.35385267e+17]
    Valor absoluto de (arr1 - 25): [15  5  5 15]

### Trigonometric functions

```python
#Funciones trigonométricas
seno = np.sin(np.radians(arr1))
print(f"Seno de arr1 en grados: {seno}")

coseno = np.cos(np.radians(arr1))
print(f"Coseno de arr1 en grados: {coseno}")

tangente = np.tan(np.radians(arr1))
print(f"Tangente de arr1 en grados: {tangente}")
```

    Seno de arr1 en grados: [0.17364818 0.34202014 0.5        0.64278761]
    Coseno de arr1 en grados: [0.98480775 0.93969262 0.8660254  0.76604444]
    Tangente de arr1 en grados: [0.17632698 0.36397023 0.57735027 0.83909963]

## Statistical Operations

Las operaciones estadísticas ([[Aggregate Functions|funciones de agregación en SQL]]) se realizan utilizando funciones que permiten calcular resúmenes numéricos de los datos contenidos en arrays. Estas funciones son esenciales para el análisis de datos, permitiendo obtener información relevante como la media, la mediana, la desviación estándar, entre otros.

- ``np.sum(array)``: Calcula la suma de todos los elementos del array.

```python
#Suma total con sum()
print(f"Suma total de arr1: {np.sum(arr1)}")
```

    Suma total de arr1: 100

- `np.mean(array)`: Calcula la media aritmética de los elementos de un array.

```python
#Promedio con mean()
print(f"Promedio de arr1: {np.mean(arr1)}")
```

    Promedio de arr1: 25.0

- ``np.median(array)``: Encuentra la mediana de los elementos del array. La mediana es el valor central cuando los datos están ordenados.

```python
#Mediana con median()
print(f"Mediana de arr1: {np.median(arr1)}")
```

    Mediana de arr1: 25.0

- ``np.std(array)``: Calcula la desviación estándar, que mide la dispersión de los datos respecto a la media.

```python
#Desviación estándar con std()
print(f"Desviación estándar de arr1: {np.std(arr1)}")
```

    Desviación estándar de arr1: 11.180339887498949

- ``np.var(array)``: Determina la varianza, que es el cuadrado de la desviación estándar.

```python
#Varianza con var()
print(f"Varianza de arr1: {np.var(arr1)}")
```

    Varianza de arr1: 125.0

- ``np.min(array)``: Encuentra el valor mínimo en el array.
- ``np.max(array)``: Encuentra el valor máximo en el array.

```python
#Mínimo y máximo con min() y max()
print(f"Mínimo de arr1: {np.min(arr1)}")
print(f"Máximo de arr1: {np.max(arr1)}")
```

    Mínimo de arr1: 10
    Máximo de arr1: 40

- ``np.ptp(array)``: Calcula el rango (diferencia entre el máximo y el mínimo). 

```python
#Diferencia entre máximo y mínimo
print(f"Diferencia entre máximo y mínimo de arr1: {np.ptp(arr1)}")
```

    Diferencia entre máximo y mínimo de arr1: 30

- ``np.percentile(array, q)``: Calcula el percentil q-ésimo de los datos. Por ejemplo, el percentil 75 indica el valor por debajo del cual se encuentra el 75% de los datos.

```python
#Percentiles con percentile()
print(f"Percentil 25 de arr1: {np.percentile(arr1, 25)}")
print(f"Percentil 50 de arr1: {np.percentile(arr1, 50)}")
print(f"Percentil 75 de arr1: {np.percentile(arr1, 75)}")
```

    Percentil 25 de arr1: 17.5
    Percentil 50 de arr1: 25.0
    Percentil 75 de arr1: 32.5
