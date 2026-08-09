---
Fecha de creación: 2026-04-27T18:00:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-04-27
---
[[Clase 13 - CI-CD con GitHub Actions|← Clase anterior]]

# Documentación de APIs con Swagger / OpenAPI
(Swagger ya apareció activado por defecto desde el primer ejemplo de la [[Clase 01 - Introducción a la Programación Web#Métodos ``HTTP``|Clase 01]]; esta clase explica qué es y cómo aprovecharlo a fondo, como cierre del curso)

## ¿Qué es OpenAPI y Qué es Swagger?

**OpenAPI** es una especificación estándar (un formato JSON/YAML) para describir de forma completa una API REST: sus endpoints, los parámetros que aceptan, los formatos de petición y respuesta, y los códigos de estado posibles. **Swagger** es el conjunto de herramientas (originado antes de que la especificación se estandarizara como OpenAPI) que genera y consume esos documentos — en particular, **Swagger UI**, la interfaz web interactiva donde se puede explorar y probar la API directamente desde el navegador.

## ¿Por Qué Documentar la API?

Sin documentación, cualquier equipo que consuma la API (un frontend, una app móvil, otro servicio) depende de leer el código fuente o de preguntar directamente a quien la escribió. Swagger genera esa documentación **automáticamente a partir del propio código**, por lo que nunca queda desactualizada respecto a lo que la API realmente hace — a diferencia de una documentación escrita a mano por separado.

## Configuración en .NET 8

Como ya se vio en la [[Clase 01 - Introducción a la Programación Web|Clase 01]], activar Swagger requiere solo un par de líneas:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.MapControllers();
app.Run();
```

Con esto, visitar `/swagger` en el navegador muestra una interfaz interactiva con todos los endpoints del `ProductosController` y `UsuariosController` construidos a lo largo del curso, generada automáticamente a partir de sus atributos `[HttpGet]`, `[HttpPost]`, etc.

## Enriquecer la Documentación

Swagger detecta los endpoints automáticamente, pero se puede (y se debe) enriquecer con detalles que el código por sí solo no comunica:

```csharp
/// <summary>
/// Obtiene un producto por su identificador.
/// </summary>
/// <param name="id">Identificador único del producto</param>
/// <response code="200">Producto encontrado</response>
/// <response code="404">No existe un producto con ese id</response>
[HttpGet("{id}")]
[ProducesResponseType(typeof(Producto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public ActionResult<Producto> Get(int id)
{
    var producto = productos.FirstOrDefault(p => p.Id == id);
    return producto == null ? NotFound() : Ok(producto);
}
```

`[ProducesResponseType]` le dice explícitamente a Swagger qué códigos de estado puede devolver el endpoint (recordando los códigos `2xx`/`4xx`/`5xx` de la [[Clase 01 - Introducción a la Programación Web#Códigos de Estado ``HTTP``|Clase 01]]), y los comentarios `///` se convierten en la descripción visible en la interfaz.

## Documentar la Autenticación en Swagger

Dado que muchos endpoints de este curso requieren un JWT (ver [[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia#Autenticación Segura con JWT en .NET 8|Clase 08]]), Swagger UI puede configurarse para aceptar el token y enviarlo automáticamente en cada petición de prueba, evitando volver a Postman (visto en la [[Clase 05 - Pruebas Manuales y Automatizadas|Clase 05]]) solo para probar endpoints protegidos:

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "Ingresa el token JWT: Bearer {token}",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "Bearer" }
            },
            Array.Empty<string>()
        }
    });
});
```

## Buenas Prácticas Finales del Curso

Como cierre, un resumen de los principios transversales que aparecieron en las 14 clases:

1. **Diseño primero, implementación después:** entender REST ([[Clase 02 - Introducción a API REST|Clase 02]]) antes de escribir controladores.
2. **Validar en el servidor, siempre:** nunca confiar solo en el cliente ([[Clase 03 - Formatos de Datos - JSON vs XML#Validación de Entradas en Backend|Clase 03]]).
3. **Separar responsabilidades:** Code First y capas del proyecto ([[Clase 04 - Bases de Datos y Code First|Clase 04]]), Arquitectura Limpia y patrones de diseño ([[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia#Arquitectura Limpia|Clase 08]], [[Clase 09 - SQL Injection y Patrones de Diseño|Clase 09]]).
4. **Probar antes de confiar:** pruebas manuales y automatizadas ([[Clase 05 - Pruebas Manuales y Automatizadas|Clase 05]]).
5. **Nunca confiar en la entrada del usuario:** SQL Injection, hashing y salting ([[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia|Clase 08]], [[Clase 09 - SQL Injection y Patrones de Diseño|Clase 09]]).
6. **Automatizar lo repetible:** CI/CD y contenedores ([[Clase 12 - Despliegue de APIs con Docker|Clase 12]], [[Clase 13 - CI-CD con GitHub Actions|Clase 13]]).
7. **Documentar para quien consume, no solo para quien escribe:** Swagger, esta misma clase.

Una API construida siguiendo esta secuencia — REST bien diseñada, datos validados, segura, probada, desplegada de forma automatizada y bien documentada — está lista para producción.
