# PROGRAMACIÓN I: clase 5

Fecha de creación: 22 de febrero de 2025 11:48
Clase: PROGRAMACIÓN I
Fecha de la clase: 22 de febrero de 2025

[[Clase 04 - Listas, Arreglos, Matrices y Estructuras Condicionales|← Clase anterior]] | [[Clase 07 - Indexación y Slicing, Collections y Deques|Clase siguiente →]]

# Operador Ternario (Sentencia Condicional)
(ver [[Control Flow#Operador Condicional Ternario|Operador Condicional Ternario]])

Un operador ternario es una **expresión condicional que toma tres argumentos y permite ejecutar diferentes bloques de código dependiendo de si una condición es verdadera o falsa**.

```python
edad = 18

if edad >= 18:
    mensaje = "Eres mayor de edad"
else:
    mensaje = "Eres menor de edad"

print(mensaje)
```

```python
edad = 18
mensaje = "Eres mayor de edad" if edad >= 18 else "Eres menor de edad"

print(mensaje)
```

<aside>
📝

También se puede anidar otro `if` en el `else` pero ya se vuelve un poco ilegible el código.

</aside>

```python
myList = [1, 3, -6, -9, -0.2, -1]

bool_list = [True if x > 0 else False for x in myList]
```

# Bucles

Los bucles son estructuras de control que permiten ejecutar un bloque de código repetidamente mientras se cumpla una condición especifica. Son esenciales en la programación porque:

- Automatizan tareas repetitivas, reduciendo la cantidad de código necesario.
- Mejoran la eficiencia y escalabilidad del código al evitar la repetición manual.
- Facilitan el procesamiento de datos en estructuras como listas, arreglos o bases de datos.
- Se combinan con otras estructuras (como condicionales) para realizar operaciones mas complejas.

## Bucle `for`
(ver [[Control Flow#Bucle `for`|Bucle for]])

Se usa cuando **sabemos cuantas veces queremos repetir un bloque de código.** Se ejecuta sobre secuencias como listas o rangos.

```python
for i in range(1, 6):
	print(f"Iteracion {i}")
```

<aside>
📝

`range(start, stop, step)`

- `start` (opcional): Numero desde donde comienza la secuencia (por defecto es `0`).
- `stop`: Número donde termina la secuencia (no esta incluido).
- `step` (opcional): Incremento o decremento entre los números (por defecto es `1`).
</aside>

```python
frutas = ["Manzana", "Mango", "Aguacate", "Fresa"]

for indice, fruta in enumerate(frutas, start=1):
	print(f"{fruta}, en el indice {indice}")
```

<aside>
📝

`enumerate(iterable, start=0)`

- `iterable`: La secuencia que queremos recorrer (lista, tupla, string, etc.).
- `start`(opcional): El índice desde donde comienza la numeración (por defecto es `0`).
</aside>

```python
for i in range(1, 10):
	if i == 5:
		break
	print(i)
```

```python
for i in range(1, 6):
	if i == 3:
		continue
	print(i)
```

## Bucle `while`
(ver [[Control Flow#Bucle `while`|Bucle while]])

Ejecuta un bloque de código mientras una condición sea verdadera. **Se usa cuando no sabemos cuantas veces se repetirá el ciclo**, pero si conocemos la condición de salida.

```python
contador = 1

while contador <= 5:
	print(f"Iteracion {contador}")
	contador += 1
```