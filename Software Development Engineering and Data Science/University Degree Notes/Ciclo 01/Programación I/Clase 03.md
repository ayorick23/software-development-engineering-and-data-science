# PROGRAMACIÓN I: clase 3

Fecha de creación: 8 de febrero de 2025 14:17
Clase: PROGRAMACIÓN I
Fecha de la clase: 8 de febrero de 2025

### Resolución de diagrama de flujo de foro de participación

1. Se inicia con una elipse
2. Necesitamos dos variables (primer número, otro número)
3. Proceso leer número para evitar el error que se ingrese otro tipo de dato
4. Decisión si el dato ingresado es número
5. En caso de ser así se pasa a otra decisión en donde se evalúa el residuo para saber si es par o impar
6. Impresión en pantalla si es par o impar
7. Decisión de ingresar otro número para evaluar
8. En caso de si regresa a ingresar un número, caso contrario finaliza el diagrama

## Operadores Aritméticos

Permiten realizar operaciones matemáticos sobre valores numéricos. Python admite operaciones básicas como suma, resta, multiplicación y otras más avanzadas como exponenciación y módulo.

| Operador | Descripción | Ejemplo | Resultado |
| --- | --- | --- | --- |
| `+` | Suma | 5 + 3 | 8 |
| `-` | Resta | 10 - 4 | 6 |
| `*` | Multiplicación | 6 * 2 | 12 |
| `/` | División (retorna float) | 10 / 3 | 3.3333 |
| `//` | División Entera | 10 // 3 | 3 |
| `%` | Módulo (residuo de la división) | 10 % 3 | 1 |
| `**` | Exponenciación | 2 ** 3 | 8 |

## Operadores Lógicos

Permiten combinar expresiones booleanas. Se usan principalmente en estructuras de control como `if`, `while` y `for`.

### Tabla de operadores lógicos

| Operador | Descripción |
| --- | --- |
| `and` | Devuelve True si ambas condiciones son True |
| `or` | Devuelve True si al menos una condición es verdadera |
| `not` | Invierte el valor lógico |

<aside>
📝

**NOTA:** Prioridad de operadores lógicos **(NOT, AND, OR)**

</aside>

## Operadores de Comparación

Comparan valores y devuelven un valor booleano (`True` o `False`).

| Operador | Descripción |
| --- | --- |
| `==` | Igual a |
| `!=` | Diferente de |
| `>` | Mayor que |
| `<` | Menor que |
| `>=` | Mayor o igual que |
| `<=` | Menor o igual que |

## Prioridad de Operación

Cuando una expresión contiene múltiples operadores, Python sigue un orden de precedencia para evaluar la expresión correcta. Este orden define qué operadores se evalúan primero.

1. Paréntesis
2. Exponenciación
3. Multiplicación, División, División entera
4. Comparaciones
5. NOT
6. AND
7. OR

<aside>
📝

**NOTA:** Toda entrada desde teclado con la función `input` será de tipo `string`.

</aside>

### Ejemplos:

```python
num1 = input(int("Ingrese un numero: "))
num2 = input(int("Ingrese un numero: "))

#Suma
suma = num1 + num2

#Resta
resta = num1 - num2

#Multiplicacion
multiplicacion = num1 * num2

#Division decimal
division = num1 / num2

#Division entera
div_entera = num1 // num2

#Residuo
residuo = num1 % num2

#Exponenciacion
exponenciacion = num1 ** 2

print(type(num1))
print(type(num2))

print(f"La suma de {num1} y {num2} es: {suma}"
```