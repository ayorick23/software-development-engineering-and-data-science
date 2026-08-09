---
Fecha de creación: 2026-03-14T14:12:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-03-14
---
[[Clase 07 - Autenticación y Autorización|← Clase anterior]] | [[Clase 09 - SQL Injection y Patrones de Diseño|Clase siguiente →]]

# Cifrado, Hashing, Salting y Arquitectura Limpia

EI manejo de contraseñas es uno de los aspectos más críticos en el desarrollo de aplicaciones seguras.

A lo largo de la historia, las brechas de seguridad más devastadoras han tenido un denominador común: **malas prácticas en el almacenamiento y manejo de contraseñas**.

Una contraseña mal protegida compromete no solo a un usuario, sino potencialmente a miles, ya que muchas personas reutilizan sus credenciales en diferentes servicios.

EI objetivo principal del manejo seguro de contraseñas es que, aunque un atacante obtenga acceso a la base de datos, no pueda deducir las contraseñas reales debido al uso de algoritmos no reversibles y costosos computacionalmente.

En el ecosistema .NET moderno (.NET 8 y ASP.NET Core), la seguridad se fundamenta en el uso de:

- Hashing criptográfico con salt aleatorio.
- Algoritmos adaptativos (bcrypt, PBKDF2, Argon2).
- Tokens de autenticacion JWT firmados con claves seguras.
- Conexiones HTTPS/TLS para proteger los datos en tránsito.
- Mecanismos antifuerza bruta y MFA.

## Cifrado

EI **cifrado** transforma un texto legible en uno ilegible mediante una **clave secreta**, pero el proceso es **reversible**: quien posea la clave puede descifrar el texto.

Se utiliza para proteger información en tránsito (como datos bancarios o mensajes), **pero no debe usarse para contraseñas** ya que implica la posibilidad de revertir la información.

**Ejemplo de cifrado con AES en .NET:**

```csharp
using System.Security.Cryptography;
using System.Text;

string texto = "Hola1234";
using Aes aes = Aes.Create();
aes.Key = Encoding.UTF8.GetBytes("claveDe32BytesDeLongitudExacta!");
aes.IV = new byte[16];

var encriptador = aes.CreateEncryptor(aes.Key, aes.IV);
byte[] datos = encriptador.TransformFinalBlock(Encoding.UTF8.GetBytes(texto), 0, texto.Length);
```

### Desventaja

Si un atacante obtiene la clave de cifrado, puede recuperar todas las contraseñas almacenadas. Por ello, el cifrado no se usa para contraseñas, solo para datos que deban ser recuperables.

## Hashing

EI **hashing** convierte un texto (por ejemplo, una contraseña) en una **cadena única e irreversible** mediante una función criptográfica.

A diferencia del cifrado, **no se puede recuperar el texto original** desde el hash.

Por ejemplo, la función SHA256 produce un hash de longitud fija de 256 bits, sin importar la longitud del texto original.

**Ejemplo:**

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

class Program
{
	static void Main()
	{
		string password = "MiContraseñaSegura123";
		
		// Hashear la contraseña
		string hashedPassword = HashPassword(password);
		
		Console.WriteLine("Contraseña original: " + password);
		Console.WriteLine("Hash SHA256: " + hashedPassword);
	}
	
	static string HashPassword(string password)
	{
		using (SHA256 sha256 = SHA256.Create())
		{
			byte[] bytes = Encoding.UTF8.GetBytes(password);
			byte[] hash = sha256.ComputeHash(bytes);
			
			StringBuilder result = new StringBuilder();
			foreach (byte b in hash)
			{
				result.Append(b.ToString("x2"));
			}
			
			return result.ToString();
		}
	}
}
```

El hashing asegura que:

- No se puede revertir a la contraseña original.
- Las contraseñas iguales produzcan el mismo hash.

Sin embargo, es vulnerable a ataques de diccionario o tablas rainbow, donde los atacantes comparan hashes conocidos para adivinar contraseñas comunes.

## Salting

EI **salting** consiste en agregar un valor aleatorio (llamado salt) a la contraseña antes de generar el hash.

Esto garantiza que dos usuarios con la misma contraseña obtengan hashes completamente
diferentes, haciendo ineficaces los ataques basados en tablas precomputadas.

**Ejemplo en C# con PBKDF2:**

```csharp
using System.Security.Cryptography;
using System.Text;

string password = "Hola1234";
byte[] salt = RandomNumberGenerator.GetBytes(16);
byte[] hash = Rfc2898DeriveBytes.Pbkdf2(
	password,
	salt,
	100000,
	HashAlgorithmName.SHA512,
	64
);

Console.WriteLine(Convert.ToBase64String(hash));
```

Cada usuario debe tener su **propio salt único**, que se almacena junto al hash.

## Hashing Seguro en .NET con Algoritmos Modernos

En .NET 8, la seguridad de contraseñas se implementa mediante algoritmos **adaptativos**: su complejidad se puede aumentar para contrarrestar el incremento del poder de cómputo de los atacantes.

Los algoritmos más usados son:

| Algoritmo  | Descripción                                       | Nivel de Seguridad |
| ---------- | ------------------------------------------------- | ------------------ |
| ``PBKDF2`` | Algoritmo estándar con iteraciones configurables. | Alto               |
| ``bcrypt`` | Algoritmo lento y resistente a ataques de GPU.    | Muy alto           |
| ``Argon2`` | Ganador del Password Hashing Competition (PHC)    | Excelente          |

**Ejemplo usando ``PasswordHasher`` en ASP.NET Core Identity:**

```csharp
using Microsoft.AspNetCore.Identity;

var hasher = new PasswordHasher<string>();
string password = "Admin123!";
string hash = hasher.HashPassord(null, password);
Console.WriteLine(hash);

// Verificación
var resultado = hasher.VerifyHashedPassword(null, hash, "Admin123!");
Console.WriteLine(resutlado); // Succes
```

El ``PasswordHasher`` usa ``PBDKDF2`` por defecto, generando un _salt_ aleatorio interno y aplicando iteraciones configuradas para dificultar los ataques.

## Flujo Seguro de Registro y Login

### Registro

El flujo seguro de registro implica que:

1. El usuario envía correo y contraseña.
2. El backend valida los datos.
3. Se aplica hashing y salting a la contraseña.
4. Se guarda elhash, el salt y los metadatos, **nunca la contraseña original**.

```csharp
[HttpPost("registro")]
public IActionResult Registrar(UsuarioDto dto)
{
	var hasher = new PasswordHasher<Usuario>();
	var usuario = new Usuario
	{
		Email = dto.Email;
		PasswordHash = hasher.HashPassword(null, dto.Password)
	};
	
	_dbContext.Usuarios.Add(usuario);
	_dbContext.SaveChanges();
	return Ok("Usuario registrado exitosamente");
}
```

### Login

En el proceso de login:

1. El usuario envía su correo y contraseña.
2. El backen obtiene el hash almacenado.
3. Aplica el mismo algoritmo de hash a la contraseña enviada.
4. Compara resultados de forma segura.
5. Si coinciden, genera un **JWT (JSON Web Token)**.

```csharp
[HttpPost("login")]
public IActionResult Login(LoginDto dto)
{
	var user = _dbContext.Usuarios.SingleOrDefault(u => u.Email == dto.Email);
	if (user == null) return Unauthorized();
	
	var hasher == new PasswordHasher<Usuario>();
	var result == hasher.VerifyHashedPassword(user, user.PasswordHash, dto.Password);
	if (result == PasswordVerificationResult.Failed)
	{
		return Unauthorized();
	}
	
	var token = _jwtService.GenerateToken(user);
	return Ok(new { Token = token });
}
```

## Autenticación Segura con JWT en .NET 8

### ¿Qué es JWT?


**JWT (JSON Web Token)** es un estándar (RFC 7519) que permite autenticar usuarios mediante **tokens firmados digitalmente**, en lugar de mantener sesiones en el servidor.

Cada token contiene información codificada (claims) que identifican al usuario, su rol y tiempo de expiración.

Estructura del JVW:

```
xxxxx.yyyyy.77777
```

Donde:

- **Header (``xxxxx``):** contiene el tipo de token y algoritmo de firma (ej. HS256).
- **Payload (``yyyyy``):** contiene los claims (ej. usuariold, rol, expiración).
- **Signature (``zzzzz``):** firma criptográfica que garantiza que el token no ha sido alterado.

**Ejemplo de generación de JWT en .NET 8:**

```csharp
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

public class JwtService
{
	private readonly string _secretKey;

	public JwtService(IConfiguration config)
	{
		_secretKey = config["Jwt:SecretKey"]; // clave larga y secreta, nunca en el código fuente
	}

	public string GenerateToken(Usuario user)
	{
		var claims = new[]
		{
			new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
			new Claim(ClaimTypes.Email, user.Email),
			new Claim(ClaimTypes.Role, user.Rol)
		};

		var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_secretKey));
		var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

		var token = new JwtSecurityToken(
			claims: claims,
			expires: DateTime.UtcNow.AddHours(1), // token de corta duración
			signingCredentials: credentials
		);

		return new JwtSecurityTokenHandler().WriteToken(token);
	}
}
```

### Validación del JWT

El backend debe validar cada petición autenticada verificando la firma y la expiración del token. En ASP.NET Core esto se configura una sola vez, como middleware de autenticación, en lugar de validarlo manualmente en cada endpoint:

```csharp
// En Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
	.AddJwtBearer(options =>
	{
		options.TokenValidationParameters = new TokenValidationParameters
		{
			ValidateIssuerSigningKey = true,
			IssuerSigningKey = new SymmetricSecurityKey(
				Encoding.UTF8.GetBytes(builder.Configuration["Jwt:SecretKey"])),
			ValidateIssuer = false,
			ValidateAudience = false,
			ValidateLifetime = true, // rechaza tokens expirados automáticamente
			ClockSkew = TimeSpan.Zero
		};
	});

// ...
app.UseAuthentication(); // debe ir antes de UseAuthorization
app.UseAuthorization();
```

Con esto configurado, basta con decorar un endpoint con `[Authorize]` (o `[Authorize(Roles = "Admin")]`, como en la Clase 07) para que ASP.NET Core valide automáticamente la firma y la expiración del token antes de ejecutar el método.

## Protección contra Ataques de Fuerza Bruta

Además del hashing, es importante prevenir ataques por intentos repetidos.

Medidas de mitigación:

1. **Bloqueo temporal** tras varios intentos fallidos.
2. **Registro de intentos en logs**.
3. **CAPTCHA o MFA** para verificar al usuario.
4. **Rate limiting** en endpoints sensibles.

**Ejemplo de configuración de bloqueo en ASP.NET Identity:**

```csharp
options.Lockout.MaxFailedAccessAttempts = 5;
options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
```

## Buenas Prácticas de Seguridad

1. Nunca almacenar contraseñas en texto plano.
2. Usar algoritmos adaptativos con salt.
3. Exigir contraseñas robustas (mayúsculas, minúsculas, números, símbolos).
4. No enviar contraseñas por correo.
5. Forzar el cambio periódico de contraseña.
6. Implementar 2FA (autenticación multifactor).
7. Usar HTTPS en todas las comunicaciones.
8. Revocar tokens antiguos al cambiar contraseñas.
9. Aplicar rate limiting y auditorías de seguridad.
10. Monitorear intentos de acceso sospechosos.

## Arquitectura Limpia
(evoluciona la [[Clase 04 - Bases de Datos y Code First#Estructura Profesional de un Proyecto ASP.NET|estructura de carpetas]] vista en la Clase 04 hacia una separación explícita por capas)

La **Arquitectura Limpia (_Clean Architecture_)**, propuesta por Robert C. Martin, organiza el código en capas concéntricas donde las capas externas dependen de las internas, pero nunca al revés. El objetivo es que la lógica de negocio (el "dominio") no dependa de detalles técnicos como el framework web o el motor de base de datos — esos detalles deben poder cambiarse sin tocar las reglas de negocio.

### Las Capas

1. **Domain (Dominio):** el núcleo. Contiene las entidades (`Usuario`, `Producto`) y las reglas de negocio puras, sin ninguna dependencia externa — ni de EF Core, ni de ASP.NET, ni de ninguna librería.
2. **Application (Aplicación):** casos de uso del sistema (ej. "registrar usuario", "crear producto"). Define *qué* hace el sistema, orquestando las entidades del dominio, pero sin saber *cómo* se persisten los datos o *cómo* llegan las peticiones HTTP.
3. **Infrastructure (Infraestructura):** implementaciones concretas — el `DbContext` de EF Core, el `JwtService`, el envío de correos, llamadas a APIs externas. Depende de Application y Domain, nunca al revés.
4. **Presentation (Presentación):** los `Controllers` de la API, responsables solo de recibir peticiones HTTP, invocar los casos de uso de Application, y devolver la respuesta.

### ¿Por Qué Importa Esta Separación?

- **Independencia de frameworks:** el dominio no sabe que existe ASP.NET Core; se podría exponer por una API REST, una CLI o un worker sin cambiar la lógica de negocio.
- **Independencia de la base de datos:** cambiar de SQL Server a PostgreSQL, o de relacional a MongoDB, solo afecta la capa de Infrastructure.
- **Testabilidad:** los casos de uso en Application se pueden probar unitariamente sin levantar una base de datos real, ya que dependen de abstracciones (interfaces), no de implementaciones concretas — el mismo principio de inyección de dependencias usado para inyectar el `AppDbContext` en los controladores.

### Ejemplo de Organización de Carpetas

```
MiApi
│
├── MiApi.Domain
│     Entities/Usuario.cs
│     Entities/Producto.cs
│
├── MiApi.Application
│     Interfaces/IUsuarioRepository.cs
│     UseCases/RegistrarUsuario.cs
│
├── MiApi.Infrastructure
│     Data/AppDbContext.cs
│     Repositories/UsuarioRepository.cs
│     Services/JwtService.cs
│
└── MiApi.Presentation (o MiApi.Api)
      Controllers/UsuariosController.cs
      Program.cs
```

A diferencia de la estructura plana de la Clase 04 (una sola carpeta `Controllers`, `Models`, `Data`), aquí cada capa es su propio proyecto dentro de la solución, lo que obliga —a nivel de compilador— a respetar la dirección de las dependencias.
