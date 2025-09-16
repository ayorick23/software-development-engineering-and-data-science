# Introduction to Docker

## ¿Qué es Docker?

**Docker** es una plataforma de código abierto que facilita la creación, el despliegue y la ejecución de aplicaciones utilizando contenedores. Un **contenedor** es una unidad de software estandarizada que empaqueta todo el código de una aplicación y sus dependencias (bibliotecas, configuraciones, etc.), asegurando que se ejecute de manera consistente en cualquier entorno, ya sea en un portátil, en un servidor local o en la nube.

## El problema que resuelve Docker

Antes de Docker, era común que una aplicación funcionara perfectamente en el entorno de desarrollo de un programador, pero fallara al ser desplegada en producción. Esto se conocía como el problema del "funciona en mi máquina". Las diferencias en los sistemas operativos, las versiones de bibliotecas y las configuraciones eran las principales causas.

Docker resuelve este problema al aislar la aplicación y sus dependencias en un contenedor. Esto garantiza que el entorno de ejecución sea idéntico en todos los lugares, eliminando la inconsistencia.

## Contenedores vs. Máquinas Virtuales (VMs)

Es crucial entender la diferencia entre un contenedor y una máquina virtual.

| Característica     | Contenedores (Docker)                                                                           | Máquinas Virtuales (VMs)                                                                      |
| ------------------ | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Tecnología**     | Virtualización a nivel de sistema operativo                                                     | Virtualización a nivel de hardware                                                            |
| **Recursos**       | Comparten el kernel del sistema operativo del host. Son ligeros y arrancan en segundos.         | Cada VM incluye su propio sistema operativo (SO) completo. Son pesadas y arrancan en minutos. |
| **Portabilidad**   | Altamente portables. Un contenedor puede ejecutarse en cualquier SO que tenga instalado Docker. | Menos portables. Una VM necesita un hipervisor para ejecutarse (ej. VirtualBox, VMware).      |
| **Uso de memoria** | Menor consumo de recursos.                                                                      | Mayor consumo de recursos.                                                                    |

En resumen, los contenedores son una forma más eficiente, ligera y rápida de aislar aplicaciones.

## Conceptos Clave

Para entender cómo funciona Docker, es necesario familiarizarse con tres conceptos principales: **Imágenes**, **Contenedores** y **Docker Hub**.

### Imágenes (Images)
(ver [[Images]])

Una **imagen** de Docker es una plantilla inmutable y de solo lectura que contiene las instrucciones para crear un contenedor. Se puede pensar en ella como una "clase" en la programación orientada a objetos. Una imagen se construye a partir de un [[Images#Construyendo Imágenes con Dockerfiles|Dockerfile]], que es un archivo de texto simple con una lista de comandos para crear la imagen.

- **Ejemplo:** Una imagen de Nginx, una imagen de Ubuntu, una imagen de una aplicación Node.js.

### Contenedores (Containers)
(ver [[Containers]])

Un **contenedor** es una instancia ejecutable de una imagen. Si la imagen es la plantilla, el contenedor es el objeto creado a partir de esa plantilla. Los contenedores son el corazón de Docker, ya que encapsulan la aplicación y todo lo que necesita para ejecutarse.

- **Ejemplo:** Un contenedor que ejecuta un servidor web Nginx, una instancia de una base de datos MySQL. Puedes tener múltiples contenedores ejecutándose a partir de la misma imagen.

### Docker Hub

**Docker Hub** es un registro público y en la nube de imágenes de Docker. Es como un "_GitHub para imágenes de Docker_". Permite a los usuarios encontrar, compartir y almacenar imágenes, tanto oficiales (creadas y mantenidas por los propios proveedores, como Ubuntu, Nginx, Python) como comunitarias. Es una pieza fundamental en el flujo de trabajo de Docker, ya que facilita la distribución de aplicaciones.

- **Ejemplo:** `docker pull nginx` descarga la imagen oficial de Nginx desde Docker Hub.

## Instalación de Docker

Para empezar a trabajar con Docker, primero debes instalar el **Docker Engine**. El proceso varía según el sistema operativo.

- **Linux:** Usa el script de instalación oficial o los paquetes específicos para tu distribución (Debian, Ubuntu, CentOS, etc.).
- **Windows y macOS:** Descarga e instala Docker Desktop, una aplicación de escritorio que incluye el Docker Engine, Docker Compose, Kubernetes y una interfaz gráfica.

Puedes encontrar las instrucciones detalladas en la [documentación oficial de Docker](https://docs.docker.com/get-started/get-docker/).

### Comprobación de la instalación

Una vez instalado, abre una terminal y ejecuta el siguiente comando para verificar que Docker está funcionando correctamente:

```bash
docker --version
```

Deberías ver la versión instalada de Docker.

## Mi primer contenedor: "Hola Mundo"

Para entender la magia de Docker, ejecutemos nuestro primer contenedor. Este comando es un "Hola Mundo" clásico que no necesita una imagen local.

```bash
docker run hello-world
```

**¿Qué hace este comando?**

- `docker run`: Le dice a Docker que inicie un contenedor.
- `hello-world`: Es el nombre de la imagen que queremos ejecutar.

Docker detecta que no tienes la imagen `hello-world` localmente, por lo que la descarga automáticamente desde Docker Hub. Luego, crea y ejecuta un contenedor a partir de esa imagen, el cual simplemente imprime un mensaje de bienvenida y finaliza.

## Comandos básicos

Aquí tienes una lista de comandos esenciales para empezar:

### `docker ps`

Lista los contenedores que se están ejecutando. El comando ``docker ps -a`` muestra todos los contenedores, incluso los que están detenidos.

```docker
docker ps
```

### `docker images`

Muestra las imágenes que tienes descargadas en tu máquina.

```docker
docker images
```

### `docker pull`

Descarga una imagen desde Docker Hub.

```docker
docker pull <nonbre_imagen>
```

### `docker stop`

Detiene un contenedor en ejecución.

```docker
docker stop <id_contenedor>
```

### `docker rm`

Elimina un contenedor detenido.

```docker
docker rm <id_contenedor>
```

### `docker rmi`

Elimina una imagen.

```docker
docker rmi <id_imagen>
```
