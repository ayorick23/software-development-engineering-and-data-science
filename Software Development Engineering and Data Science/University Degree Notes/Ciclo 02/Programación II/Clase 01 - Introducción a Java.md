---
Fecha de creación: 2025-07-07T18:09:00
Materia:
  - Programación II
Fecha de clase: 2025-07-07
---
# Historia de Java

**Java** fue creado en 1991 por James Gosling y su equipo en **Sun Microsystems**. Originalmente se llamó "**Oak**".

El objetivo era desarrollar un lenguaje para dispositivos electrónicos pequeños, pero con el auge de Internet en los 90, Java se adaptó para la web gracias a su portabilidad y su capacidad para ejecutarse en navegadores mediante applets.

En 1995, Sun lanzó Java 1.0, destacando su **JVM** (_Java Virtual Machine_) y su enfoque orientado a objetos. Con los años, Java evolucionó con mejoras como:
- **Java SE** (_Standard Edition_) para aplicaciones generales.
- **Java EE** (_Enterprise Edition_) para sistemas corporativos.
- **Java ME** (_Micro Edition_) para dispositivos móviles.

Hoy, Java es uno de los lenguajes más usados en desarrollo backend, Android (aunque ahora Kotlin es preferido) y sistemas empresariales. Su filosofía de seguridad, portabilidad y comunidad lo mantiene relevante.

## Definición de una Clase en Java

Una clase es como un molde o plantilla que define cómo serán los objetos.

**Por ejemplo:**
Si tienes un molde para hacer galletas, la clase sería el molde y las galletas serían los objetos creados a partir de ese molde.

## ¿Qué es un Objeto en Java?

Un objeto es una instancia de una clase, es decir, una entidad concreta creada a partir de un "molde" (la clase) que define su estructura y comportamiento.

### Características de un objeto en Java

- Estado (Atributos): Datos que representan las características del objeto (ej: nombre, edad, color).
- Comportamiento (Métodos): Acciones que el objeto puede realizar (ej: ``calcular()``, ``imprimir()``).
- Identidad: Cada objeto es único  (aunque tenga los mismos valores que otro).

```java
public class HolaMundo{
	public static void main(String[] args){
		System.out.println("Hola Mundo");
	}
}
```

## Datos Primitivos Enteros

| Tipo  | Tamaño (bits) | Potencia                   | Valores Posibles           | Rango |
| ----- | ------------- | -------------------------- | -------------------------- | ----- |
| byte  | 8             | 28                         | 256                        |       |
| short | 16            | 216                        | 65,536                     |       |
| int   | 32            | 4,294,967,296              | 4,294,967,296              |       |
| long  | 64            | 18,446,744,073,709,551,616 | 18,446,744,073,709,551,616 |       |

## Datos Primitivos Decimales

| Tipo   | Tamaño (bits) | Precisión (bits significativos) | Valor por Defecto |
| ------ | ------------- | ------------------------------- | ----------------- |
| float  | 32            | 24 (=7 dígitos decimales)       | 0.0f              |
| double | 64            | 53 (=15-16 dígitos decimales)   | 0.0d              |

## Datos Primitivos Char

| Tipo | Tamaño (bits) | Rango      | Valores Posibles   | Valor por Defecto |
| ---- | ------------- | ---------- | ------------------ | ----------------- |
| char | 16            | 0 a 65,535 | 1 carácter Unicode | '\u000' (null)    |

## Datos Primitivos Booleanos

| Característica    | Valor/Datos                                          |
| ----------------- | ---------------------------------------------------- |
| Tamaño            | No está estrictamente definido (generalmente 1 byte) |
| Valores Posibles  | `true` o `false`                                     |
| Valor por Defecto | `false` (en variables de clase)                      |
## Operadores Aritméticos
(Similares a los [[Introduction to Python#Operadores Aritméticos|Operadores Aritméticos en Python]])

| Operador | Nombre         | Ejemplo | Resultado (si a = 10, b = 3) |
| -------- | -------------- | ------- | ---------------------------- |
| ``+``    | Suma           | a + b   | 13                           |
| ``-``    | Resta          | a - b   | 7                            |
| ``*``    | Multiplicación | a * b   | 30                           |
| ``/``    | División       | a / b   | 3 (división entera)          |
| ``%``    | Módulo (resto) | a % b   | 1 (resto de 10 / 3)          |
| ``++``   | Incremento     | a++     | a = 11 (post-incremento)     |
| ``--``   | Decremento     | a--     | a = 9 (post-decremento)      |

## Operadores de Asignación
(Similares a los [[Introduction to Python#Operadores de Asignación|Operadores de Asignación en Python]])

| Operador | Ejemplo    | Equivalente a |
| -------- | ---------- | ------------- |
| ``=``    | ``a = 5``  | ``a = 5``     |
| ``+=``   | ``a += 3`` | ``a = a + 3`` |
| ``-=``   | ``a -= 2`` | ``a = a - 2`` |
| ``*=``   | ``a *= 4`` | ``a = a * 4`` |
| ``/=``   | ``a /= 2`` | ``a = a / 2`` |
| ``%=``   | ``a %= 3`` | ``a = a % 3`` |

## Operadores Relacionales (Comparación)
(Similares a los [[Introduction to Python#Operadores Relacionales|Operadores Relacionales en Python]])

| Operador | Nombre            | Ejemplo | Resultado (`a = 5`, `b = 3`) |
| -------- | ----------------- | ------- | ---------------------------- |
| ==       | Igual a           | a == b  | false                        |
| !=       | Diferente de      | a != b  | true                         |
| >        | Mayor que         | a > b   | true                         |
| <        | Menor que         | a < b   | false                        |
| >=       | Mayor o igual que | a >= 5  | true                         |
| <=       | Menor o igual que | a <= 3  | false                        |

## Operadores Lógicos
(Similares a los [[Introduction to Python#Operadores Lógicos|Operadores Lógicos en Python]])

| Operador | Nombre             | Ejemplo      | Resultado (a=true, b=false)           |
| -------- | ------------------ | ------------ | ------------------------------------- |
| ``&&``   | ``AND`` (Y lógico) | ``a && b``   | ``false`` (ambos deben ser ``true``)  |
| ``\|\|`` | ``OR`` (O lógico)  | ``a \|\| b`` | ``true`` (al menos uno debe ser true) |
| ``!``    | ``NOT`` (Negación) | ``!a``       | ``false`` (invierte el valor)         |

```java
if (a > 0 && b > 0):
	System.out.println("Ambos son positivos");
```

## Estructuras de Control

### ``if-else``
(Similar a [[Control Flow#Sentencias Condicionales `if`, `elif`, `else`|if-elif-else en Python]])

Permite ejecutar un bloque de código si se cumple una condición.

```java
int edad = 10:

if (edad >= 10) {
	System.out.println("Eres mayor de edad");
} else {
	System.out.println("Eres menor de edad");
}
```

### ``if-else if-else``

Para múltiples condiciones.

```java
int nota = 75:

if (nota >= 90){
	System.out.println("Excelente (A)");
} else if (nota >= 80) {
	System.out.println("Muy bien (B)");
} else if (nota >= 70) {
	System.out.println("Bien (C)");
} else {
	System.out.println("Necesitas mejorar");
}
```

### ``switch``
(Similar a [[Control Flow#Estructura `match`-`case`|match-case en Python]])

Para selección múltiple basada en un valor.

```java
int diaSemana = 3;
String nombreDia;

switch (diaSemana) {
	case 1:
		nombreDia = "Lunes";
		break;
	case 2:
		nombreDia = "Martes";
		break;
	case 3:
		nombreDia = "Miércoles";
		break;
	// ... otros casos
	default:
		nombreDia = "Día inválido";
}
System.out.println(nombreDia);
```

### ``for``
(Similar a [[Control Flow#Bucle `for`|for en Python]])

Para ejecutar un bloque un número específico de veces.

```java
for (int i = 1; 1 <= 5; i++){
	System.out.println("Iteracion " + 1);
}
```

### ``while``
(Similar a [[Control Flow#Bucle `while`|while en Python]])

Ejecuta mientras se cumpla una condición.

```java
int contador = 1;
while (contador <= 5){
	System.out.println("Contador: " + contador);
	contador++
}
```

### ``do-while``
(Similar a [[Control Flow#Bucle `do`-`while`|do-while en Python]])

Similar al while, pero garantiza al menos una ejecución.

```java
int num = 5;
do {
	System.out.println("Número: " + num);
	num--;
} while (num > 0);
```

## Estructuras de Salto
(Similar a las [[Control Flow#Declaraciones de control de bucle|declaraciones de control de bucle en Python]])

### ``break``

Termina la ejecución de un bucle o switch.

```java
for (int = 0; 1 < 10; i++){
	if (1 == 5){
		break; // Sale del bucle cuando 1 es 5
	}
	System.out.println(i)
}
```
### ``continue``

Salta a la siguiente iteración del bucle.

```java
for (int i = 0; 1 < 10; i++) {
	if (i % 2 == 0) {
		continue; // Salta números pares
	}
	System.out.println(i);
}
```
### ``return``
(Similar a los [[Functions#Valores de Retorno|valores de retorno en Python]])

Sale de un método y opcionalmente devuelve un valor.

```java
public int sumar(int a, int b) {
	return a + b;
}
```