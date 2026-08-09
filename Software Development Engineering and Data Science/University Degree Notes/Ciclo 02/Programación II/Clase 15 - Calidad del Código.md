---
Fecha de creación: 2025-10-21T18:17:00
Materia:
  - Programación II
Fecha de clase: 2025-10-21
---
[[Clase 14 - Principios SOLID|← Clase anterior]]

# Técnicas para Mejorar Eficiencia, Legibilidad y Mantenibilidad

Desarrollar estas habilidades es muy importante para:
- Optimizar el trabajo en equipo.
- Minimizar los tiempos de depuración.
- Facilitar las nuevas implementaciones.
- Aumentar la calidad percibida por los usuarios y clientes.

## Evitar Código Duplicado

### DRY: "Don't Repeat Yourself"

- Cada pieza de conocimiento o comportamiento debe tener **una sola representación** en el sistema.
- El código duplicado incrementa el riesgo de inconsistencias: si se corrige un bug en un lugar, pero se olvida en otro, surgen errores inesperados.

**Ejemplo de mal diseño (duplicación):**

```java
if (order.getStatus().equals("Pending")) {
	sendEmail(order.getCustomerEmail());
}
...
if (order.getStatus().equals("Pending")) {
	logPendingOrder(order);
}
```

**Mejor diseño:**

```java
if (isOrderPending(order)) {
	sendEmail(order.getCustomerEmail());
	logPendingOrder(order);
}

private boolean isOrderPending(Order order) {
	return "Pending".equals(order.getStatus());
}
```

## Métodos Pequeños y de Única Responsabilidad

**Un método = Una responsabilidad**

- Métodos muy extensos dificultan la comprensión del flujo lógico.
- Dividir ayuda a testear y reutilizar partes pequeñas.

**Consejos prácticos:**

- Métodos ideales: entre 5 y 20 líneas.
- Si un método hace más de una cosa, **divídelo**.

## Nombrado Significativo

**Nombres son contratos.**

- Si un nombre es ambiguo, el lector puede tener dudas sobre el verdadero funcionamiento.
- Elimina nombres como ``temp``, ``data``, ``doStuff``.

**Ejemplo:**

```java
// Malo
public double c(double b) {
	return b * 3.1416;
}

// Mejor
public double calcularAreaCirculo(double radio) {
	return radio * 3.1416;
}
```

## Control de Estructuras Anidadas

Evitar estructuras con 3 o más niveles de anidación (if, for, while) porque:

- Aumentan el "costo cognitivo".
- Hacen difícil seguir el flujo de ejecución.

**Ejemplo de anidación excesiva:**

```java
if (cliente != null) {
	if (cliente.getDireccion() != null) {
		if (cliente.getDireccion().getPais().equals("El Salvador")) {
			procesarEnvio(cliente);
		}
	}
}
```

**Forma mejorada usando _guard clauses_:**

```java
if (cliente == null || cliente.getDireccion() == null) return;

if ("El Salvador".equals(cliente.getDireccion().getPais())) {
	procesarEnvio(cliente);
}
```

## Selección Adecuada de Colecciones

Comprender y aplicar la colección correcta en Java puede optimizar muchísimo el rendimiento:

| Colección      | Uso Ideal                                        |
| -------------- | ------------------------------------------------ |
| ``ArrayList``  | Acceso rápido por índice, pocos inserts/removals |
| ``LinkedList`` | Muchas inserciones/borrados intermedios          |
| ``HashSet``    | Búsqueda rápida, no permite duplicados           |
| ``HashMap``    | Asociación de claves únicas con valores          |

**Tips adicionales:**

- Siempre inicializar colecciones con un tamaño estimado si es posible (``new ArrayList<>(100)``).
- Evitar operaciones costosas como ``contains()`` en listas grandes, mejor usar ``Set``.

## Minimizar Operaciones Costosas en Bucles

Realizar cálculos pesados, ejecutar consultas a bases de datos o abrir archivos dentro de bucles puede matar el rendimiento.

**Mal Ejemplo:**

```java
for (Producto p:productos) {
	int stock = consultarStockBaseDatos(p.getId()); // Cada iteración va a la BD
	if (stock > 0) {
		...
	}
}
```

**Mejor:**

```java
Map<Integer, Integer> stockProductos = consultarStockMasivo(productos);

for (Producto p:productos) {
	int stock = stockProductos.getOrDefault(p.getId(), 0);
	if (stock > 0) {
		...
	}
}
```

## Uso Eficiente de Cadenas de Texto

**Concatenar con `+` en un bucle es muy ineficiente.**

**Ejemplo Malo:**

```java
String result = "";
for (Strin s:lista) {
	result += s; // Cada vez crea un nuevo objeto String
}
```

**Ejemplo Bueno:**

```java
StringBuilder result = new StringBuilder();
fro (String s:lista) {
	result.append(s);
}
```

## Aplicación de Patrones de Diseño

Los **Patrones de Diseño** son soluciones probadas a problemas recurrentes en la arquitectura de software.

**Ejemplos:**

- **Singleton:** Garantiza una única instancia global.
- **Factory:** Creación controlada de objetos.
- **Strategy:** Permitir intercambiar algoritmos en tiempo de ejecución.
- **Observer:** Notifica de forma automática cambios a otros objetos.

Adoptarlos correctamente aumente:

- Reusabilidad
- Testabilidad
- Adaptibilidad

# Herramientas para Analizar Calidad de Código

No basta con la revisión manual. El análisis automático del código:

- Detecta errores humanos.
- Estandariza la calidad.
- Evita problemas de seguridad y rendimiento.

## SonarQube

- Detecta bugs en tiempo oportuno.
- Analiza cobertura de pruebas, duplicación de código, complejidad ciclomática.
- Se puede configurar de manera personalizada las reglas.
- Es de fácil integración en Jenkins, GitHub Actions, GitLab, Azure DevOps.

**Concepto Clave: Deuda Técnica**

Tiempo estimado para corregir problemas  de calidad en el proyecto.

## Checkstyle

- Valida el cumplimiento de estándares en el código: tamaño de clases, nombres, formato, espaciado.
- Factibilidad de integración con IDEs como Eclipse, IntelliJ IDEA.
- Obliga al equipo a seguir un estilo coherente (lo cual es vital en proyectos grandes).

## PMD

Detecta:

- Duplicación de código.
- Variables no inicializadas.
- Sentencias ``switch`` sin ``default``.
- Bucles innecesarios.

Ejemplos de reglas útiles:

- **Optimizar bucles:** ``for`` simples en vez de ``while``.
- **Variables que pueden ser finales:** aumenta la inmutabilidad y seguridad.

## Refactorización para Reducir Complejidad

Refactorizar de manera consciente permite:

- Optimiza el diseño sin modificar el comportamiento.
- Facilitar futuras funcionalidades.
- Minimiza deuda técnica.

### Técnicas de Refactorización Comunes

| Técnica                 | Ejemplo                                        |
| ----------------------- | ---------------------------------------------- |
| Extraer Métdo           | Obtener parte del código a un nuevo método     |
| Renombrar               | Mejorar nombres ambiguos                       |
| Encapsular Campos       | Utilizar ``private`` y ``getters``/``setters`` |
| Eliminar Código Muerto  | Borrar funciones no usadas                     |
| Simplificar Condiciones | Usar guard caluses                             |

## Principios SOLID en Refactorización

Los [[Clase 14 - Principios SOLID#Principios SOLID|principios SOLID]] guían la evolución saludable del código.

1. [[Clase 14 - Principios SOLID#**S** - Single Responsibility Principle (SRP)|Single Responsability Principle (SRP)]]
2. [[Clase 14 - Principios SOLID#**O** - Open/Closed Principle (OCP)|Open/Closed Principle (OCP)]]
3. [[Clase 14 - Principios SOLID#**L** - Liskov Substitution Principle (LSP)|Liskov Substitution Principle (LSP)]]
4. [[Clase 14 - Principios SOLID#**I** - Interface Segregation Principle (ISP)|Interface Segregation Principle (ISP)]]
5. [[Clase 14 - Principios SOLID#**D** - Dependency Inversion Principle (DIP)|Dependency Inversion Principle (DIP)]]

### Caso de Estudio de Refactorización

**Código Original:**

```java
if (user != null) {
	if (user.getAge() > 18) {
		if (user.isActive()) {
			System.out.println("Usuario válido");
		}
	}
}
```

**Refactorizado:**

```java
if (esUsuarioValido(user)) {
	System.out.println("Usuario válido");
}

private boolean esUsuarioValido(User user) {
	return user != null && user.getAge() > 18 && user.isActive();
}
```

**Mejoras:**

- **Legibilidad:** Ahora queda claro lo que se está validando.
- **Reutilización:** ``esUsuarioValido()`` puede ser usada en más partes del código.
- **Mantenibilidad:** Cambiar la condición en un solo lugar.
