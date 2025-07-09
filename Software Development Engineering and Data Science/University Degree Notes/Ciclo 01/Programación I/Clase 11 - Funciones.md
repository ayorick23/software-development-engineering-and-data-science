# PROGRAMACIÓN I: clase 11

Fecha de creación: 3 de mayo de 2025 15:08
Clase: PROGRAMACIÓN I
Fecha de la clase: 5 de abril de 2025

# Funciones en Python

Una función es un bloque de código reutilizable que realiza una tarea específica. Permiten estructurar el código en partes más pequeñas y manejables, evitando la repetición y facilitando la depuración.

Las funciones en Python se definen con la palabra clave `def`, seguidas de un nombre y paréntesis `()` que pueden contener parámetros.

### Ejemplo básico de una función sin parámetros

```python
def saludar():
	print("Hola, bienvenido")
	
saludar()
```

- `def saludar()`: Define una función llamada saludar.
- `print(”Hola, bienvenido”)`: Imprime un mensaje cuando se llama a la función.

### Ejemplo con parámetros y valor de retorno

```python
def sumar(a, b):
	return a + b
	
resultado = sumar(5, 3)
print(resultado) #8
```

- `def sumar(a, b)`: Declara una función que recibe dos parámetros (a y b).
- `return a + b`: Devuelve la suma de a y b.
- `resultado = sumar(5, 3)`: Llama a la función con valores y almacena el resultado.

### Ventajas de usar funciones

- Evitan la repetición de código.
- Mejoran la organización y legibilidad.
- Permiten la reutilización de código en diferentes partes del programa.

## Parámetros en funciones

Los parámetros permiten a las funciones recibir datos externos para procesarlos. Existen diferentes tipos de parámetros en Python:

### Parámetros por posición

Los valores se pasan en el mismo orden en que fueron definidos.

```python
def info(nombre, edad):
	print(f"Nombre: {nombre}, Edad: {edad}")
	
info("Ana", 25) #Nombre: Ana, Edad: 25
```

### Parámetros por valores por defecto

Si no se proporciona un valor, se usa el predeterminado.

```python
def saludar(nombre = "Invitado"):
	print(f"Hola", {nombre})
	
saludar()       #Hola, Invitado
saludar("Luis") #Hola, Luis
```

### Parámetros arbitrarios `(*args)`

Se usa para pasar una cantidad variable de argumentos en forma de tupla.

```python
def suma(*numeros):
	return sum(numeros)
	
print(suma(1, 2, 3, 4, 5)) #15
```

- `*args` convierte todos los valores en una tupla y permite operar sobre ellos.

### Parámetros con palabras clave `(**kwargs)`

Se usa para recibir argumentos con nombre en forma de diccionario.

```python
def mostrar_info(**datos):
	for clave, valor in datos.items():
		print(f"{clave}: {valor}")
		
mostrar_info(nombre="Ana", edad=25, ciudad="Madrid")
```

- `**kwargs` convierte los parámetros en un diccionario.
- Se puede recorrer con `.items()`.