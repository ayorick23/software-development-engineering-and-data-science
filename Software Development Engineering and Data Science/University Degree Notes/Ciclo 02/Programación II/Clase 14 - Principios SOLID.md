---
Fecha de creación: 2025-10-13T17:53:00
Materia:
  - Programación II
Fecha de clase: 2025-10-13
---
# Principios SOLID

## **S** - Single Responsibility Principle (SRP)

>_"Una clase debe tener una única razón para cambiar."_

### ¿Qué Significa?

Cada clase debe estar enfocada en una sola responsabilidad o funcionalidad dentro del sistema. Lo que nos da como resultado que el sistema sea más fácil de mantener, probar y extender, ya que cada componente tiene un propósito claro y aislado.

**Beneficios:**

- Mayor cohesión.
- Facilidad para modificar una parte sin afectar el resto.
- Mejora en la legibilidad y pruebas unitarias.

**Mal ejemplo (violando SRP):**

```java
public Class Empleado {
	public String nombre;
	public double salario;
	
	public void calcularSalario() {/*P lógica*/}
	public void generarReporte() {/*r lógica de PDF*/}
	public void guardarEnBD() {/*lógica de persistencia*/}
}
```

**Problemas:**

Esta clase tiene demasiado: cálculo, reporte y persistencia. Cambiar el formato del reporte, por ejemplo, afectaría a la clase, aunque el negocio no haya cambiado.

**Buen ejemplo (aplicando SRP):**

```java
public Class Empleado {
	public String nombre;
	public double salario;
	public double calcularSalario() { return salario * 1 .15; }
}

public Class ReporteEmpleado {
	public void generarPDF(Empleado emp) {/*lógica PDF*/}
}

public Class RepositorioEmpIeado {
	public void guardar(Empleado emp) {/*persistencia*/}
}
```

**Explicación:**

Cada clase tiene solo una responsabilidad:

- ``Empleado`` gestiona los datos y cálculos relacionados con eI empleado.
- ``ReporteEmpIeado`` se encarga de la generación de reportes.
- ``RepositorioEmpIeado`` gestiona el almacenamiento.

Esto nos ayuda a que los cambios aplicados en la lógica de almacenamiento o del reporte **no afecten** la lógica de cálculo del salario, facilitando el mantenimiento y la extensibilidad del sistema.

## **O** - Open/Closed Principle (OCP)

>_"Las entidades de software deben estar abiertas para extensión, pero cerradas para modificación."_

### ¿Qué significa?

Una vez que se haya probado y validado una clase, no se tendría que modificar para poder agregar comportamientos nuevos. En lugar de modificar código existente, se debe extender por medio de herencia o composición.

**Beneficios:**

- Reduce errores al no aplicar cambios código probado y validado.
- Mejora la escalabilidad.
- Nos ayuda a poder agregar nuevas funcionalidades sin la necesidad de romper o modificar las existentes.

**Mal ejemplo (violando OCP):**

```java
public Class Impresora {
	public void imprimir(String tipo) {
		if (tipo.equals("PDF")) {
			// lógica de impresión PDF
		} else if (tipo.equals("Texto")) {
			// lógica de impresión de texto
		}
	}
}
```

**Problemas:**

Cada vez que se desee añadir un nuevo tipo de documento, se debe modificar el método ``imprimir()`` de la clase ``Impresora``, lo que se convierte en un riesgo en sistemas grandes.

**Buen ejemplo (aplicando OCP con polimorfismo):**

```java
public interface Documento {
	void imprimir();
}

public Class PDF implements Documento {
	public void imprimir() { lógica de impresión PDF }
}

public Class Texto implements Documento {
	public void imprimir() { lógica de impresión de texto }
}

public Class Impresora {
	public void imprimirDocumento(Documento doc) {
		doc.imprimir();
	}
}
```

**Explicación:**

Al establecer una interfaz ``Documento``, cada tipo de documento (PDF, Texto) se implementa de forma independiente. Ahora, si se requiere agregar más tipos de documentos, únicamente debemos agregar nuevas clases que implementen ``Documento`` sin modificar la clase ``Impresora``.

## **L** - Liskov Substitution Principle (LSP)

>_"Los objetos de una subclase deben poder sustituir a objetos de su superclase sin alterar el comportamiento esperado."_

### ¿Qué Significa?

Cuando hacemos uso de herencia, debemos estar seguros de que las subclases puedan ser usadas como si fueran instancias de la clase base sin modificar el comportamiento esperado del sistema. Si al reemplazar una instancia de la clase base por una subclase el programa falla, estamos violando este principio.

**Beneficios:**

- Coherencia: EI comportamiento del sistema re realizar de manera predecible.
- Flexibilidad. Es posible sustituir instancias de clases base por sus subclases sin ingresar o generar errores.

**Mal ejemplo (violando LSP):**

```java
Class Animal {
	public void hacerSonido() {}
}

Class Perro extends Animal {
	public void hacerSonido() {
		System.out.printIn("Guau");
	}
}

Class Pez extends Animal {
	public void hacerSonido() {
		throw new hago sonido");
	}
}
```

**Problemas:**

La clase ``Pez`` genera una excepción cuando se invoca ``hacerSonido()``, lo que viola el principio de sustitución de Liskov, ya que ``Pez`` no debería romper la interfaz de ``Animal``.

**Buen ejemplo (respetando LSP):**

```java
interface Sonido {
	void hacerSonido();
}

Class Perro implements Sonido {
	public void hacerSonido() {
		System.out.println("Guau");
	}
}

Class Gato implements Sonido {
	public void hacerSonido() {
		System.out.println("Miau");
	}

Class Pez implements Sonido {
	public void hacerSonido() {
		System,out.println("Blub");
	}
}
```

**Explicación:**

Ahora, tanto ``Perro``, ``Gato`` como ``Pez`` implementan la interfaz ``Sonido``, lo que permite que todos se comporten de forma adecuada y consistente, de esta manera sea posible sustituir sin problemas.

## **I** - Interface Segregation Principle (ISP)

>_"Los clientes no deben estar forzados a depender de interfaces que no usan."_

### ¿Qué Significa?

En lugar de definir una interfaz extensa que imponga a una clase la implementación de métodos irrelevantes, es más adecuado estructurar múltiples interfaces específicas y enfocadas. Esto permite que cada clase implemente únicamente aquellos métodos que le resultan pertinentes para su funcionalidad.

**Beneficios:**

- Desacoplamiento: Las clases solo dependen de lo que realmente usan.
- Cohesión: Las interfaces se centrar en un conjunto pequeño y coherente de funcionalidades.

**Mal ejemplo (violando ISP):**

```java
interface Trabajador {
	void programar();
	void diseñar();
	void testear();
}

class Tester implements Trabajador {
	public void programar() {
		// lógica
	}
	public void diseñar() {
		// lógica
	}
	public void testear() {
		// lógica
	}
}
```

**Problemas:**

La clase ``Tester`` está forzada a implementar métodos (programar y diseñar) que no utiliza.

**Buen ejemplo:**

```java
interface Programador {
	void programar();
}

interface Diseniador {
	void diseñar();
}

interface Tester {
	void testear();
}
```

**Explicación:**

Ahora, cada tipo de trabajador implementa únicamente los métodos necesarios para su rol. Lo que minimiza el acoplamiento y mejora la legibilidad y mantenimiento del código.

## **D** - Dependency Inversion Principle (DIP)

>_"Las clases de alto nivel no deben depender de clases de bajo nivel. Ambas deben depender de abstracciones."_

### ¿Qué Significa?

Este principio nos indica que las clases de alto nivel no deben depender de las implementaciones concretas de las clases de bajo nivel, sino de abstracciones. De igual forma, las clases de bajo nivel deben depender de las abstracciones, no de las clases de alto nivel.

**Beneficios:**

- **Desacoplamiento:** Las clases de nivel superior permanecen independientes de las modificaciones realizadas en clases de nivel inferior, lo que favorece una arquitectura más estable y robusta.
- **Flexibilidad:** Es posible sustituir o modificar la implementación de una clase sin provocar impactos en las demás clases que dependen de ella, facilitando la adaptabilidad del sistema.
- **Mantenibilidad:** Promueve la reutilización de clases a través de interfaces y la inyección de dependencias.

**Mal ejemplo (violando DIP):**

```java
class ConexionMySQL {
	public void conectar() {
		// lógica de conexión a MySQL
	}
}

class Servicio {
	ConexionMySQL conexion = new ConexionMySQL();
	public void usarDB() {
		conexion.conectar();
	}	
}
```

**Problemas:**

``Servicio`` depende directamente de una implementación concreta (``ConexionMySQL``). Si cambiamos a otro tipo de base de datos, debemos modificar la clase ``Servicio``.

**Buen ejemplo (respetando DIP):**

```java
interface ConexionDB {
	void conectar();
}

class ConexionMySQL implements ConexionDB {
	public void conectar() {
		// lógica de conexión a MySQL
	}
}

class Servicio {
	private ConexionDB conexion;
	
	public Servicio(ConexionDB conexion) {
		this.conexion = conexion;
	}
	
	public void usarDB() {
		conexion.conectar();
	}
}
```

**Explicación:**

Ahora, ``Servicio`` depende de la abstracción ``ConexionDB`` y no de una implementación concreta. Podemos inyectar cualquier implementación de ``ConexionDB`` sin cambiar la clase ``Servicio``.

## Optimización del Código

La optimización del código no es simplemente hacer las coas más rápido, sino de mejorar su eficiencia y minimizar el uso de recursos, manteniendo al mismo tiempo la claridad y mantenibilidad del código. Aquí veremos algunas estrategias para optimizar tu código:

### Estrategias de Optimización

1. **Evitar duplicación de código**
	- Utilizar funciones auxiliares para evitar el mismo código varias veces.
	- Aplica herencia o composición para reutilizar funcionalidades comunes.
2. **Elegir estructuras de datos adecuadas**
	- Elige la estructura de datos que se adapte mejor a tus necesidades. Por ejemplo:
		- ``ArrayList`` para acceso rápido por índice.
		- ``LinkedList`` cuando las inserciones y eliminaciones sean más frecuentes.
		- ``HashMap`` para búsquedas rápidas.
		- ``TreeMap`` si necesitas mantener los elementos ordenados.
3. **Reducir anidamiento**
	- Usa ``early return`` para reducir el nivel de anidamiento y hacer el código más legible.
	- Utiliza polimorfismo para evitar estructuras de control complejas como ``if`` o ``switch``.
4. **Evitar operaciones costosas dentro de bucles**
	- Evita hacer consultas a la base de datos, leer archivos o realizar cálculos complejos dentro de bucles.
5. **Aplicar lazy loading y caching**
	- Carga datos solo cuando realmente se necesiten, no de manera anticipada.
	- Usa caching para almacenar resultados de operaciones costosas y evitar repetirlas.
6. **Usar streams (en Java 8+) para operaciones sobre colecciones**

```java
List <String> nombres = lista.stream()
					.filter(n -> n.length() > 4)
					.map(String::toUpperCase)
					.collect(Collectors.toList());
```

**Explicación:**

Los **streams** permiten escribir operaciones de manera más concisa y, en muchos casos, más eficiente, especialmente para colecciones grandes. La operación ``filter`` elimina los elementos que no cumplen con la condición, mientras que ``map`` transforma los elementos. Finalmente, ``collect`` convierte el ``stream`` en una lista.
