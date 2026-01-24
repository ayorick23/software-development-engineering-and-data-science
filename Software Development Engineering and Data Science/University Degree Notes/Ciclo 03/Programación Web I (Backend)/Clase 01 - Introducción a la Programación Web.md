---
Fecha de creación: 2026-01-19T18:45:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-01-19
---
# Arquitectura Cliente-Servidor, Diseño de APIs y Manipulación de Datos

## Historia de la Web y HTTP

La World Wide Web surge a finales de los años 80 en el CERN, gracias a Tim Berners-Lee, quien en 1989 propuso un sistema para compartir documentos de investigación entre científicos de diferentes países.

- **1990:**
	- Se publica la primera versión de HTTP, muy básica (``HTTP/0.9``). Solo permitía hacer solicitudes ``GET`` y recibir documentos de texto.
- **1996:**
	- Nace ``HTTP/1.0``, donde ya se definen cabeceras (headers), códigos de estado y soporte para diferentes tipos de contenido.
- **1997:**
	- Aparece ``HTTP/1.1``, la versión más duradera y aún ampliamente usada. Introduce mejoras como conexiones persistentes y soporte para caché.
- **2015:**
	- ``HTTP/2 trae ``paralelismo, compresión de cabeceras y mejor rendimiento.
- **2022:**
	- ``HTTP/3 se ``basa en QUIC (sobre UDP) y mejora la velocidad y seguridad. _Dato curioso: la primera página web del CERN aún está disponible: [http://info.cern.ch/hypertext/WWW/TheProject.html](http://info.cern.ch/hypertext/WWW/TheProject.html)_

## Arquitectura Cliente-Servidor

La arquitectura cliente-servidor es un modelo de organización en el que las funciones de un sistema se dividen en dos partes claramente diferenciadas.

Por un lado, está el cliente, que normalmente es el programa o dispositivo que utiliza el usuario final y que se encarga de solicitar servicios, datos o recursos. Por otro lado, está el servidor, que es el sistema encargado de recibir esas solicitudes, procesarlas y devolver una respuesta adecuada.

Lo interesante de este modelo es que los clientes no se comunican entre ellos directamente, sino que todas las interacciones pasan por el servidor, lo cual permite centralizar la gestión de datos, mejorar la seguridad y facilitar el control del sistema. Además, un mismo servidor puede atender a múltiples clientes de manera simultánea, lo que lo hace muy útil en entornos donde hay muchos usuarios conectados al mismo tiempo.

Este tipo de arquitectura es la base de la mayoría de aplicaciones y servicios que utilizamos
hoy en día: por ejemplo, cuando abrimos un navegador web y pedimos acceder a una
página, el navegador actúa como cliente y envía la solicitud al servidor donde está alojada
esa página, el cual procesa la petición y envía de vuelta el contenido. De igual forma,
aplicaciones móviles, sistemas bancarios y plataformas de mensajería funcionan bajo este
mismo principio.

En pocas palabras, la arquitectura cliente-servidor permite dividir responsabilidades: el cliente se centra en la interacción con el usuario, mientras que el servidor administra los datos y la lógica central del sistema.

La arquitectura cliente-servidor divide el trabajo en dos roles principales:

- **Cliente:** envía solicitudes (ejemplo: navegador, app móvil, aplicación de escritorio).
- **Servidor:** recibe la solicitud, ejecuta lógica, accede a la base de datos y responde.

### Analogía

Un restaurante es una buena analogía:

- El cliente es el comensal que hace un pedido.
- El servidor es la cocina que prepara el platillo.
- El menú es como la documentación de la API.
- El mesero es el canal de comunicación (``HTTP``).
- El platillo servido es la respuesta.

### Ventajas de este Modelo

- **Separación de responsabilidades:** el cliente se centra en mostrar datos y el servidor en procesarlos.
- **Escalabilidad:** un mismo servidor puede atender a miles de clientes.
- **Seguridad:** el servidor controla qué datos expone.
- **Flexibilidad:** distintos clientes (web, móvil, IoT) pueden consumir la misma API.

## El Protocolo ``HTTP``

El protocolo ``HTTP`` (_HyperText Transfer Protocol_) es uno de los protocolos más importantes en el mundo de las comunicaciones digitales, ya que constituye la base del funcionamiento de la World Wide Web.

Su principal objetivo es establecer un lenguaje común que permita la **transferencia de
información entre un cliente y un servidor**, siguiendo un modelo de arquitectura cliente-
servidor.

En este modelo, el cliente (que puede ser un navegador web, una aplicación móvil o
cualquier software que consuma recursos de internet) envía una solicitud ``HTTP`` (request) al
servidor. Esa solicitud incluye varios elementos: el método ``HTTP`` que indica la acción que se
quiere realizar (por ejemplo: ``GET`` para obtener datos, ``POST`` para enviar información, ``PUT``
para actualizar, ``DELETE`` para eliminar), la URL que identifica el recurso, los encabezados
(headers) con información adicional, y en algunos casos un cuerpo (body) con datos que se
envían al servidor.

EI servidor, por su parte, recibe la solicitud, la procesa y responde con un mensaje ``HTTP`` de
respuesta (response). Este mensaje contiene un código de estado (status code) que informa
si la operación fue exitosa o si ocurrió un error. Por ejemplo:

- ``200 OK`` significa que todo salió bien.
- ``404 Not Found`` indica que el recurso no existe.
- ``500 Internal Server Error`` señala un problema interno en el servidor.

Además del código, la respuesta puede incluir encabezados y un cuerpo con el contenido
solicitado, como una página HTML, un archivo JSON, una imagen, un video o cualquier otro
recurso.

### Ciclo de Vida de una Petición

1. El cliente hace una solicitud con un **método HTTP** hacia una URL.
2. El servidor procesa la petición.
3. El servidor responde con:
	1. Código de estado
	2. Headers (cabeceras)
	3. Body (contenido: HTML, JSON, imagen, etc.)

### Componentes de una Petición ``HTTP``

Cuando un cliente (navegador, app, etc.) envía **una petición HTTP (HTTP Request)** a un
servidor, esa petición está formada por varios elementos que indican **qué recurso se quiere obtener o qué acción se quiere realizar.** Los principales componentes son:

#### Línea de petición (Request Line)

Es la primera línea de la petición e incluye tres partes:

- **Método HTTP:** indica la acción a realizar (ej: ``GET``, ``POST``, ``PUT``, ``DELETE``).
- **Ruta o recurso (URL):** especifica el recurso al que se quiere acceder.
- **Versión de HTTP:** define qué versión del protocolo se está usando (ej: HTTP/1.1, HTTP/2).

**Ejemplo:**

```bash
GET /productos HTTP/1.1
```

### Encabezados (Headers)

Son pares clave-valor que envían información adicional sobre la petición.

Algunos ejemplos comunes:

- **Host:** indica el nombre del servidor (Host: www.ejemplo.com).
- **User-Agent:** información sobre el cliente que hace la petición (navegador, app, etc.).
- **Accept:** formatos que el cliente acepta corno respuesta (application/json, text/html),
- **Authorization:** credenciales para autenticación (tokens, API keys, etc.).

**Ejemplo:**

```bash
Host: www.ejemplo.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer abc123
```

### Cuerpo o Contenido (Body) [Opcional]

No todas las peticiones lo tienen. Se usa principalmente en métodos como ``POST`` o ``PUT`` para enviar datos al servidor (por ejemplo, datos de un formulario, información en formato JSON, archivos, etc.).

**Ejemplo JSON:**

```JSON
{
	"usuario": "dereck",
	"password": "232323"
}
```

### Componentes de una Respuesta ``HTTP``

**Ejemplo:**

```bash
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 5
{
	"id": 101,
	"nombre": "Laptop Acer Aspire",
	"precio": 550.00
}
```

- Línea de estado: ``HTTP/1.1 200 OK``
- Encabezados: ``Content-Type``, ``Content-Length``
- Cuerpo: JSON con los datos del producto

## Métodos ``HTTP``

Cada método define la intención de la solicitud:

- ``GET`` - Obtener recursos (ej: lista de productos).
- ``POST`` - Crear un nuevo recurso.
- ``PUT`` - Reemplazar completamente un recurso.
- ``PATCH`` - Modificar parcialmente un recurso.
- ``OPTIONS`` - Preguntar qué métodos están disponibles.

## Códigos de Estado ``HTTP``

Los códigos de estado son respuestas numéricas que un servidor envía al cliente (como tu
navegador o una app) para indicar el resultado de una solicitud. Son fundamentales en
protocolos como ``HTTP``, y aunque muchas veces no los ves directamente, están detrás de
cada interacción web.

- ``2xx`` — Éxito
	- ``200 OK`` - Petición exitosa.
	- ``201 Created`` - Recurso creado.
- ``4xx`` — Error del cliente
	- ``400 Bad Request`` - Error en la solicitud.
	- ``401 Unauthorized`` -  No autenticado.
	- ``404 Not Found`` - Recurso inexistente.
- ``5xx`` — Error del servidor
	- ``500 Internal Server Error`` - Fallo interno.
	- ``503 Service Unavailable`` - Servicio no disponible.

**Ejemplo en ``.NET 8``:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Activar Swagger
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Middleware Swagger
if (app.Environment.IsDevelopment()) {
	app.UseSwagger();
	app.UseSwaggerUI();
}

// Endpoints
app.MapGet("/", () => new {
	Mensaje = "Servidor Activo",
	Estado = "OK",
	Hora = DateTime.Now
});

app.MapGet("/saludo/{nombre}", (string nombre) => new {
	Mensaje = $"Hola, {nombre}!",
	Estado = "OK",
	Hora = DateTime.Now
})

app.Run();
```
