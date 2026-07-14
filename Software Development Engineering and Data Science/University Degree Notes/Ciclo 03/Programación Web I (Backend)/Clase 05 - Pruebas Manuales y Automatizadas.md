---
Fecha de creación: 2026-02-24T21:19:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-02-16
---
# Pruebas Manuales y Automatizadas

En el ciclo de vida del desarrollo de software, las pruebas cumplen un papel crítico. No basta con que un endpoint devuelva datos; es necesario comprobar que:

- Cumpla con las especificaciones.
- Valide correctamente los datos.
- Responda en diferentes escenarios.
- Mantenga un comportamiento consistente a lo largo del tiempo.

En el contexto de las APIs RESTful, las pruebas aseguran que el servicio pueda integrarse de manera confiable con aplicaciones móviles, web, de escritorio u otros servicios externos.

## Pruebas Manuales con Postman o Insomnia

**Postman** e **Insomnia** son clientes HTTP que permiten enviar solicitudes (``GET``, ``POST``, ``PUT``, ``DELETE``) a una API y analizar sus respuestas.

Son ampliamente usados en entornos profesionales para pruebas rápidas y documentación viva de las APIs.

### Ventajas

- Simples de usar.
- Permiten guardar colecciones de peticiones.
- Soportan autenticación (Basic, OAuth2, JWT).
- Se pueden automatizar pruebas con scripts internos (en Postman).

**Ejemplo de prueba con Postman:**

Supongamos que tenemos un endpoint: POST /api/productos

Body válido:

```json
{
	"nombre": "Laptop Lenovo",
	"precio": 1200,
	"stock": 15
}
```

Respuesta esperada:

```json
{
	"id": 1,
	"nombre": "Laptop Lenovo",
	"precio": 1200,
	"stock": 15
}
```

Si enviamos datos inválidos:

```json
{
	"nombre": "",
	"precio": -10,
	"stock": -5
}
```

Respuesta esperada:

```json
{
	"nombre": ["El nombre es obligatorio"],
	"precio": ["El precio debe ser mayhor que 0"],
	"stock": ["El stock no puede ser negativo"]
}
```

## Introducción a Pruebas Automatizadas

Son programas que verifican automáticamente que el código funciona como se espera. Una vez escritos, se ejecutan en segundos y garantizan que los cambios no rompan la aplicación.

### Beneficios

- Detectan errores antes de que lleguen a producción.
- Ahorran tiempo en comparación con pruebas manuales.
- Documentan el comportamiento esperado de la aplicación.
- Se integran con CI/CD (Integración Continua/Despliegue Continuo).

### Tipos de Pruebas

- **Unitarias:** verifican métodos o funciones individuales.
- **De integración:** prueban cómo interactúan distintos componentes (ej: controlador + base de datos).
- **End-to-End (E2E):** simulan el comportamiento del usuario final.

## Pruebas Automatizadas en .NET

En .NET existen varios frameworks de preubas:

- **xUnit** (más popular en .NET Core/8).
- **NUnit** (clásico y aún vigente).
- **MSTest** (propio de Microsoft).

### Instalación con xUnit

En un proyecto de API, podemos añadir pruebas con:

```bash
dotnet new xunit -m MiApi.Tests
dotnet add MiApi.Tests reference MiApi
```

**Ejemplo de prueba unitaria simple:**

```csharp
public cass Calculadora
{
	public int Sumar(int a, int b) => a + b;
}

using Xunit; // Importa el framework de pruebas xUnit

// Clase que contiene las pruebas unitarias para la clase Calculadora
public class CalculadoraTests
{
	// [Fact] indica que este método es una prueba unitaria independiente
	[Fact]
	public void Sumar_DosNumeros_ReteronaSumaCorrecta()
	{
		// ARRANGE (Preparar)
		// Se crea una instancia de la clase que se quiere probar
		var calc = new Calculadora();
		
		// ACT (Actuar)
		// Se ejecuta el método Sumar con los números 2 y 3
		var resultado = calc.Sumar(2, 3);
		
		// ASSERT (Verificar)
		// Verifica que el resultado devuelto sea 5
		// Si resultado == 5, la prueba pasa
		// Si resultado != 5, la prueba falla
		Assert.Equal(5, resultado);
	}
}
```

- Es una clase que contiene **casos de prueba** para la clase ``Calculadora``.
- Por convención, las clases de prueba se nombran con el sufijo ``Tests``.
- ``[Fact]`` indica que el método es una **prueba unitaria** que ``xUnit`` debe ejecutar.
- EI nombre del método describe lo que se prueba:
	- ``Sumar_DosNumeros_RetornaSumaCorrecta`` cuando sumamos dos números, esperamos que retorne la suma correcta.

### Prueba de un Controlador de API

Supongamos que tenemos un ``ProductosController``. Podemos probar que crear un producto válido devuelve ``201 Created``:

```csharp
using Microsoft.AspNetCore.Mvc;
using MiApi.Data;
using MiApi.Models;

namespace MiApi.Controllers
{
	[ApiController]
	[Route("api/[controller]")]
	public class ProductosController : ControllerBase
	{
		private readonly AppDbContext _context;
		
		public ProductosController(AppDbContext context)
		{
			_context = context;
		}
		
		// POST: api/productos
		[HttpPost]
		public IActionResult CrearProducto([FromBody] Producto producto)
		{
			if (producto == null)
			{
				return BadRequest();
			}
			
			_context.Productos.Add(producto);
			_context.SaveChanges();
			
			// Retorna 201 Created con el producto creado
			return CreatedAtAction(nameof(CrearProducto), new { id = producto.Id }, producto);
		}
	}
}
```

**Clase ``Producto``:**

```csharp
using System.ComponentModel.DataAnnotations;

namespace MiApi.Models
{
	public class Producto
	{
		[Key]
		public int Id { get; set; }         // Clave primaria
		public string Nombre { get; set; }  // Nombre del producto
		public decimal Precio { get; set; } // Precio unitario
		public int Stock { get; set; }      // Cantidad en inventario
	}
}
```

Aquí estamos usando InMemoryDatabase de EF Core para simular la BD sin necesidad de SQL Server.

```csharp
using Xunit;
using System;
using MiApi.Data;
using Microsoft.EntityFrameworkCore;
using MiApi.Controllers;
using MiApi.Models;
using Microsoft.AspNetCore.Mvc;

public class ProductosControllerTests
{
	// Método auxiliar que crea un DbContext en memoria para las pruebas
	private AppDbCOntext GetDbContext()
	{
		var options = new DbContextOptionsBuilder<AppDbContext>().UseInMemoryDatabase(databaseName: Guid.NewGuid(ToString)) // Nuevo DB para cada test
		
		return new AppDbContext(options);
	}
	
	[Fact]
	public void CrearProducto_ProductoValido_RetornaCreated()
	{
		// ARRANGE
		var context = GetDbContext(); // Base de datos en memoria
		var controller = new ProductosController(context);
		var producto = new Producto { Nombre = "Mouse", Precio = 20, Stock = 5 }
		// ACT
		var resultado = controller.CrearProducto(producto) as CreatedAtActionResult;
		
		// ASSERT
		Assert.NotNull(resultado); // Verifica que se haya devuelto un CreatedAtActionResult
		var productoCreado = resultado.Value as Producto;
		Assert.NotNull(productoCreado); // Verifica que el producto no sea nulo
		Assert.Equal("Mouse", productoCreado.Nombre); // Verifica el nombre
		Assert.Equal(20, productoCreado.Precio); // Verifica el precio
		Assert.Equal(5, productoCreado.Stock); // Verifica el stock
	}
}
```

## Conceptos Básicos de Escalabilidad

Aunque el foco está en pruebas, es importante introducir cómo las pruebas ayudan a sistemas escalables.

### Escalabilidad en cliente-servidor

- Escalabilidad vertical: aumentar recursos de un servidor.
- Escalabilidad horizontal: añadir más servidores balanceados.

### Relación con pruebas

- Un sistema escalable necesita pruebas automatizadas para validar que los cambios funcionan igual en entornos con más carga.
- Las pruebas de rendimiento y estrés (Locust, JMeter) complementan a las unitarias.
