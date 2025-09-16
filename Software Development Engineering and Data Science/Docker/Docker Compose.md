# Docker Compose

**Docker Compose** es una herramienta para definir y ejecutar aplicaciones multi-contenedor de Docker. Con Compose, usas un archivo **YAML** para configurar los servicios de tu aplicación. Luego, con un solo comando, puedes crear y arrancar todos los servicios desde esa configuración.

El problema que resuelve Compose es la complejidad de gestionar múltiples contenedores de forma manual. Imagina una aplicación web que requiere un servidor web (como Nginx), una base de datos (como PostgreSQL) y un servicio de backend (como una [[APIs#APIs|API en Python]]). Sin Compose, tendrías que ejecutar comandos [[Containers#`docker run`|docker run]] por separado para cada contenedor, asegurándote de que todos estén en la misma red y que los volúmenes estén adjuntos correctamente.

Con Docker Compose, todo este proceso se simplifica. Defines la arquitectura de tu aplicación en un único archivo, `docker-compose.yml`, y Compose se encarga de crear, enlazar y gestionar los contenedores por ti.

## Diferencia entre YAML y JSON
(ver [[JSON and YAML|YAML y JSON en Python]])

Docker Compose usa **YAML** (YAML Ain't Markup Language) para su archivo de configuración. Es importante entender la diferencia con **JSON** (JavaScript Object Notation), ya que ambos son formatos para la serialización de datos.

| Característica     | YAML                                                                                         | JSON                                                                    |
| ------------------ | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Sintaxis**       | Se basa en la indentación y espacios. No usa corchetes `{}` ni comas `,`.                    | Usa corchetes, llaves y comas.                                          |
| **Legibilidad**    | Es más legible para humanos, especialmente para archivos de configuración complejos.         | Más verboso y menos legible debido a la repetición de sintaxis.         |
| **Comentarios**    | Permite comentarios usando el símbolo `#`. Esto es crucial para documentar la configuración. | No permite comentarios.                                                 |
| **Tipos de datos** | Soporta tipos de datos más complejos y puede ser más flexible.                               | Soporta tipos de datos básicos como objetos, arrays, strings y números. |

En resumen, **YAML** es más adecuado para la configuración de archivos como el `docker-compose.yml` debido a su enfoque en la legibilidad y su capacidad para incluir comentarios, facilitando la colaboración y el mantenimiento.

## Archivo `docker-compose.yml`

El archivo `docker-compose.yml` es el corazón de Docker Compose. Es donde defines la estructura de tu aplicación en términos de servicios, redes y volúmenes.

La estructura básica del archivo es la siguiente:

```yaml
version: "3.8" # La versión de la sintaxis de Docker Compose
services:
  <nombre_del_servicio_1>:
    # Configuración del primer contenedor
  <nombre_del_servicio_2>:
    # Configuración del segundo contenedor
# Otras secciones opcionales (networks, volumes, etc.)
```

### Parámetros Clave para un Servicio

Dentro de la sección `services`, puedes definir la configuración de cada contenedor. Los parámetros más comunes son:

- `image`: Especifica la [[Images#Construyendo Imágenes con Dockerfiles|imagen]] de Docker a usar para el servicio.

  - **Ejemplo:**

    ```docker
    image: postgres:13
    ```

- `build`: Si necesitas construir la imagen a partir de un Dockerfile en lugar de usar una imagen existente, usas `build`. Puedes especificar la ruta al directorio del Dockerfile.

  - **Ejemplo:**

    ```docker
    build: .
    ```

- `container_name`: Asigna un nombre específico al contenedor creado.

  - **Ejemplo:**

    ```docker
    container_name: mi-api
    ```

- `ports`: Mapea los puertos del host a los del contenedor.

  - **Ejemplo:**

    ```docker
    - "8080:80" # Mapea el puerto 8080 del host al 80 del contenedor
    ```

- `volumes`: Monta volúmenes o `bind mounts` para persistir datos.

  - **Ejemplo:**

    ```docker
    - ./data:/var/lib/mysql
    ```

- `environment`: Define variables de entorno dentro del contenedor. Son muy útiles para configurar credenciales o parámetros de bases de datos.

  - **Ejemplo:**

    ```docker
    - POSTGRES_PASSWORD=mi_clave_secreta
    ```

- `depends_on`: Declara la dependencia de un servicio a otro. Esto garantiza que un servicio se inicie antes que otro.

  - **Ejemplo:**

    ```docker
    - depends_on: - db # El servicio actual se iniciará solo después de que el servicio db se inicie
    ```

## Comandos de Docker Compose

Una vez que has configurado tu archivo `docker-compose.yml`, gestionar tu aplicación es muy simple con unos pocos comandos:

### `docker compose`

El comando principal. Construye (si es necesario), crea y arranca todos los servicios definidos en el archivo. Si agregas la opción `-d` (`docker compose up -d`), los servicios se ejecutarán en segundo plano (modo _detached_).

```docker
docker compose up
```

Detiene y elimina todos los contenedores, redes y volúmenes (si no están especificados para ser persistentes) creados por `up`. Es la forma limpia de apagar toda la aplicación.

```docker
docker compose down
```

Muestra el estado de todos los servicios. Es similar a [[Introduction to Docker#`docker ps`|docker ps]], pero enfocado en los servicios de tu proyecto.

```docker
docker compose ps
```

Muestra los _logs_ de los servicios. Puedes especificar el nombre de un servicio para ver solo sus logs.

```docker
docker compose logs
```

Construye o reconstruye las imágenes de los servicios que tienen una instrucción `build`.

```docker
docker compose build
```

Ejecuta un comando en un contenedor en ejecución. Es muy útil para entrar en el contenedor.

```docker
docker compose exec <servicio> <comando>
```

- **Ejemplo:**

  ```docker
	  docker compose exec web bash # Abre una terminal dentro del contenedor del servicio web
  ```

## Flujo de Trabajo Común con Compose

1. **Definición:** Crea el archivo `docker-compose.yml` en la raíz de tu proyecto, describiendo todos los servicios de la aplicación.
2. **Ejecución:** En la terminal, ejecuta `docker compose up` para construir y arrancar la aplicación completa.
3. **Desarrollo:** Haz cambios en tu código. Docker Compose, en muchos casos (como con `bind mounts`), detectará los cambios automáticamente.
4. **Parada:** Cuando termines de trabajar, ejecuta `docker compose down` para limpiar los contenedores y redes.

Con Docker Compose, pasas de gestionar contenedores individuales a gestionar toda una aplicación de manera cohesiva, lo que agiliza enormemente el desarrollo y la implementación.
