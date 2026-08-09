---
Fecha de creación: 2026-02-02T18:10:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-02-02
---
[[Clase 01 - Introducción a la Programación Web|← Clase anterior]] | [[Clase 03 - Formatos de Datos - JSON vs XML|Clase siguiente →]]

# Introducción a API REST

## ¿Qué es una API y por qué se usa?
(ver [[APIs|APIs en Python]])

Las **APIs (_Application Programming Interfaces_)** son esenciales en el desarrollo de software actual. Permiten que distintos sistemas puedan comunicarse, intercambiar información y trabajar de forma integrada. Por ejemplo, cuando usamos una app de transporte, esta consulta información de ubicación mediante la API de Google Maps; o cuando compramos en línea, el sistema se conecta con APIS de pago como PayPal o Stripe.

En este contexto surge **REST (_Representational State Transfer_)**, un estilo arquitectónico que define cómo deben diseñarse las APIS para aprovechar al máximo el protocolo HTTP. REST se ha convertido en el estándar por su simpleza, escalabilidad y universalidad.

En el desarrollo backend, las APIs son fundamentales porque:

- Permiten que **clientes** (web, móvil, escritorio o sistemas externos) hagan solicitudes de datos o acciones.
- El servidor recibe la solicitud, la procesa y devuelve una **respuesta estructurada** (generalmente ``JSON``).

**Ejemplos reales:**

- **APIs meteorológicas:** Una aplicación del clima muestra la temperatura de tu ciudad.
- **APIs de autenticación:** Inicio de sesión con Google o Facebook.
- **APIs de pago:** PayPal, Stripe, Visa Checkout.

**Beneficio clave:** Estandarizan la comunicación entre sistemas heterogéneos, evitando dependencias rígidas y permitiendo escalabilidad.

## Introducción a REST

A diferencia de un protocolo rígido corno HTTP, REST no define comandos específicos, sino un conjunto de principios y restricciones que guían el diseño de aplicaciones que aprovechan las capacidades de la web.

Su gran aporte fue establecer cómo los recursos pueden representarse y manipularse a través de un conjunto de operaciones estándar (los métodos HTTP), logrando así que los sistemas sean simples, escalables, modulares y fáciles de integrar.

>**REST no es un protocolo**

Una confusión común es pensar que REST es un **protocolo** (como HTTP, FTP o SMTP). En realidad, REST es un **estilo arquitectónico** que se implementa sobre protocolos existentes, siendo HTTP el más utilizado.

### Principios Fundamentales de REST

REST se basa en una serie de principios o restricciones que definen cómo deben diseñarse los servicios:

1. **Cliente-Servidor**
	- La arquitectura debe separar la interfaz de usuario (cliente) de la gestión de datos (servidor).
	- Esta separación mejora la portabilidad del código en el cliente y la escalabilidad del servidor.
	- **Ejemplo:** Una aplicación móvil consume la API de un servidor en .NET, pero el servidor no depende de cómo se ve la interfaz.
2. **Sin estado (Stateless)**
	- Cada petición del cliente al servidor debe contener toda la información necesaria para procesarla.
	- EI servidor no debe almacenar el estado de la sesión entre peticiones.
	- **Ejemplo:** En lugar de que el servidor recuerde al usuario autenticado, el cliente envía en cada solicitud un token JWT en la cabecera Authorization.
3. **Cacheable**
	- Las respuestas deben indicar si son almacenables en caché o no, para mejorar la eficiencia y escalabilidad.
	- **Ejemplo:** Al consultar un producto en una tienda online, la API puede indicar que los datos son válidos por 5 minutos usando encabezados HTTP (Cache-Control).

### Interfaz Uniforme

La simplicidad de REST proviene de una interfaz uniforme para interactuar con los recursos:

- **Identificación de recursos mediante URLs**
	- **Ejemplo:** https://api.mitienda.com/productos/45 identifica eL producto con ID 45.
- **Representaciones de recursos**
	- Un recurso puede representarse en ``JSON``, ``XML`` o incluso ``HTML``.
	- **Ejemplo:** ``{ "id"' 45, "nombre": "Laptop Acer", "precio": 750``.
- **Operaciones estándar (HTTP methods)**
	- GET/productos: obtener productos.
	- POST/productos: crear un nuevo producto.
	- PUT/productos/45: actualizar eL producto con ID 45.
	- DELETE/productos/45: eliminar el producto.
- **HATEOAS (Hypermedia As The Engine Of Application State)**
	- La respuesta puede incluir enlaces para navegar en la API.
	- **Ejemplo:**
		```json
		{
			"id": 45,
			"nombre": "Laptop Acer",
			"precio": 750,
			"links": {
				"self": "/productos/45",
				"reviews": "/productos/45/reviews"
			}
		}
		```

### Sistema en Capas

- La arquitectura puede organizarse en capas (por ejemplo, seguridad, balanceadores, cache intermedio) sin que el cliente necesite saberlo.
- **Ejemplo:** Un cliente puede consumir una API, pero la petición puede pasar antes por un **API Gateway** o un **proxy caché**.

### Código Bajo Demanda (Opcional)

- EI servidor puede enviar código ejecutable al cliente (como scripts en JavaScript) para extender su funcionalidad.
- Este principio es opcional y poco usado en APIs REST modernas, pero sí en casos como APIs que devuelven scripts para clientes web.

### Ventajas de REST

- **Simplicidad:** Se basa en HTTP, que ya es ampliamente conocido.
- **Escalabilidad:** Gracias a su naturaleza sin estado.
- **Interoperabilidad:** Se puede usar con múltiples lenguajes y plataformas (.NET, Java, Python, etc.).
- **Flexibilidad:** Los recursos pueden representarse en distintos formatos (``JSON``, ``XML``, ``YAML``, etc.).
- **Eficiencia:** Uso de caché y separación cliente-servidor.

### Ejemplo Práctico en .NET (Web API)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductosController : ControllerBase
{
	// Lista estática que simula una base de datos en memoria
	private static List<Producto> productos = new List<Producto>
	{
		new Producto { Id = 1, Nombre = "Lapto Acer", Precio = 750 },
		new Producto { Id = 2, Nombre = "Mouse Logitech", Precio = 25}
	};
	
	// Método GET que devuelve todos los productos
	[HttpGet]
	public ActionResult<IEnumerable<Producto>> Get() => productos;
	
	// Método GET que devuelve un producto específico por su Id
	[HttpGet("{id}")]
	public ActionResult<Producto> Get(int id)
	{
		var producto = productos.FirstOrDefault(p => p.Id == id);
		return producto == null ? NotFound() : Ok(producto);
	}
	
	// Método POST para agregar un nuevo producto
	[HttpPost]
	public ActionResult<Producto> Post(Producto nuevoProducto)
	{
		// Se asigna un Id nuevo incrementando el mayor existente
		nuevoProducto.Id = productos.Max(p => p.Id) + 1;
		productos.Add(nuevoProducto);
		
		// Devuelve 201 Created con la ruta del nuevo recurso
		return CreatedAtAction(nameof(Get), new { id = nuevoProducto.Id }, nuevoProducto);
	}
	
	// Método PUT para actualizar un producto existente
	[HttpPut("{id}")]
	public IActionResult Put(int id, Producto productoActualizado)
	{
		var producto = productos.FirstOrDefault(p => p.Id == id);
		if (producto == null) return NotFound();
		
		// Se actualizan los valores del producto
		producto.Nombre = productoActualizado.Nombre;
		producto.Precio = productoActualizado.Precio;
		
		// No devuelve contenido, solo confirma la operación
		return NoContent();
	}
	
	// Método DELETE para eliminar un producto
	[HttpDelete("{id}")]
	public IActionResult Delete(int id)
	{
		var producto = productos.FirstOrDefault(p => p.Id == id);
		if (producto == null) return NotFound();
		
		productos.Remove(producto);
		return NoContent();
	}
}
```

Con esto:

- GET /api/productos devuelve todos los productos.
- GET /api/productos/1 devuelve el producto con ID 1.
- POST /api/productos crea un nuevo producto.
- PUT /api/productos/1 actualiza el producto con ID 1.
- DELETE /api/productos/1 elimina el producto con ID 1.
