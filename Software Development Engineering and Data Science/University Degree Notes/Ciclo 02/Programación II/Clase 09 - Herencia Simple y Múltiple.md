---
Fecha de creación: 2025-09-08T18:11:00
Materia:
  - Programación II
Fecha de clase: 2025-09-08
---
[[Clase 08 - Git y el Control de Versiones|← Clase anterior]] | [[Clase 10 - Interfaces y Clases Abstractas en Java|Clase siguiente →]]

# Diferencia entre Herencia Simple y Múltiple
(ver [[Clase 03 - Herencia y Clases Abstractas#Herencia Simple y Multinivel|Herencia Simple y Multinivel en Java]])

La herencia es un mecanismo que permite que una clase derive características y comportamientos de otra. Existen dos tipos principales de herencia:

## Herencia Simple

La herencia simple es un mecanismo fundamental de la programación orientada a objetos (_POO_) en Java. Permite que una clase (subclase o clase hija) herede atributos y métodos de otra clase (superclase o clase padre). La característica "_simple_" radica en que una subclase solo puede heredar directamente de una única superclase.

### Conceptos Clave

- **Superclase (Clase Padre):** Es la clase cuyos atributos y métodos se heredan.
- **Subclase (Clase Hija):** Es la nueva clase que hereda de la superclase.
- **Palabra Clave `extendes`:** Se utiliza para indicar la relación de herencia entre una subclase y su superclase.

### Beneficios de la Herencia Simple

- **Reutilización de código:** Permite evitar duplicación de código al heredar atributos y métodos comunes de la superclase.
- **Extensibilidad:** Facilita la creación de nuevas clases basadas en clases existentes, lo que promueve la extensibilidad del software.
- **Organización y mantenimiento:** Mejora la organización del código al establecer jerarquías de clase y simplifica el mantenimiento al centralizar la lógica común en la superclase.
- **Relaciones "es-un":** La herencia simple modela relaciones "es-un" entre clases. Por ejemplo, un "Perro" es un "Animal".
- **Polimorfismo:** La herencia es un pilar fundamental para el polimorfismo, que permite tratar objetos de diferentes subclases de manera uniforme.

### Limitaciones

- Java no soporta la herencia múltiple de clases, lo que significa que una clase no puede heredar directamente de múltiples superclases.
- Para poder emular la herencia múltiple, Java utiliza interfaces.

**Ejemplo:**

```java
class Animal {
	void hacerSonido() {
		System.out.println("Sonido genérico de animal");
	}
}

class Perro extends Animal {
	void ladrar() {
		System.out.println("Guau, guau");
	}
}
```

- En este caso, `Perro` solo hereda de `Animal`.

## Herencia Múltiple

La herencia múltiple es cuando una clase puede heredar características (como atributos y métodos) de más de una clase padre.

En otras palabras, una clase hija puede tener más de una superclase directa.

### El Problema del Diamante

Uno de los mayores problemas de la herencia múltiple es el llamado "_Problema del diamante_". Esto sucede cuando una clase hereda de dos clases que, a su vez, heredan de una misma clase base. En este caso, puede haber confusión sobre cuál versión de un método o atributo debe usarse, ya que ambas clases padre podrían tenerlo.

Debido a las complicaciones y ambigüedades que puede generar, Java no admite la herencia múltiple de clases y así evitar errores y confusiones.

### Interfaces como Solución en Java

Aunque Java no permite que una clase herede de múltiples clases, sí permite que una clase implemente varias interfaces.

### Interfaces

Una [[Clase 03 - Herencia y Clases Abstractas#Interfaces|interfaz]] es como un contrato: define qué métodos debe tener una clase, pero no dice cómo se hacen (al menos hasta Java 8, donde se permiten algunos métodos con cuerpo).

Una clase puede implementar múltiples interfaces, lo que le permite heredar los contratos (métodos) definidos en esas interfaces.

Esto permite que una clase adopte múltiples comportamientos sin los problemas de ambigüedad asociados con la herencia múltiple de clases.

### Características de las Interfaces

- No contienen implementación de métodos (antes de Java 8).
- Pueden contener constantes
- Una clase puede implementar múltiples interfaces utilizando la palabra clave [[Clase 03 - Herencia y Clases Abstractas#`implements`|implements]].

**Ejemplo:**

```java
interface Caminante {
	void caminar();
}

interface Nadador {
	void nadar();
}

class Anfibio implements Caminante, Nadador {
	@Override
	public void caminar() {
		System.out.println("El anfibio camina");
	}
	
	@Override
	public void nadar() {
		System.out.println("El anfibio nada");
	}
	
	public class Main {
		public static void main(String[] args) {
			Anfibio anfibio = new Anfibio();
			anfibio.caminar();
			anfibio.nadar();
		}
	}
}
```

Aquí, ``Anfibio`` implementa dos interfaces (``Caminante`` y ``Nadador``), logrando un comportamiento similar a la herencia simple.

## Identificación de la Jerarquía entre Clase Base y Derivadas

En la POO, las clases siguen una estructura jerárquica, en la que:

- Clase base (superclase o padre): Es la clase original de la que otras clases obtienen sus características. Contiene atributos y métodos comunes que pueden ser compartidos con otras clases.
- Clases derivadas (subclases o hijas): Es la clase que hereda de la clase base. Puede usar todo lo que tiene la clase padre, y también puede agregar nuevas funciones o cambiar el comportamiento de lo que heredó.

**Ejemplo:**

```java
class Vehiculo {
	void mover() {
		System.out.println("El vehículo se mueve");
	}
}

class Coche extends Vehiculo {
	void acelerar() {
		System.out.println("El coche acelera");
	}
}

class Moto extends Vehiculo {
	void hacerCaballito() {
		System.out.println("La moto hace un caballito");
	}
}
```

**Jerarquía visual:**

![[Drawing 2025-09-08 19.04.45.excalidraw]]

En este caso:

- ``Vehiculo`` es la superclase.
- ``Coche`` y ``Moto`` son subclases de ``Vehiculo``.

## Redefinición de Métodos en Clases Hijas (`@Override`)

Cuando una clase hija necesita modificar el comportamiento de un método heredado de su clase base, puede hacerlo mediante la **sobreescritura de métodos.** Para indicar que un método está siendo redefinido, se utiliza la anotación `@Override`.

**Ejemplo:**

```java
class Animal {
	public void hacerSonido() {
		System.out.println("Sonido genérico de animal");
	}
}

class Gato extends Animal {
	@Override
	public void hacerSonido() {
		System.out.println("Miau, miau");
	}
}

public class Main {
	public static void main(String[] args) {
		Animal miGato = new Gato();
		miGato.hacerSonido(); // Miau, miau
	}
}
```

## Polimorfismo: Un Objeto, Múltiples Comportamientos

El polimorfismo permite que un mismo objeto pueda comportarse de diferentes maneras según el contexto en el que se utilice.

**Ejemplo:**

```java
class Vehiculo {
	public void acelerar() {
		System.out.println("El vehículo está acelerando");
	}
}

class Coche extends Vehiculo {
	@Override
	public void acelerar() {
		System.out.println("El coche acelera rápidamente");
	}
}

class Bicicleta extends Vehiculo {
	@Override
	public void acelerar() {
		System.out.println("La bicicleta acelera lentamente");
	}
}

public class Main {
	public static void main(String[] args) {
		Vehiculo miVehiculo = new Coche();
		miVehiculo.acelerar();
		
		miVehiculo = new Bicicleta();
		miVehiculo.acelerar();
	}
}
```

- **Herencia:**
	- Las clases `Coche` y `Bicicleta` heredan de la clase base `Vehiculo`. Esto establece una relación "es-un" (un `Coche` es un `Vehiculo`, una `Bicicleta` es un `Vehiculo`).
- **Sobreescritura de métodos (`@Override`):**
	- Cada subclase (`Coche` y `Bicicleta`) proporciona su propia implementación del método `acelerar()`, sobrescribiendo el método definido en la clase base. Esto permite que cada tipo de vehículo acelera de manera diferente.
- **Referencia de la superclase:**
	- En el método `main()`, se utiliza una referencia de la superclase (``Vehiculo miVehiculo``) para referirse a objetos de las subclases (`Coche` y `Bicicleta`).
- **Comportamiento dinámico:**
	- Cuando se llama al método `miVehiculo.acelerar()`, el comportamiento real del método que se ejecuta depende del tipo de objeto al que apunta la referencia `miVehiculo` en tiempo de ejecución. Esto es polimorfismo en acción.

### Explicación del Comportamiento

1. `Vehiculo miVehiculo = new Coche();`
	1. Aquí, `miVehiculo` se refiere a un objeto de la clase `Coche`.
	2. Cuando se llama a `miVehiculo.acelerar()`, se ejecuta la implementación del método `acelerar()` definida en la clase `Coche`.
2. `miVehiculo = new Bicicleta();`
	1. Ahora, `miVehiculo` se refiere a un objeto de la clase `Bicicleta`.
	2. Cuando se llama a `miVehiculo.acelear()` de nuevo, se ejecuta la implementación del método `acelerar()` definida en la clase `Bicicleta`.
