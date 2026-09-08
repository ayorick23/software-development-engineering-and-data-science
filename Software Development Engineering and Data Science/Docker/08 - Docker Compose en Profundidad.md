---
tags: [docker, docker-compose, cheat-sheet]
---

# 08 — Docker Compose en Profundidad

## El problema que resuelve

Gestionar múltiples contenedores relacionados (un backend, una base de datos, un cache) a mano con `docker run` requiere recordar todos los flags de red, volúmenes y variables de entorno de cada uno, en el orden correcto, cada vez. **Docker Compose** describe todo el conjunto de servicios en un solo archivo declarativo y lo levanta con un comando.

## Un `docker-compose.yml` completo

```yaml
services:
  backend:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://usuario:clave@db:5432/mi_db
    depends_on:
      - db
    volumes:
      - ./app:/app

  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=clave
      - POSTGRES_DB=mi_db
    volumes:
      - datos_postgres:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  datos_postgres:
```

## Comandos principales

```bash
docker compose up              # construye (si hace falta) y levanta todos los servicios
docker compose up -d           # en segundo plano
docker compose up --build      # forzar reconstrucción de imágenes antes de levantar
docker compose down            # detener y eliminar contenedores, redes (los volúmenes se conservan por defecto)
docker compose down --volumes  # además elimina los volúmenes -- destructivo, pierde datos persistentes
docker compose ps              # estado de los servicios del proyecto (similar a docker ps pero por proyecto)
docker compose logs -f backend # logs de un servicio específico, en tiempo real
docker compose exec backend bash   # abrir una terminal dentro de un servicio en ejecución
```

## `depends_on` — orden de arranque, no espera de disponibilidad

```yaml
depends_on:
  - db
```

Un error común: `depends_on` solo garantiza el **orden de arranque** del contenedor (que `db` inicie antes que `backend`), no que el servicio de base de datos ya esté **listo para aceptar conexiones** — Postgres puede tardar unos segundos en inicializar después de que su contenedor arranca. Para esperar disponibilidad real, se usa una condición de healthcheck:

```yaml
services:
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U usuario"]
      interval: 5s
      timeout: 3s
      retries: 5

  backend:
    depends_on:
      db:
        condition: service_healthy   # ahora sí espera a que el healthcheck pase
```

## Redes automáticas entre servicios

Compose crea automáticamente una red compartida para todo el proyecto (ver [[06 - Redes en Docker]]) — cada servicio es accesible desde los demás usando su nombre de servicio como hostname, sin configuración adicional:

```python
# dentro del contenedor "backend", "db" resuelve automáticamente
conexion = "postgresql://usuario:clave@db:5432/mi_db"
```

## Variables de entorno desde un archivo `.env`

```bash
# .env
POSTGRES_PASSWORD=clave_secreta
```

```yaml
services:
  db:
    environment:
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
```

Compose carga automáticamente un archivo `.env` en el mismo directorio — mantener `.env` fuera de control de versiones (`.gitignore`) es la práctica estándar para no filtrar credenciales.

## Entornos distintos: desarrollo vs. producción

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

Un patrón común: un `docker-compose.yml` base con la configuración compartida, y archivos adicionales (`docker-compose.override.yml` para desarrollo, `docker-compose.prod.yml` para producción) que sobreescriben solo lo que cambia entre entornos.

## Ver también

- [[06 - Redes en Docker]]
- [[07 - Volúmenes y Persistencia de Datos]]
- [[11 - Orquestación y Escalabilidad]] — cuándo Compose ya no es suficiente
