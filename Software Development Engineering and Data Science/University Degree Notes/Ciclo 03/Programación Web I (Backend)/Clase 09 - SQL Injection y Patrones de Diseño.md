---
Fecha de creación: 2026-03-16T18:08:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-03-16
---
[[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia|← Clase anterior]] | [[Clase 10 - Autenticación en .NET 8|Clase siguiente →]]

# SQL Injection y Patrones de Diseño
(ver [[30-Principios-SOLID-y-Clean-Code-para-ML|Principios SOLID y Clean Code]] para la base de los patrones de diseño)

## SQL Injection

La **inyección SQL (_SQL Injection_)** es una vulnerabilidad que ocurre cuando la entrada de un usuario se concatena directamente dentro de una consulta SQL, permitiendo que un atacante altere la consulta que realmente se ejecuta en la base de datos.

**Ejemplo vulnerable (nunca hacer esto):**

```csharp
// El usuario controla "usuarioInput" a través de un formulario de login
string query = $"SELECT * FROM Usuarios WHERE Nombre = '{usuarioInput}' AND Password = '{passwordInput}'";
```

Si un atacante escribe en el campo de nombre de usuario:

```
' OR '1'='1
```

la consulta ejecutada se convierte en:

```sql
SELECT * FROM Usuarios WHERE Nombre = '' OR '1'='1' AND Password = '...'
```

Como `'1'='1'` siempre es verdadero, la condición se cumple sin importar el password, permitiendo iniciar sesión sin credenciales válidas. Ataques más avanzados pueden usar esta técnica para leer, modificar o eliminar toda la base de datos.

### Por Qué Entity Framework Core Ya Te Protege

Las consultas construidas con **EF Core** (como `_context.Usuarios.FirstOrDefault(u => u.Nombre == usuarioInput)`, usadas desde la [[Clase 04 - Bases de Datos y Code First|Clase 04]]) son seguras por defecto: EF Core traduce estas expresiones LINQ a **consultas parametrizadas**, donde el valor del usuario se envía por separado del texto SQL, y el motor de base de datos nunca lo interpreta como parte del comando.

El riesgo reaparece cuando se usa SQL crudo concatenando strings manualmente:

```csharp
// VULNERABLE
_context.Database.ExecuteSqlRaw($"SELECT * FROM Usuarios WHERE Nombre = '{usuarioInput}'");

// SEGURO: parámetro real, no concatenación
_context.Database.ExecuteSqlRaw("SELECT * FROM Usuarios WHERE Nombre = {0}", usuarioInput);
```

### Buenas Prácticas contra SQL Injection

- Usar siempre consultas parametrizadas o un ORM (EF Core) — nunca concatenar entrada de usuario en SQL.
- Validar y sanear la entrada del usuario (ver [[Clase 03 - Formatos de Datos - JSON vs XML#Validación de Entradas en Backend|Validación de Entradas]] de la Clase 03).
- Aplicar el **principio de menor privilegio** al usuario de base de datos que usa la aplicación (no debería poder `DROP TABLE` si solo necesita `SELECT`/`INSERT`).
- Nunca mostrar mensajes de error de base de datos crudos al cliente, ya que revelan la estructura de las tablas.

## Patrones de Diseño en el Backend

Los patrones de diseño resuelven problemas recurrentes de organización del código. En una API backend son especialmente comunes tres:

### Repository Pattern

Encapsula el acceso a los datos detrás de una interfaz, para que el resto de la aplicación no dependa directamente de EF Core ni de cómo se consulta la base de datos — la misma idea de desacoplamiento que motiva la [[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia#Arquitectura Limpia|Arquitectura Limpia]] de la Clase 08.

```csharp
public interface IUsuarioRepository
{
	Task<Usuario?> ObtenerPorIdAsync(int id);
	Task AgregarAsync(Usuario usuario);
}

public class UsuarioRepository : IUsuarioRepository
{
	private readonly AppDbContext _context;
	public UsuarioRepository(AppDbContext context) => _context = context;

	public async Task<Usuario?> ObtenerPorIdAsync(int id) =>
		await _context.Usuarios.FindAsync(id);

	public async Task AgregarAsync(Usuario usuario)
	{
		_context.Usuarios.Add(usuario);
		await _context.SaveChangesAsync();
	}
}
```

Con esto, un controlador o caso de uso depende de `IUsuarioRepository` (una abstracción), no de `AppDbContext` directamente — lo que facilita las pruebas unitarias vistas en la [[Clase 05 - Pruebas Manuales y Automatizadas|Clase 05]].

### DTO (Data Transfer Object)

Un DTO es una clase simple usada solo para transportar datos entre capas (por ejemplo, entre el cliente y la API), evitando exponer directamente las entidades del dominio (que pueden tener campos sensibles como `PasswordHash`).

```csharp
public class UsuarioResponseDto
{
	public int Id { get; set; }
	public string Nombre { get; set; }
	public string Email { get; set; }
	// Nótese: sin PasswordHash
}
```

### Factory Pattern

Centraliza la creación de objetos complejos en un solo lugar, útil cuando construir un objeto requiere lógica adicional (por ejemplo, crear un `JwtService` configurado según el entorno).

```csharp
public static class JwtServiceFactory
{
	public static JwtService Crear(IConfiguration config) => new JwtService(config);
}
```
