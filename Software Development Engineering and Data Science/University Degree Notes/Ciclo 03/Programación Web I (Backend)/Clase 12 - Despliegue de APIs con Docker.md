---
Fecha de creación: 2026-04-13T18:45:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-04-13
---
[[Clase 10 - Autenticación en .NET 8|← Clase anterior]] | [[Clase 13 - CI-CD con GitHub Actions|Clase siguiente →]]

# Despliegue de APIs con Docker
(la [[Clase 10 - Autenticación en .NET 8#Fundamentos de ngrok|Clase 10]] exponía la API local temporalmente con ngrok; Docker resuelve el mismo problema — ejecutar la API fuera de tu máquina — de forma permanente y reproducible)

## ¿Por Qué Contenerizar una API?

Hasta ahora, la API .NET 8 de este curso corre directamente en la máquina de desarrollo, con su propia versión del SDK de .NET, su propia configuración y su propia base de datos local. Ese entorno **no viaja** con el código: el clásico problema de "en mi máquina funciona". Un **contenedor Docker** (ver [[Introduction to Docker]]) empaqueta la API junto con exactamente la versión del runtime de .NET que necesita, de modo que corre igual en la laptop del desarrollador, en el servidor de un compañero o en la nube.

## Dockerfile para una API .NET 8

Un `Dockerfile` describe, paso a paso, cómo construir la imagen de la aplicación. Para una API ASP.NET Core se usa típicamente una compilación en dos etapas (_multi-stage build_): una etapa con el SDK completo para compilar, y una etapa final más liviana, solo con el runtime, para ejecutar.

```dockerfile
# Etapa 1: compilar la aplicación con el SDK completo
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o /app/publish

# Etapa 2: imagen final, liviana, solo con el runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=build /app/publish .

EXPOSE 8080
ENTRYPOINT ["dotnet", "MiApi.dll"]
```

**¿Por qué dos etapas?** El SDK completo (necesario para compilar) pesa varios cientos de MB más que el runtime (necesario solo para ejecutar). La imagen final solo copia el resultado ya compilado (`/app/publish`), sin arrastrar todo el SDK — igual que en la [[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia#Arquitectura Limpia|Arquitectura Limpia]] de la Clase 08, se separa lo que se necesita para construir de lo que se necesita para ejecutar.

## Construir y Ejecutar el Contenedor

```bash
# Construir la imagen a partir del Dockerfile
docker build -t mi-api-dotnet .

# Ejecutar el contenedor, mapeando el puerto 8080
docker run -d -p 8080:8080 --name mi-api mi-api-dotnet
```

Con esto, la API queda accesible en `http://localhost:8080`, ejecutándose dentro de un contenedor aislado — el mismo patrón usado en la [[Clase 03 - Gestores de Bases de Datos NoSQL#Práctica 1: MongoDB con Docker|Clase 03 de Bases de Datos II]] para levantar MongoDB y Cassandra.

## Docker Compose: API + Base de Datos Juntas

En desarrollo, normalmente se necesita la API **y** su base de datos corriendo a la vez. `docker-compose.yml` permite definir ambos servicios y levantarlos con un solo comando:

```yaml
services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      - ConnectionStrings__DefaultConnection=Server=db;Database=InventarioDB;User=sa;Password=Password123!;
    depends_on:
      - db

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=Password123!
    ports:
      - "1433:1433"
```

```bash
docker compose up -d
```

Esto reemplaza la conexión manual a `Server=.;Database=InventarioDB;...` usada desde la [[Clase 04 - Bases de Datos y Code First#Configurar el DbContext|Clase 04]] por una conexión al servicio `db` dentro de la misma red interna de Docker.

## Variables de Entorno y Secretos

Nunca se debe incluir una cadena de conexión real o una clave JWT (ver [[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia#Autenticación Segura con JWT en .NET 8|Clase 08]]) directamente en el `Dockerfile` o en el código fuente. En su lugar, se inyectan como variables de entorno al momento de ejecutar el contenedor, tal como se ve en el `environment` del `docker-compose.yml` anterior.
