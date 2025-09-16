---
Fecha de creación: 2025-09-15T19:00:00
Materia:
  - Programación II
Fecha de clase: 2025-09-15
---
# Interfaces y Clases Abstractas en Java

La programación orientada a objetos (**POO**) es un paradigma que nos permite estructurar el código entorno a objetos que podrían ejemplificar entidades del mundo real. Bajo este enfoque, las interfaces y las clases abstractas son dos herramientas muy importantes que facilitan la organización del código y la reutilización de funcionalidades.

## ¿Qué es una Interfaz en Java?

Una interfaz en Java es una estructura que rige una especie de contrato para que sea aplicado en una clase. Este contrato está conformado por un conjunto de métodos que las clases que implementen la interfaz deben definir. En otras palabras, una interfaz actúa como una plantilla que garantiza que las clases adherentes tengan ciertas funcionalidades, sin imponer cómo deben implementarlas.

Por ejemplo, si se desea crear objetos que puedan volar en un programa, podemos definir una interfaz `Volador`, que contenga un método `volar()`. De esta forma, cualquier clase que utilice esta interfaz debe proporcionar su  propia implementación del método.

## Beneficios del Uso de Interfaces

- **Desacoplamiento:** Las interfaces permiten disminuir la dependencia entre clases, lo que optimiza el mantenimiento y escalabilidad del código.
- **Reutilización de código:** Distintas clases pueden utilizar la misma interfaz sin necesidad de compartir una jerarquía de herencia.
- **Facilitan la programación modular:** Se pueden crear módulos independientes que interactúan a través de interfaces, lo que facilita la división del trabajo en equipos de desarrollo.
- **Permiten el uso de polimorfismo:** Al programar con interfaces, se puede tratar a los objetos de distintas clases de la misma forma, siempre y cuando se implemente la misma interfaz.

**Ejemplo:**

```java
interface Volador {
	void volar();
}

class Pajaro implements Volador {
	@Override
	public void volar() {
		System.out.println("El pájaro vuela por el cielo.");
	}
}

class Avion implements Volador {
	@Override
	public void volar() {
		System.out.println("El avión despega y surca los cielos.");
	}
}
```

En este ejemplo:

- La interfaz ``Volador`` define el método `volar()` sin tener implementación.
- Tanto la clase `Pajaro` como la clase `Avión` implementan la interfaz `Volador`, lo que significa que ambas clases deben definir el método `volar()`.
- Gracias a la interfaz, podemos tratar objetos de `Pajaro` y `Avion` como instancias de `Volador`, sin importar cómo está implementando el método.

## Diferencias entre Interfaces y Clases Abstractas

Ambas herramientas permiten definir estructuras comunes para un conjunto de clases, pero tienen diferencias relevantes:

### Implementación Parcial VS Implementación Total

- Una **clase abstracta** puede contar tanto con métodos abstractos (sin implementación) como con método concretos (con implementación). Esto permite compartir lógica común entre varias subclases.
- Una **interfaz**, hasta Java 8, solo podía contener métodos sin implementación. Sin embargo, con la introducción de los métodos `default` y `static`, ahora es posible definir métodos con implementación dentro de una interfaz.

**Ejemplo:**

```java
abstract class Animal {
	abstract void hacerSonido();
	
	public void respirar() {
		System.out.println("Respirando...");
	}
}
```

En este ejemplo, el método `hacerSonido()` debe ser implementado por las subclases, mientras que `respirar()` ya tiene una implementación.

## Herencia y Múltiples Implementaciones

- Java no permite herencia múltiple de clases, por lo que una clase solo puede heredar de una **clase abstracta**.
- Sin embargo, una clase puede **implementar múltiples interfaces**, lo que permite mayor flexibilidad en la estructura del código.

**Ejemplo:**

```java
interface Nadador {
	void nadar();
}

interface Volador {
	void volar();
}

class Pato implements Nadador, Volador {
	@Override
	public void nadar() {
		System.out.println("El pato nada en el lago");
	}
	
	@Override
	public void volar() {
		System.out.println("El pato vuela sobre el agua");
	}
}
```

En este ejemplo se demuestra como la clase `Pato` puede implementar tanto el método `Nadador` como `Volador`, lo que le permite comportarse como un animal que puede nadar y volar sin necesidad de heredar de múltiples clases.

## Métodos `default` y `static` en Interfaces

Desde Java 8, las interfaces pueden contener métodos con implementación utilizando las palabras clave `default` y `static`. Esto extiende la flexibilidad de las interfaces al permitir agregar nuevos métodos sin afectar las clases existentes.

### Métodos `default` en Interfaces

Un método `default` permite agregar nuevas funcionalidades a una interfaz sin necesidad de modificar todas las clases que la implementan.

```java
interface Operaciones {
	default void imprimirMensaje() {
		System.out.println("Este es un método default en una interfaz");
	}
}

class Calculadora implements Operaciones {
	public void realizarOperacion() {
		System.out.println("Realizando una operación...");
	}
}

public class Main {
	public static void main(String[] args) {
		Calculadora calc = new Calculadora();
		calc.imprimirMensaje(); // Llamando al método default
		calc.realizarOperacion();
	}
}
```

Aquí, la clase `Calculadora` no implementa el método `imprimirMensaje()`, pero puede usarlo porque ya tiene una implementación en la interfaz.

### Métodos `static` en Interfaces

Los **métodos estáticos** en una interfaz no pertenecen a las clases que la usan, sino a la propia interfaz. Sirven para crear funciones útiles que no necesitan que se cree un objeto para usarlas.

**Ejemplo:**

```java
interface Utilidad {
	static void mostrarInfo() {
		System.out.println("Método estático en una interfaz");
	}
}

public class Main {
	public static void main(String[] args) {
		Utilidad.mostrarInfo(); // Se llama directamente desde la interfaz
	}
}
```

Los métodos estáticos en interfaces pueden ser útiles para definir métodos de ayuda que no necesiten ser sobrescritos por las clases que implementan la interfaz.