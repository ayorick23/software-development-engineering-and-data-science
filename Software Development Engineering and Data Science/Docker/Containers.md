# Containers

## Ciclo de Vida del Contenedor

Un **contenedor** es una instancia ejecutable de una [[Images#Construyendo Imágenes con Dockerfiles|imagen]] de Docker. Su ciclo de vida es el conjunto de estados por los que pasa desde que se crea hasta que se elimina. Comprender este ciclo es fundamental para gestionar tus aplicaciones.

### Estados de un Contenedor

- **Creado (Created):** Es el estado inicial. Un contenedor se crea con `docker create`, pero aún no se ha iniciado. Esto es útil para preconfigurar un contenedor antes de ejecutarlo.
- **En ejecución (Running):** El contenedor está activo y ejecutando el comando definido en la imagen. La mayoría del tiempo los contenedores están en este estado.
- **En pausa (Paused):** El contenedor está en un estado de "suspensión". El proceso principal está congelado, pero el contenedor sigue existiendo y consumiendo recursos. Se puede reanudar.
- **Detenido (Stopped):** El contenedor ha finalizado su ejecución o ha sido detenido manualmente. Sus procesos ya no se están ejecutando, pero su estado y sistema de archivos permanecen intactos. Puedes reiniciarlo o eliminarlo.
- **Eliminado (Deleted):** El contenedor ha sido eliminado del sistema. Todos los datos no persistentes se han perdido.

## Comandos Esenciales del Ciclo de Vida

### `docker create`

Crea un contenedor a partir de una imagen pero no lo inicia.

```docker
docker create <imagen>
```

### `docker run`

El comando más usado. Crea un contenedor y lo inicia en un solo paso.

```docker
docker run <imagen>
```

### `docker logs`

El comando `docker logs` te permite ver la salida estándar (`stdout`) y los errores estándar (`stderr`) de un contenedor en ejecución o que ya ha terminado. Es esencial para la monitorización y depuración de tus aplicaciones, ya que te muestra lo que está sucediendo dentro del contenedor, como mensajes de depuración, errores de aplicación o simplemente la salida de un script.

```docker
docker logs <ID_o_nombre_del_contenedor>
```

### `docker start`

Inicia un contenedor que está en estado `Created` o `Stopped`.

```docker
docker start <id_o_nombre>
```

### `docker stop`

Detiene la ejecución de un contenedor.

```docker
docker stop <id_o_nombre>
```

### `docker restart`

Detiene y luego inicia un contenedor.

```docker
docker restart <id_o_nombre>
```

### `docker pause`

Suspende todos los procesos dentro del contenedor.

```docker
docker pause <id_o_nombre>
```

### `docker unpause`

Reanuda los procesos de un contenedor en pausa.

```docker
docker unpause <id_o_nombre>
```

### `docker kill`

Fuerza la detención de un contenedor (equivalente a `SIGKILL`). Útil cuando `docker stop` falla.

```docker
docker kill <id_o_nombre>
```

### `docker rm`

Elimina un contenedor detenido. Para eliminar un contenedor en ejecución, usa `docker rm -f`.

```docker
docker rm <id_o_nombre>
```

### `docker exec`

El comando `docker exec` te permite ejecutar un comando dentro de un contenedor que ya está en funcionamiento. Esto es fundamental para tareas de mantenimiento, depuración o para interactuar directamente con un proceso en el contenedor sin tener que detenerlo o reiniciarlo. A diferencia de `docker run`, que crea y ejecuta un nuevo contenedor, `docker exec` se conecta a uno existente.

```docker
docker exec <opciones> <ID_o_nombre_del_contenedor> <comando>
```

**IMPORTANTE:** Para consultar todos los argumentos que pueden ser utilizados en estos comandos se usa `--help`.

## Persistencia de Datos: Volúmenes

Por defecto, los datos dentro de un contenedor son temporales. Si el contenedor se elimina, los datos también lo hacen. Para garantizar la persistencia de los datos, Docker ofrece varias opciones, siendo los volúmenes la más recomendada.

### Tipos de Persistencia de Datos

- **Volúmenes:** Son la forma preferida de persistir datos. Son administrados por Docker y residen en una parte del sistema de archivos del host que Docker gestiona.
- **Bind Mounts:** Permiten vincular un archivo o directorio de tu máquina local directamente al sistema de archivos del contenedor. Son útiles para el desarrollo, ya que los cambios en el código local se reflejan instantáneamente en el contenedor.
- **`tmpfs` Mounts:** Almacenan datos en la memoria del host. Son volátiles, pero muy rápidos. Ideales para datos temporales que no necesitas persistir.

### Gestión de volúmenes

### `docker volume`

Crea un volumen con un nombre específico.

```docker
docker volume create <nombre>
```

Muestra una lista de todos los volúmenes en el sistema.

```docker
docker volume ls
```

Muestra información detallada sobre un volumen, incluyendo su ubicación en el sistema de archivos del host.

```docker
docker volume inspect <nombre>
```

Elimina un volumen. Los volúmenes no se eliminan automáticamente cuando eliminas un contenedor.

```docker
docker volume rm <nombre>
```

### Cómo Usar Volúmenes

Puedes adjuntar un volumen a un contenedor al momento de crearlo con la opción `-v` o `--mount`.

- **Con volúmenes nombrados:**

  ```docker
  docker run -d --name mi-db -v datos-db:/var/lib/mysql mysql:5.7
  ```

  En este ejemplo, el volumen `datos-db` se crea (si no existe) y se monta en la ruta `/var/lib/mysql` dentro del contenedor.

- **Con Bind Mounts:**

  ```docker
  docker run -d --name mi-app -v $(pwd)/src:/app/src nginx
  ```

  Aquí, el directorio `src` de la máquina local se monta en el directorio `/app/src` del contenedor.

## Redes en Docker

Docker tiene un sistema de redes robusto que permite a los contenedores comunicarse entre sí y con el mundo exterior. Por defecto, cada contenedor tiene su propio espacio de nombres de red, lo que significa que están aislados.

### Tipos de redes por defecto

- `bridge`: Es la red predeterminada. Los contenedores en esta red pueden comunicarse entre sí y con el host.
- `host`: El contenedor comparte el espacio de nombres de red del host. No hay aislamiento de red entre el contenedor y el host, lo que ofrece un mejor rendimiento pero menos seguridad.
- `none`: Desactiva todas las interfaces de red del contenedor.

### Redes Personalizadas (user-defined bridge networks)

La mejor práctica es crear tu propia red de tipo `bridge`. Esto ofrece varias ventajas:

1. **Resolución de nombres automática:** Los contenedores en una red personalizada pueden comunicarse entre sí usando sus nombres, sin necesidad de direcciones IP.
2. **Aislamiento:** Mantiene la comunicación de los contenedores separada de la red `bridge` por defecto.

### Comandos de Redes

### `docker network`

Muestra las redes disponibles.

```docker
docker network ls
```

Crea una nueva red tipo `bridge`.

```docker
docker network create --driver bridge <nombre>
```

Muestra información detallada sobre una red.

```docker
docker network inspect <nombre>
```

Elimina una red.

```docker
docker network rm <nombre>
```

### Cómo Conectar Contenedores a una Red

Puedes especificar la red al ejecutar el contenedor con la opción `--network`.

```docker
docker run -d --name contenedor1 --network mi-red imagen1
docker run -d --name contenedor2 --network mi-red imagen2
```

En este caso, `contenedor1` y `contenedor2` pueden comunicarse entre sí usando sus nombres.
