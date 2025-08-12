---
Fecha de creación: 2025-08-11T18:13:00
Materia:
  - Programación II
Fecha de clase: 2025-08-11
---
# Colecciones en Java
(similar a [[Data Structures and Algorithms]])

Java nos ofrecen colecciones dinámicas, estas colecciones son estructuras que permiten almacenar y manipular un conjunto de datos de manera flexible y eficiente, a diferencia de los [[Clase 04 - Entrada de Datos y Arrays#Array en Java|arrays]] tradicionales. Las colecciones no tienen un tamaño fijo definido cuando declaramos una variable de tipo colección. Las colecciones dinámicas pueden cambiar su tamaño cuando el programa está en tiempo de ejecución, por lo cual estas estructuras son recomendadas cuando tenemos que almacenar datos cuando no sabemos que tendremos que manipular.

La **Java Collections Framework (JFC)** nos ofrece varias implementaciones para trabajar con colecciones dinámicas, las clases mas comunes que implementan colecciones son:
- ArrayList
- HashSet
- HashMap

## ArrayList: Lista Dinámica

El ArrayList es una de las colecciones más populares que Java ofrece, es utilizada para almacenar una lista dinámica de elementos. La diferencia con los arreglos estáticos es que un ArrayList puede crecer y reducir su tamaño en tiempo de ejecución según se agreguen o eliminen elementos.

### Características del ArrayList

- **Tamaño variable:** A medida que agregamos elementos, el tamaño de la lista se adaptará automáticamente.
- **Rápido acceso:** Nos permite acceder a los elementos por índice lo que lo hace familiar a los arreglos comunes.
- **Permite duplicados:** No restringe los elementos duplicados, por lo que podemos tener instancias del mismo objeto dentro del ArrayList.

### Operaciones Comunes con ArrayList

- **Agregar elementos:** Permite agregar elementos con el método ``add()``, este método agrega el elemento al final de la colección.
- **Eliminar elementos:** Permite eliminar elementos con el método ``remove()``, o limpiar la lista completamente con ``clear()``.
- **Buscar elementos:** Java nos ofrece métodos como el ``contains()`` que nos permite saber si un elemento está presente en el ArrayList, también tenemos el ``indexOf()``, que nos devuelve el índice de la primera aparición de un elemento.

**Ejemplo:**

```java
import java.util.ArrayList;

public class EjemploArrayList {
	public static void main(String[] args) {
		ArrayList<String> frutas = new ArrayList<>();
		// Agregar elementos
		frutas.add("Manzana");
		frutas.add("Banana");
		frutas.add("Naranja");

		// Mostra lista
		System.out.println("Frutas: " + frutas); // [Manzana, Banana, Naranaja]

		// Verificar si contiene un elemento
		boolean contieneBanana = frutas.contains("Banana"); // true

		// Obtener el elemento en una posición específica
		String frutaEnIndice = frutas.get(1);
		System.out.println("Fruta en índice 1: " + frutaEnIndice); // Banana

		// Eliminar un elemento
		frutas.remove("Naranja");
		System.out.println("Frutas después de eliminar Naranja: " + frutas); // [Manzana, Banana]
	}
}
```

### Cuando Usar ArrayList

- Cuando necesitas una lista ordenada.
- Si se requiere poder tener elementos duplicados.
- Si se necesita acceso rápido a los elementos por índice.

## HashSet

El HashSet es una implementación de la interfaz ``set`` que Java proporciona, esta se usa cuando no necesitamos que los elementos en la colección estén ordenados, además no queremos permitir elementos duplicados.

### Características del HashSet

- **Elementos únicos:** No permite que los elementos de la colección tengan elementos repetidos, si agregas un elemento que ya existe no se agregará.
- **No mantienen el orden:** Los elementos no se almacenan en el orden en que se agregan, la colección se basa en una tabla Hash, lo que mejora la velocidad de las búsquedas en la colección.
- **Acceso rápido:** Esta estructura su programación interna permite que la búsqueda de elementos sea más eficiente.

### Operaciones Comunes con HashSet

Se pueden implementar los mismo métodos que con los ArrayList.

**Ejemplo:**

```java
import java.util.HashSet;

public class EjemploHashSet {
	public static void main(String[] args) {
		HashSet<String> frutas = new HashSet<>();
		// Agregar elementos
		frutas.add("Manzana");
		frutas.add("Banana");
		frutas.add("Naranja");

		// Intentar agregar un elemento duplicado
		frutas.add("Manzana"); // No se agregará

		// Mostrar conjunto
		System.out.println("Frutas: " + frutas); // [Manzana, Banana, Naranja]

		// Verificar si contiene un elemento
		boolean contieneNaranaja = frutas.contains("Naranja");
		System.out.println("Contiene Naranaja: " + contieneNaranaja); // true

		// Eliminar un elemento
		frutas.remove("Banana");
		System.out.println("Frutas después de eliminar Banana: " + frutas) // [Manzana, Naranja]
	}
}
```

### Cuando Usar HashSet

- Cuando necesitamos garantizar que no tendremos elementos duplicados.
- Si el orden de los elementos no importa.
- Para realizar búsquedas más rápidas en colecciones de elementos únicos.

## HashMap: Mapa de Claves y Valores

Un HashMap es una estructura que se utiliza en Java para almacenar pares de datos, cada dato está compuesto de una clave y un valor, esta estructura es útil cuando se necesita asociar un valor a una única clave.

### Características de HashMap

- **Claves únicas:** Cada clave en un HashMap es única, pero los valores asociados pueden repetirse.
- **Acceso rápido:** La búsqueda de un valor a partir de su clave es muy eficiente.
- **No garantiza el orden:** El orden de los elementos no está garantizado, debido a que no depende de la interna de la tabla hash.

### Operaciones Comunes en HashMap

- **Agregar pares clave-valor:** Para agregar un par usamos el método ``put()``.
- **Eliminar pares clave-valor:** Para eliminar un par clave-valor usamos el método ``remove()``, este nos permite eliminar un par, para borrar todos los pares usamos ``clear()``.
- **Buscar elementos:** Usamos ``get()`` para obtener el valor asociado con una clave, o ``containsKey()`` para verificar si una clave está presente en el mapa.

## Diferencia entre Listas Estáticas y Dinámicas

Los arreglos estáticos tienen un tamaño fijo que se define al momento de su creación. Esto significa que no se pueden agregar o eliminar sin crear un nuevo arreglo. Por otro lado, las colecciones dinámicas como ArrayList, HashSet y HashMap pueden ajustarse automáticamente según se agregan o eliminan elementos.

### Ventajas de las Colecciones Estáticas

- Mejor rendimiento al no tener que manejar el redimensionamiento de la estructura.
- Menor sobrecarga de memoria.

### Ventajas de las Colecciones Dinámicas

- Flexibilidad para trabajar con una cantidad variable de elementos.
- Facilitan la gestión de colecciones cuyo tamaño no es conocido de antemano.

## Métodos de Búsqueda en Colecciones

- ``contains()``
El método ``contains()`` se utiliza para verificar si un elemento está presente en la colección. Devuelve ``true`` si el elemento existe, ``false`` si no lo encuentra.

- ``indexOf()``
``indexOf()`` devuelve el índice del primer elemento que coincide con el que le pasas como parámetro. Si no encuentra el elemento, devuelve -1.

- ``filter()`` (Java 8+)
``filter()`` es un método de los Streams en Java que permite seleccionar solo aquellos elementos que cumplan con una condición específica.

**Ejemplo:**

```java
import java.util.List;
import java.util.stream.Collectors;

public class EjemploFilter {
	public static void main(String[] args) {
		List<String> frutas = List.of("Manzana", "Banana", "Naranaja", "Mango");
		List<String> filtradas = frutas.stream()
							.filter(f -> f.startsWith("M"))
							.collect(Collectors.toList());
		System.out.println(filtradas) // [Manzana, Mango]
	}
}
```

# ¿Qué es UML?

El **Lenguaje de Modelado Unificado** (UML, por sus siglas en inglés) es un estándar para representar sistemas de software de manera visual. Fue desarrollado para proporcionar una notación común que facilite la comunicación entre desarrolladores, analistas y arquitectos de software.

UML permite diseñar y documentar sistemas mediante diagramas, los cuales describen diferentes aspectos del software, como su estructura, comportamiento e interacciones entre sus componentes.

## Diagramas UML

UML se compone de varios tipos de diagramas, que se dividen en dos grandes categorías:

- Diagramas Estructurales
- Diagramas de Comportamiento

### Diagramas Estructurales

Describen la organización y la estructura estática del sistema. Los más relevantes para el desarrollo con colecciones en Java son:

- Diagrama de clases
- Diagrama de objetos
- Diagrama de paquetes

### Diagramas de Comportamiento

Representan la dinámica del sistema y la interacción entre sus componentes. Entre ellos destacan:

- Diagrama de casos de uso
- Diagrama de secuencia
- Diagrama de actividad

### Diagrama de Clases

El diagrama de clases es el más importante dentro de UML, ya que representa la estructura estática de un sistema mostrando clases, atributos, métodos y sus relaciones.

#### Elementos Clave en un Diagrama de Clases

- **Clases:** Representadas por rectángulos con tres secciones:
	- Nombre de la clase
	- Atributos de la clase
	- Métodos de la clase
- **Relaciones** entre clases, como **asociaciones**, **herencia**, **composición** y **agregación**.

![[Drawing 2025-08-11 21.18.12.excalidraw]]

Este diagrama nos muestra que:

- La clase ``Estudiante`` tiene atributos nombre y edad.
- La clase ``Materia`` tiene nombre y código.
- Existe una relación de uno a muchos (``1...*``), donde un estudiante puede estar inscrito en varias materias.

En código Java, esto se implementaría así:

```java
import java.util.List;

class Estudiante {
	private String nombre;
	private int edad;
	private List<Materia> materias;

	public Estudiante(String nombre, int edad, List<Materia> materias) {
		this.nombre = nombre;
		this.edad = edad;
		this.materias = materias;
	}

	public String getNombre() {
		return nombre;
	}

	public int getEdad() {
		return edad;
	}

	public List<Materia> getMaterias() {
		return materias;
	}
}

class Materia {
	private String nombre;
	private String codigo;

	public Materia(String nombre, String codigo) {
		this.nombre = nombre;
		this.codigo = codigo;
	}

	public String getNombre() {
		return nombre;
	}

	public String getCodigo() {
		return codigo;
	}
}
```
