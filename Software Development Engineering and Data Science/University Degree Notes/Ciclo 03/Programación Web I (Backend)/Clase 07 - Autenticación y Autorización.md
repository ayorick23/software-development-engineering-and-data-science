---
Fecha de creación: 2026-03-02T18:17:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-03-02
---
# Autenticación y Autorización

Estos dos conceptos suelen confundirse, pero son procesos distintos y complementarios dentro de la seguridad backend.

## Autenticación

La **autenticación** es el proceso de verificar la identidad de un usuario o servicio. En otras palabras, responde a la pregunta: "¿Quién eres?".

En .NET, existen múltiples mecanismos de autenticación:

- **Autenticación por credenciales:** usuario y contraseña verificadas contra una base de datos.
- **Autenticación por token:** el servidor emite un token (por ejemplo, JWT) que el cliente usa peticiones posteriores.
- **Autenticación externa (OAuth2/OpenID Connect):** integración con proveedores externos como Google, Microsoft o Facebook.

**Ejemplo en .NET 8:**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Identity;
using System.Threading.Tasks;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
	private readonly UserManager<IdentityUser> _userManager;
	private readonly SignInManager<IdentityUser> _signInManager;
	
	public AuthController(UserManager<IdentityUser> userManager, SignInManager<IdentityUser> signInManager)
	{
		_userManager = userManager;
		_signInManager = signInManager;
	}
	
	[HttpPost("login")]
	public async Task<IActionResult> Login([FromBody] LoginRequest request)
	{
		// Busca el usuario por nombre
		var user = await _userManager.FindByNameAsync(request.Username);
		if (user == null)
		{
			return Unauthorized(new { message = "Credenciales inválidas" });
		}
		
		// Intenta iniciar sesión
		var result = await _signInManager.CheckPasswordSignInAsync(user, request.Password, lockoutOnFailure: false);
		
		if (!result.Succeeded)
		{
			return Unauthorized(new { message = "Credenciales inválidas" });
		}
		
		// Aquí normalmente generarías un JWT real
		var token = Guid.NewGuid().ToString(); // Solo simulación
		
		return Ok(new
		{
			message = "Login correcto",
			token,
			user = new
			{
				user.Id,
				user.UserName,
				user.Email
			}
		});
	}
}

public class LoginRequest
{
	public string Username { get; set; } = string.Empty;
	public string Password { get; set; } = string.Empty;
}
```

Este ejemplo es básico, pero ilustra la autenticación: si las credenciales son válidas, se devuelve un token (en una implementación real, sería un **JWT firmado**).

## Autorización

La **autorización** es el proceso de determinar qué puede hacer un usuario autenticado. Responde a la pregunta: "¿Qué estás autorizado a hacer?".

Por ejemplo:

- Un **usuario administrador** puede eliminar registros.
- Un **usuario estándar** solo puede leer y modificar su propia información.

**Ejemplo en .NET 8:**

```csharp
[Authorize(Roles = "Admin")]
[HttpDelete("usuarios/{id}")]
public IActionResult EliminarUsuario(int id)
{
	// Solo los usuarios con rol "Admin" pueden ejecutar esto
	return Ok($"Usuario {id} eliminado correctamente");
}
```

ASP.NET Core integra mecanismos de autorización basados en **roles, políticas y calims**, permitiendo un control granular y declarativo sobre el acceso a recursos.

## Encriptación y Manejo Seguro de Datos

### ¿Qué es la encriptación?

La encriptación es el proceso de convertir datos legibles en una forma ilegible mediante un algoritmo y una clave. Solo quien tenga la clave puede **desencriptar** y acceder a los datos originales.

En un sistema backend, la encriptación se aplica en:

- **Datos en tránsito:** información que se mueve entre cliente y servidor.
- **Datos en reposo:** información almacenada en bases de datos o archivos.

### Tipos de Encriptación

1. **Simétrica:** usa la misma clave para cifrar y descifrar (ej: AES).
2. **Asimétrica:** usar un par de claves (pública y privada), como en RSA.
3. **Hashing:** no se puede revertir; usado para contraseñas (ej: SHA-256 o bcrypt).

**Ejemplo en .NET - Cifrado simétrico con AES:**

```csharp
using System.Security.Cryptography;
using System.Text;

public static class Encriptador
{
	public static string Encriptar(string texto, string clave)
	{
		using var aes = Aes.Create();
		var key = Encoding.UTF8.GetBytes(clave.PadRight(32));
		aes.Key = key;
		aes.GenerateIV();
		using var encryptor = aes.CreateEncryptor();
		var bytes = Encoding.UTF8.GetBytes(texto);
		var cifrado = encryptor.TransformFinalBlock(bytes, 0, bytes,Length);
		return Convert.ToBase64String(aes.IV.Concat(cifrado).ToArray());
	}
}
```

Este ejemplo demuestra cómo en .NET puede implementarse una encriptación básica de datos antes de almacenarlos.

### Buenas prácticas de manejo seguro

- Nunca guardar contraseñas en texto plano.
- Usar hash + satl (bcrypt, PBKDF2).
- No exponer claves o secretos en el código fuente (usar Secret Manager o variables de entorno).
- Proteger los tokens de sesión y renovarlos periódicamente.

## HTTPS y Certificados SSL/TLS

### Importancia de HTTPS

El protocolo **HTTPS (Hypertext Transfer Protocol Secure)** añade una capa de seguridad al HTTP tradicional mediante **SSL/TLS**. Esta capa garantiza:

- **Confidencialidad:** los datos viajan cifrados.
- **Integridad:** los datos no pueden ser modificados en tránsito.
- **Autenticidad:** el servido es realmente quien dice ser.

Antes de HTTPS, el tráfico HTTP podía ser interceptado fácilmente mediante ataques tipo **Man-in-the-Middle (MITM)**.

Con TLS (Transport Layer Security), los navegadores y servidores negocian claves de sesión únicas que protegen toda la comunicación.

### Cómo funciona el proceso TLS

1. El cliente envía una solicitud de conexión al servidor.
2. El servidor envía su certificado digital.
3. El cliente verifica la validez del certificado.
4. Si es válido, se establece una clave de sesión compartida.
5. A partir de ese momento, toda la comunicación se cifra.

### Configuración de HTTPS en .NET 8

ASP.NET CORE activa HTTPS por defecto durante el desarrollo. El archivo launchSettings.json contiene algo así:

```json
"applicationUrl": "https://localhost:7071;http://localhost:5071"
```

Para entornos de producción, se usan certificados emitidos por una Autoridad de Certificación (CA) confiable como Let's Encrypt o DigiCert.

Se configuran en Program.cs o a nivel de servidor web (IIS, Kestrel o Nginx).

**Ejemplo: Forzar HTTPS**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseHttpsRedirection(); // Redirige HTTP a HTTPS
app.MapControllers();
app.Run();
```

Esto garantiza que todas las solicitudes sean seguras y se redirijan automáticamente a HTTPS.

## Buenas Prácticas Generales de Seguridad en Backend

1. **Principio de menor privilegio:** cada componente debe tener solo los permisos necesarios.
2. **Validación en el servidor:** nunca confiar solo en la validación del frontend.
3. **Uso de tokens cortos:** los tokens de autenticación deben tener expiración corta.
4. **Regeneración de sesiones:** después del login exitoso.
5. **Auditoría y logs de seguridad:** registrar intentos de acceso y acciones críticas.
6. **Uso de middlewares de seguridad** como UseHsts, UseAuthorization y UseHttpsRedirection.
