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



## Declaración de un Array

```java
tipo[] nombreArray;  // Forma recomendada
tipo nombreArray[];  // Forma alternativa (menos usada)
```

## Métodos Comunes con Arrays

- `.length`: Devuelve el tamaño del array.
- Ordenación y búsqueda:
	- `Arrays.sort(array)`: Ordena el array completo.
	- `Arrays.sort(array, fromIndex, toIndex)`: Ordena un rango del array.
	- `Arrays.binarySearch(array, key)`: Búsqueda binaria (**el array debe estar ordenado**).
- Comparación:
	- `Arrays.equals(array1, array2)`: Compara si dos arrays son iguales.
	- `Arrays.deepEquales(array1, array2)`: Para arrays multidimensionales.

