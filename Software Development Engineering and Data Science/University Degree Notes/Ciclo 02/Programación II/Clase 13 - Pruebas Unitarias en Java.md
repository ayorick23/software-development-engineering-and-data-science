---
Fecha de creación: 2025-10-06T18:05:00
Materia:
  - Programación II
Fecha de clase: 2025-10-06
---
[[Clase 11 - Repaso para Parcial|← Clase anterior]] | [[Clase 14 - Principios SOLID|Clase siguiente →]]

# Pruebas Unitarias en Java
(similar a [[Clase 15 - Pruebas Unitarias y Módulos]])

## ``JUnit``

``JUnit`` es un marco de pruebas para Java diseñado para escribir y ejecutar pruebas automatizadas de forma fácil.

Para cualquier desarrollador Java, esta es una herramienta esencial en su ecosistema de herramientas ya que se integra fácilmente en entornos de desarrollo como IntelliJ IDEA, Eclipse y NetBeans. Así mismo, puede utilizarse junto con herramientas de integración continua como Jenkins o GitHub Actions para ejecutar pruebas automáticamente con cada cambio en el código.

``JUnit`` utiliza anotaciones para identificar métodos de prueba y provee una serie de ``assertions`` para verificar los resultados esperados. Estas anotaciones ayudan a estructurar el código de prueba de forma clara y legible, separando la preparación del entorno, la ejecución de pruebas y la limpieza posterior.

Además, brinda soporte para pruebas parametrizadas, pruebas repetidas y pruebas condicionales, las cuales permiten una cobertura más amplia de los escenarios posibles. La comunidad de ``JUnit`` está muy activa y constantemente se añaden nuevas características para facilitar la escritura de pruebas más expresivas, efectivas y limpias.

### Anotaciones Comunes en ``JUnit``

- ``@Test``: Indica que el método es una **prueba unitaria**.
	- No debe tener parámetros.
	- Puede lanzar excepciones.
	- Se recomienda que cada prueba sea **determinista** (produce siempre el mismo resultado) y **rápida**.
	- Se puede usar junto a otras anotaciones como ``@DisplayName`` para personalizar el nombre mostrado.
- ``@BeforeEach``: Se ejecuta **antes de cada prueba** individual.
	- Se usa para **configurar** el entorno de prueba.
	- Ideal para **inicializar dependencias**, **crear mocks**, o reiniciar variables.
- ``@AfterEach``: Se ejecuta **después de cada prueba**.
	- Sirve para **realizar limpiezas**, como cerrar conexiones, borrar archivos temporales, etc.
- ``@BeforeAll``: Se ejecuta **una vez al principio, antes de todas las pruebas** de la clase.
	- Debe ser un método ``static``.
	- Útil para tareas **costosas en tiempo** que solo deben hacerse una vez: cargar configuración, conectarse a una base de datos, etc.
- ``@AfterAll``: Se ejecuta **una vez al final, después de todas las pruebas**.
	- También debe se ``static``.
	- Se utiliza para liberar recursos compartidos o cerrar conexiones persistentes.
- ``@Disabled:``
	- 
- ``@DisplayName``: Permite especificar un **nombre más descriptivo** para una prueba.
	- Mejora la legibilidad de los reportes.
	- Ejemplo: ``@DisplayName("Debe lanzar excepción si la entrada es nula");``.
- ``@Tag``: Permite clasificar pruebas bajo etiquetas.
	- Útil para ejecutar pruebas por grupos: ``@Tag("lento")``, ``@Tag("integración")``.
- ``@RepeatedTest(n)``: Ejecuta una prueba **n veces**.

**Ejemplo:**

```java
public class Calculadora {
	public int sumar(int a, int b) {
		return a + b;
	}
	
	public int multiplicar(int a, int b) {
		return a * b;
	}
	
	public int dividir(int a, int b) {
		if (b == 0) {
			throw new ArithmeticException("No se puede dividir por cero");
		}
		return a / b;
	}
	
	public boolean esMayorDeEdad(int edad) {
		return edad >= 18;
	}
	
	public String procesarSaludo(Servicio servicio, Usuario usuario) {
		return servicio.saludar(usuario);
	}
}
```

**Código del TEST:**

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

public class CalculadoraTest {
	
	@Test
	public void pruebaSuma() {
		Calculadora calc = new Calculadora();
		assertEquals(5, calc.sumar(2, 3));
	}
	
	@Test
	public void pruebaDivisionPorCero() {
		Calculadora calc = new Calculadora();
		assertThrows(ArithmeticException.class, () -> calc.dividir(10, 0));
	}
	
	@Test
	public void pruebaMultiplicacion() {
		Calculadora calc = new Calculadora();
		assertEquals(12, calc.multiplicar(3, 4));
	}
	
	@Test
	public void pruebaMayorDeEdad() {
		Calculadora calc = new Calculadora();
		assertTrue(calc.esMayorDeEdad(20));
		assertFalse(calc.esMayorDeEdad(17));
	}
}
```

En el ejemplo anterior, se validan tanto los resultados exitosos como las excepciones esperadas. Este enfoque permite cubrir diversos escenarios, probar entradas válidas e inválidas y prevenir errores lógicos o de ejecución. Las pruebas automatizadas permiten ejecutar validaciones continuamente sin intervención manual.

## ``Mockito``

``Mockito`` es una herramienta robusta para Java diseñada para facilitar la realización de pruebas unitarias. Su función principal consiste en generar objetos simulados, conocidos como ``mocks``, que imitan el comportamiento de objetos reales sin tener que instanciarlos o ejecutarlos por completo. Esta característica resulta especialmente beneficiosa cuando el código en prueba interactúa con elementos externos como bases de datos, servicios web, APIs de terceros o sistemas antiguos.

Al utilizar ``Mockito``, los desarrolladores pueden aislar un método que están probando del resto del sistema. Esto permite que las pruebas sean más **rápidas, predecibles, fáciles de escribir y mantener**, y con una mayor **independencia** respecto a la infraestructura.

### ¿Por Qué Usar Objetos Simulados?

Es muy común que, en programación orienta a objetos, las clases dependan de otras clases o servicios para cumplir sus responsabilidades. Por ejemplo, una clase ``ServicioDeUsuario`` podría depender de un repositorio de datos para obtener información de un usuario. Cuando queremos probar la lógica de negocio de esta clase, no es recomendable hacer llamadas reales a bases de datos o servicios externos por las siguientes razones:

- **Ralentiza las pruebas.**
- **Introduce variables incontrolables**, como errores de red o cambios en datos reales.
- **Dificulta reproducir errores** o probar condiciones específicas.
- **Rompe el principio de unidad**, que establece que una prueba unitaria debe validar *una unidad lógica* de forma aislada.

``Mockito`` soluciona estos problemas al permitirnos **simular el comportamiento** de las dependencias, configurando o que deben retornar, cuándo y cómo deben actuar.

### Ventajas de ``Mockito``

1. **Aislamiento total** del código bajo prueba.
2. **Velocidad:** las pruebas se ejecutan mucho más rápido.
3. **Control absoluto** sobre los valores retornados por las dependencias.
4. **Verificación de interacciones:** podemos confirmar que un método fue invocado con ciertos parámetros o cierto número de veces.
5. **Fácil integración con JUnit y otros frameworks de pruebas.**
6. **No requiere configuración pesada ni clases especiales.**

**Ejemplo:**

```java
import static org.mockito.Mockito.*;
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

public class ServicioTest {
	
	@Test
	public void pruebaSaludo() {
		// Crear un mock del objeto Usuario
		Usuario usuarioMock = mock(Usuario.class);
		
		// Configurar el comportamiento del mock
		when(usuarioMock.getNomber()).thenReturn("Carlos");
		
		// Crear instancia del servicio a probar
		Servicio servicio = new Servicio();
		
		// Ejecutar el método que depende del mock
		String saludo = servicio.saludar(usuarioMock);
		
		// Verificar el resultado esperado
		assertEquals("Hola, Carlos", saludo);
		
		// Verificar que se haya llamado al método getNombre
		verify(usuarioMock).getNombre();
	}
}
```

**¿Qué hace este ejemplo?**

- Simula una clase ``Usuario`` que tiene un método ``getNombre()``.
- Se configura el ``mock`` para que ``getNombre()`` retorne ``"Carlos"``, independientemente de su implementación real.
- Se prueba el método ``saludar()`` del servicio, que depende de ``Usuario``.
- Se verifica tanto el valor retornado como la interacción con el ``mock``.

### Verificaciones Adicionales con ``Mockito``

Además de simular valores de retorno, ``Mockito`` permite verificar interacciones con los ``mocks``. Algunos ejemplos:

```java
verify(servicio).procesar(usuarioMock); // Verifica que se llamó una vez
verify(servicio, times(3)).procesar(usuarioMock); // Verifica que se llampo 3 veces
verify(servicio, never()).eliminar(usuarioMock); // Verifica que never se llamó
verify(servicio).procesar(argThat(u -> u.getNombre().startsWith("C"))); // Verifica con condición
```

## Uso de ``assertions`` en ``JUnit``

``JUnit`` nos brinda múltiples métodos de ``assertions`` para verificar los resultados esperados. Estas sentencias permiten validar si el código funciona según se espera bajo ciertas condiciones.

- `assertEquals(expected, actual)`: Verifica que dos valores sean iguales.
- `assertTrue(condition)`/`assertFalse(condition)`: Verifica que una condición sea **verdadera**/**falsa**.
- `assertNull(value)`/`assertNotNull(value)`: Verifica que un objeto sea nulo o no nulo.
- `assertThrows(Exception.class, executable)`: Verifica que se lance una excepción específica.
- `assertAll`: Ejecuta múltiples aserciones agrupadas.

Estas ``assertions`` deben usarse estratégicamente para cubrir la lógica principal del método, casos límite (_boundary cases_) y condiciones inesperadas. Cuantas más pruebas se definan y validen correctamente, mayor será la cobertura y confiabilidad del código.

## Recomendaciones para Pruebas Efectivas

1. **Probar un solo caso por test** cada test debe validar una única responsabilidad.
2. **Usar nombres descriptivos** tanto para los métodos como para las variables en las pruebas.
3. **Preparar escenarios positivos y negativos** para asegurar que la aplicación maneje correctamente entradas válidas y errores.
4. **Utilizar mocks solo cuando sea necesario**, si puedes evitar mocks, mejor. No todo necesita ser simulado.
5. **Evitar lógica compleja dentro de los tests**, mantén los tests simples y legibles.
6. **Probar casos extremos y bordes como listas vacías**, valores nulos, números grandes o negativos.
7. **Automatizar la ejecución de pruebas** para incluirlas en los pipelines de integración continua.
