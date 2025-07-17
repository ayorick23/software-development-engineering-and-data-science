---
Fecha de creación: 2025-07-14T18:27:00
Materia:
  - Programación II
Fecha de clase: 2025-07-14
---
# Métodos en Java

En Java, un método es un bloque de código que realiza una tarea específica y puede ser invocado "_llamado_" desde otras partes del programa. Los métodos permiten organizar el código en unidades reutilizables, lo que mejora la legibilidad y mantenibilidad del programa.

## Características de los Métodos en Java

1. **Reutilización de código:** Evita repetir la misma lógica en diferentes partes del programa.
2. **Modularidad:** Divide un problema complejo en partes más pequeñas y manejables.
3. **Encapsulamiento:** Permite ocultar detalles de implementación.
4. Pueden **retornar un valor** o no (``void``).
5. Pueden **recibir parámetros** (datos de entrada) o no.

## Sintaxis de un Método en Java

```java
modificador_de_acceso tipo_de_retorno nombreDelMetodo (parámetros) {
    // Cuerpo del método (instrucciones)
    return valor; // (Opcional, solo si no es void)
}
```

1. **Modificador de acceso:** Define desde dónde puede ser llamado el método (``public``, ``private``, ``protected``, o por defecto).
2. **Tipo de retorno:** Indica qué tipo de dato devuelve el método (``int``, ``String``, ``void``, etc.).
3. **Nombre del método:** Identificador que sigue convenciones de Java (_camelCase_).
4. **Parámetros:** Datos de entrada que el método puede recibir (opcional).
5. **Cuerpo del método:** Bloque de código que se ejecuta al llamar al método.
6. **return:** Devuelve un valor (excepto en métodos ``void``).

### Métodos sin retorno (``void``) y sin parámetros

```java
public void saludar() {
	System.out.println("Hola, Mundo!");
}
```
```java
saludar (); // Imprime: Hola, Mundo!
```

### Método con retorno (``int``) y parámetros

```java
public int sumar(int a, int b) {
	return a + b;
}
```
```java
int resultado = sumar(5, 3); // resultado = 8
```

### Método con retorno (``String``)

```java
public String obtenerMensaje(String nombre) {
	return "Bienvenido, " + nombre;
}
```
```java
String mensaje = obtenerMensaje("Ana"); // mensaje = "Bienvenido, Ana"
```

### Método estático (se llama sin crear un objeto)

```java
public static int multiplicar(int x, int y) {
	return x * y;
}
```
```java
int producto = multiplicar(4, 5) // producto = 20
```

## Tipos de Métodos en Java

- Métodos de instancia: Requieren un objeto para ser llamados.
```java
Objeto obj = new Objeto();
obj.metodoDeInstancia();
```

- Métodos estáticos (``static``): Se llaman directamente desde la clase.
```java
Clase.metodoEstatico()
```

- Métodos constructores: Inicializan objetos (``public Clase()``).
- Getters y Setters: Acceden y modifican atributos privados.
- Métodos abstractos: Declarados en interfaces o clases abstractas (sin implementación).

## Modificadores de Acceso

### ``public``

```java
public class MiClase {
	public int variablePublica;

	public void metodoPublico() {
		//codigo
	}
}
```

### `protected`

```java
public class MiClase {
	public int variableProtegida;

	public void metodoProtegido() {
		//codigo
	}
}
```

### `default` (sin modificador-package-private)

```java
class MiClase {
	int variableDefault;

	void metodoDefault() {
		//codigo
	}
}
```

### `private`

```java
public class MiClase {
	public int variablePrivada;

	public void metodoPrivado() {
		//codigo
	}
}
```

### Tabla Resumen

| Modificador | Misma Clase | Mismo Paquete | Subclases | Otros Paquetes |
| ----------- | ----------- | ------------- | --------- | -------------- |
| public      | Sí          | Sí            | Sí        | Sí             |
| protected   | Sí          | Sí            | Sí        | No*            |
| default     | Sí          | Sí            | No        | No             |
| private     | Sí          | No            | No        | No             |
\*: Las subclases en otros paquetes si pueden acceder a miembros protected, pero otras clases en esos paquetes no.
Los modificadores de acceso son fundamentales para implementar el encapsulamiento en la programación orientada a objetos.