# PROGRAMACIÓN I: clase 2

Fecha de creación: 1 de febrero de 2025 14:34
Clase: PROGRAMACIÓN I
Fecha de la clase: 1 de febrero de 2025

[[Clase 03 - Operadores Aritméticos, Lógicos y de Comparación|Clase siguiente →]]

# Algoritmos y Diagramas de Flujo

Nos ayudan a entender la lógica de lo que vamos a programar

<aside>
⚠️

**HACER LAS LECTURAS SEMANALES ANTES DE LAS CLASES**

</aside>

Diagramas de flujo en LucidChart (ya tengo cuenta)

No tienen un estándar, los símbolos pueden variar

# Operadores Lógicos
(ver [[Introduction to Python#Operadores Lógicos|Operadores Lógicos]])

## Tablas de verdad

**NOT:** Negación

| Hecho | NOT |
| --- | --- |
| TRUE | FALSE |
| FALSE | TRUE |

**AND:** Conjunción (es excluyente, si un elemento es falso el resultado será falso)

| Valor1 | Valor2 | AND |
| --- | --- | --- |
| FALSE | FALSE | FALSE |
| FALSE | TRUE | FALSE |
| TRUE | FALSE | FALSE |
| TRUE | TRUE | TRUE |

**OR:** Disyunción (al menos un elemento tiene que ser verdadero para que el resultado sea verdadero)

| Valor1 | Valor2 | OR |
| --- | --- | --- |
| FALSE | FALSE | FALSE |
| FALSE | TRUE | TRUE |
| TRUE | FALSE | TRUE |
| TRUE | TRUE | TRUE |

**XOR (Exclusive Or):** Disyunción exclusiva (cuando ambas son iguales el resultado será falso)

| Valor1 | Valor2 | XOR |
| --- | --- | --- |
| FALSE | FALSE | FALSE |
| FALSE | TRUE | TRUE |
| TRUE | FALSE | TRUE |
| TRUE | TRUE | FALSE |

**NAND (Not AND):** Negación de la conjunción (es la única falsa cuando ambas son verdaderas)

| Valor1 | Valor2 | NAND |
| --- | --- | --- |
| FALSE | FALSE | TRUE |
| FALSE | TRUE | TRUE |
| TRUE | FALSE | TRUE |
| TRUE | TRUE | FALSE |

**NOR (Not OR):** Negación de la disyunción (solo es verdadera cuando ambas son falsas)

| Valor1 | Valor2 | NOR |
| --- | --- | --- |
| FALSE | FALSE | TRUE |
| FALSE | TRUE | FALSE |
| TRUE | FALSE | FALSE |
| TRUE | TRUE | FALSE |

Ejemplo:

| OR | FALSE | FALSE | FALSE | FALSE | TRUE | = | TRUE |
| --- | --- | --- | --- | --- | --- | --- | --- |
| OR | TRUE | TRUE | TRUE | TRUE | FALSE | = | TRUE |
| AND | TRUE | TRUE | TRUE | TRUE | FALSE | = | FALSE |

## Variables en Python

Un espacio en memoria para almacenar información.

Ejemplo:

```python
number = 10
print(number)
#10
```

### Enteros (`int`)

Representan números enteros positivos o negativos, incluyendo al cero.

**Rango:**

En Python, lo enteros no tienen limites explícitos. Su tamaño depende de la memoria disponible, gracias a la implementación de enteros de precisión arbitraria.

```python
x = 42
y = -922364647838374 #un valor negativo muy grande
z = 922364647838374 #un valor positivo muy grande
```

Interpolación:

```python
print(f"Valor: {x}, Tipo: {type(x)}") #Valor: 42, Tipo: <class 'int'>
print(f"Valor: {y}, Tipo: {type(y)}") #Valor: -922364647838374, Tipo: <class 'int'>
print(f"Valor: {z}, Tipo: {type(z)}") #Valor: 922364647838374, Tipo: <class 'int'>
```

### Flotantes (`float`)

Representan números con punto decimal o en notación científica.

**Rango:**

El rango de los flotantes depende de la arquitectura del sistema, per típicamente:

- Valor mínimo: -1.7976931348623157e+308
- Valor máximo: 1.7976931348623157e+308
- Precisión: Hasta 15-16 dígitos decimales

```python
pi = 3.141592653589793
velocidad = 1.2e3 #notacion cientifica
min_float = -1.7976931348623157e+308
max_float = 1.7976931348623157e+308

print(f"El tipo de la variable pi es: {type(pi)}") #<class 'float'>
print(f"El tipo de la variable velocidad es: {type(velocidad)}") #<class 'float'>
```

### Cadenas (`string`)

Representan texto como letras, palabras o frases y se definen con comillas simples, dobles o triples.

**Rango:**

La capacidad de almacenamiento de las cadenas depende de la memoria disponible. No hay limite especifico.

```python
cadena = "Hola, Mundo"
char = 'a'
charB = 'B'

print(f"Variable cadena tipo: {type(cadena)}") #<class 'str'>
print(f"Variable char tipo: {type(char)}") #<class 'str'>
print(f"Variable charB tipo: {type(charB)}") #<class 'str'>
```

```python
holaMundo = "Hola Mundo" * 100
#Se imprimirá 100 veces la cadena "Hola Mundo"
```

### Booleanos (`bool`)

Representan valores lógicos: True o False.

**Rango:**

Solo puede tener dos valores: True o False.

```python
es_activo = True
es_mayor = 5 > 3

print(f"El valor de la variable es_activo es: {es_activo}"
print(type(es_activo)) #<class 'bool'>

print(f"El valor de la variable es_mayor es: {es_mayor}"
print(type(es_mayor)) #<class 'bool'>
```