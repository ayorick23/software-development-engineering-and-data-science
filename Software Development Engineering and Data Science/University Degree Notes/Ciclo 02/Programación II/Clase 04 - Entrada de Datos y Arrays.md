---
Fecha de creación: 2025-07-28T17:57:00
Materia:
  - Programación II
Fecha de clase: 2025-07-28
---
# Entrada de Datos en Java

La entrada de datos en Java se refiere al proceso de recibir información desde fuentes externas (como el teclado, archivos, bases de datos, etc.) para ser procesada por un programa. Es una operación fundamental en la programación que permite la interacción entre el usuario y la aplicación.

En Java existen varias formas de realizar la entrada de datos, dependiendo de la versión de Java y del tipo de aplicación que estemos desarrollando. Las principales formas son:

- `Scanner (java.util.Scanner)`: La clase más común y fácil de usar para entrada desde consola.
- `BufferedReader (java.io.BufferedReader)`: Más eficiente para grandes volúmenes de datos.
- `JOptionPane (javax.swig.JOptionPane)`: Para interfaces gráficas simples.
- `System.console()`: Para lectura de contraseñas sin eco en pantalla.

# Array en Java

En Java, un array es una estructura de datos que permite almacenar una colección de elementos del mismo tipo en una única variable. Los arrays son de tamaño fijo, lo que significa que su longitud se define al momento de crearlos y no puede ser modificada posteriormente. Cada elemento del array se puede acceder a través de un índice numérico, comenzando desde 0 para el primer elemento.

## Declaración de un Array

```java
tipo[] nombreArray;  // Forma recomendada
tipo nombreArray[];  // Forma alternativa (menos usada)
```

## Métodos Comunes con Arrays

- `.length`: Devuelve el tamaño del array.
- **Ordenación y búsqueda:**
	- `Arrays.sort(array)`: Ordena el array completo.
	- `Arrays.sort(array, fromIndex, toIndex)`: Ordena un rango del array.
	- `Arrays.binarySearch(array, key)`: Búsqueda binaria (**el array debe estar ordenado**).
- **Comparación:**
	- `Arrays.equals(array1, array2)`: Compara si dos arrays son iguales.
	- `Arrays.deepEquales(array1, array2)`: Para arrays multidimensionales.
- **Relleno:**
	- `Arrays.fill(array, value)`: Rellena todo el array con un valor.
	- `Arrays.fill(array, fromIndex, toIndex, value)`: Rellena un rango.
- **Copia:**
	- `Arrays.copyOf(original, newLength)`: Copia el array con una nueva longitud.
	- `Arrays.copyOfRange(original, from to)`: Copia un rango del array.
- **Conversión a String:**
	- `Arrays.toString(array)`: Convierte a String legible.
	- `Arrays.deepToString(array)`: Para arrays multidimensionales.
- **Otros:**
	- `Arrays.stream(array)`: Convierte el array a un Stream (Java 8+).
	- `Arrays.asList(array)`: Convierte el array a una List (lista de tamaño fijo).
