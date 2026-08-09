---
Fecha de creación: 2026-02-02T18:44:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-02-02
---
[[Clase 02 - Introducción a API REST|← Clase anterior]] | [[Clase 04 - Bases de Datos y Code First|Clase siguiente →]]

# Formatos de Datos: ``JSON`` vs ``XML``
(ver [[JSON and YAML]])

## Historia y Evolución de ``XML``

- Publicado en 1998 por el W3C.
- Fue diseñado como un lenguaje de marcado extensible capaz de estructurar datos de forma jerárquica.
- Adoptado masivamente en sectores donde la interoperabilidad era critica, como la banca y los sistemas empresariales.
- ``XML`` permitió validar estructuras mediante **DTD (_Document Type Definition_)** y posteriormente con **XSD (_XML Schema Definition_)**.
- En EI Salvador, ``XML`` aún se utiliza en facturación electrónica, reportes para Hacienda y sistemas contables de bancos.

### Ventajas

- Alto nivel de estructuración.
- Capacidad de validar datos contra esquemas complejos.
- Soporte extendido en sistemas empresariales.

### Desventajas

- Verbosidad excesiva.
- Tamaño grande de los documentos, lo cual aumenta el consumo de ancho de banda.
- Procesamiento más lento en comparación con ``JSON``.

**Ejemplo:**

```xml
<usuario>
	<id>1</id>
	<nombre>Juan</nombre>
	<edad>25</edad>
</usuario>
```

## Historia y Evolución de ``JSON``

- Introducido por **Douglas Crockford** a principios de los 2000.
- Su sintaxis deriva directamente de **objetos en JavaScript**, lo que lo hace intuitivo para desarrolladores web.
- Fue clave en el auge de **AJAX**, que permitía actualizar secciones de una página sin recargarla completamente.
- Hoy en día, ``JSON`` es el estándar de facto en APIs RESTfuI.

### Ventajas

- Sintaxis clara y concisa.
- Ligero en peso de red.
- Integración nativa con JavaScript.
- Amplio soporte en múltiples lenguajes modernos.

### Limitaciones

- No tiene esquemas robustos nativos como ``XML`` (aunque existen alternativas como **JSON Schema**).
- Menos adecuado para datos extremadamente complejos o con múltiples metadatos.

**Ejemplo:**

```json
{
	"id": 1,
	"nombre": "Juan",
	"edad": 25
}
```

## Comparación Detallada ``JSON`` vs ``XML``

| Criterio           | JSON (JavaScript Object Notation)                              | XML (Extensible Markup Language)         |
| ------------------ | -------------------------------------------------------------- | ---------------------------------------- |
| Legibilidad        | Clara y consisa                                                | Verboso y redundante                     |
| Peso en red        | Ligero                                                         | Pesado                                   |
| Compatibilidad     | Nativo en JS, soporte en todos los lenguajes modernos          | Amplio pero requiere parsers específicos |
| Validación         | Limitada, se apoya en JSON Schema                              | Robusta con DTD y XSD                    |
| Popularidad actual | APIs REST, microservicios, apps móviles                        | Banca, SOAP, sistemas heredados          |
| Uso en El Salvador | Apps móviles, sistemas de salud privados, comercio electrónico |                                          |

## Serialización y Deserialización de Datos

1. **Serialización:** es el proceso de convertir un objeto en memoria en un formato que pueda transmitirse o almacenarse, como ``JSON`` o ``XML``.
	- **Ejemplo:** un objeto ``Usuario`` en memoria se transforma en una cadena ``JSON`` para enviarlo a un cliente web o guardarlo en un archivo.
2. **Deserialización:** es el proceso inverso: convertir un ``JSON`` o ``XML`` recibido en un objeto utilizable por el programa, manteniendo la estructura y los tipos de datos. Este proceso es esencial en aplicaciones distribuidas, microservicios y APIS, donde los objetos en memoria no pueden viajar directamente por la red y deben representarse en un formato estándar.

**Ejemplo C#:**

```csharp
using System. Text.Json;

var usuario = new { Id = 1, Nombre = "Ana", Edad = 22 };

// Serialización
string jsonString = JsonSeria1izer.Seria1ize(usuario);
Console.WriteLine("Serializado: " + jsonString);

// Deserialización
var usuari02 = JsonSerializer.Deserialize<Dictionary<string, object>>(jsonString);
Console.WriteLine("Deserializado:" + usuario2["Nombre"]);
```

- ``JsonSerializer.Serialize`` convierte objetos .NET en ``JSON``.
- ``JsonSerializer.Deserialize`` convierte ``JSON`` en un objeto manipulable en C#.
- Se puede deserializar directamente a clases fuertemente tipadas para mayor seguridad.

### Riesgos de Deserialización Insegura

Si los datos entrantes no se validan, pueden producirse problemas graves:

1. **Datos corruptos:** pueden provocar errores de ejecución o caídas de la aplicación.
2. **Inyección de código:** un atacante puede enviar payloads maliciosos, explotando vulnerabilidades de deserialización.
3. **Exposición de información sensible:** al deserializar datos que incluyen campos que no deberían procesarse.

### Buenas Prácticas

- Validar la estructura y los tipos de datos antes de deserializar.
- Evitar deserializar directamente objetos sin control.
- Utilizar bibliotecas confiables que implementen protecciones contra ataques de deserialización.

## Validación de Entradas en Backend

La validación asegura que los datos recibidos cumplen con reglas mínimas antes de ser procesados o almacenados. Sus objetivos principales:

- Evitar errores de ejecución.
- Prevenir ataques de seguridad (SQL Injection, XSS, CSRF).
- Garantizar consistencia y calidad de la información.
- Cumplir con normas legales y regulatorias (ejemplo: protección de datos en hospitales en EI Salvador).

### Tipos de Validación

1. **Sintáctica:** verifica que los datos cumplen con la estructura esperada.
	- Ejemplo: JSON bien formado, fecha con formato yyyy-mm-dd.
2. **Semántica:** verifica que los datos tengan sentido.
	- Ejemplo: la edad de un paciente debe ser mayor a 0 y menor a 120.
3. **De negocio:** reglas específicas del dominio de la aplicación.
	- Ejemplo: un paciente no puede registrarse dos veces con el mismo DUI.
	- En inventarios, el stock no puede ser negativo.

**Ejemplo en .NET con Data Annotations**

```csharp
public class Usuario
{
	[Required]
	[StringLength(50)]
	public string Nombre { get; set; }
	
	[Range(1, 120)]
	public int Edad { get; set; }
}
```

- ``[Required]`` obliga a que la propiedad tenga un valor.
- ``[StringLength]`` limita la longitud máxima de cadenas.
- ``[Range]`` asegura que los valores numéricos estén dentro de un rango válido.
- .NET valida automáticamente la entrada al recibir datos en una API, reduciendo errores y aumentando seguridad.

**Ejemplo Práctico en .NET 8:**

Definir el modelo ``Usuario``:

```csharp
using System.ComponentModel.DataAnnotations;
namespace ApiUsuarios.Models
{
	public class Usuario
	{
		public int Id { get; set: }
		
		[Required(ErrorMessage = "El nombre es obligatorio")]
		[StringLength(50, ErrorMessage = "El nombre no puede superar 50 caracteres")]
		public string Nombre { get: set; }
		
		[Range(1, 120, ErrorMessage = "Edad inválida")]
		public int Edad { get; set; }
	}
}
```

Crear el controlador ``UsuariosController``:

```csharp
using Microsoft.AspNetCore.Mvc;
using ApiUsuarios.Models;
using System.DataAnnotations;

namespace ApiUsuarios.Controllers
{
	[ApiController]
	[Route("api/[controller]")]
	public class UsuariosController : ControllerBase
	{
		// Lista estática en memoria para simular base de datos
		private static readonly List<Usuario> Usuarios = new List<Usuario>();
		
		// GET: api/usuarios
		[HttpGet]
		public ActionResult<IEnumerable<Usuario>> GetUsuarios()
		{
			return Ok(Usuario);
		}
		
		// GET: api/usuarios/{id}
		[HttpGet("{id}")]
		public ActionResult<Usuario> GetUsuarioPorId(int id)
		{
			var usuario = Usuarios.FirstOrDefault(u => u.Id == id);
			if (usuario == null)
			{
				return NotFound("Usuario no encontrado");
			}
			return Ok(usuario);
		}
		
		// POST: api/usuarios
		[HttpPost]
		public ActionResult<Usuario> CrearUsuario([FromBody] Usuario usuario)
		{
			// Validación usuando DataAnnotations
			var context = new ValidationContext(usuario);
			var results = new List<ValidationResult>();
			if (!Validator.TryValidateObject(usuario, context, results, true))
			{
				return BadRequest(results);
			}
			
			// Asignar Id incremental y agregar a la lista
			usuario.Id = Usuarios.Count + 1;
			Usuarios.Add(usuario);
			
			return CreatedAtAction(nameof(GetUsuarioPorId), new { id = usuario.Id }, usuario);
		}
		
		// PUT: api/usuarios/{id} - Actualizar usuario
		[HttpPut("{id}")]
		public ActionResult<Usuario> ActualizarUsuario(int id, [FromBody] Usuario usuarioActualizado)
		{
			var usuario = Usuarios.FirstOrDefault(u => u.Id == id);
			if (usuario == null)
			{
				return NotFound("Usuario no encontrado");
			}
			
			// Validación
			var context = new ValidationContext(usuarioActualizado);
			var results = new List<ValidationResult>();
			if (!Validator.TryValidateObject(usuarioActualizado, context, results, true))
			{
				return BadRequest(results);
			}
			usuario.Nombre = usuarioActualizado.Nombre;
			usuario.Edad = usuarioActualizado.Edad;
			
			return Ok(Usuario);
		}
		
		// DELETE: api/usuarios/{id} - Eliminar usuario
		[HttpDelete("{id}")]
		public ActionResult EliminarUsuario(int id)
		{
			var usuario = Usuarios.FirstOrDefault(u +> u.Id == id);
			if(usuario == null)
				return NotFound("Usuario no encontrado");
				
			Usuarios.Remove(usuario);
			return NoContent();
		}
	}
}
```
