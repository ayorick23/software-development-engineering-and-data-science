# Orchestration and Scalability

La **orquestación** de [[Containers#Ciclo de Vida del Contenedor|contenedores]] es el proceso de automatizar la gestión, despliegue, escalado, redes y disponibilidad de los contenedores. Cuando pasas de ejecutar un par de contenedores en tu máquina a cientos o miles en un entorno de producción, la gestión manual se vuelve inviable.

Las herramientas de orquestación, a menudo llamadas orquestadores, resuelven estos desafíos al automatizar tareas complejas como:

- **Despliegue y programación:** Decidir en qué servidor (o nodo) se ejecutará un contenedor.
- **Escalado:** Aumentar o disminuir el número de instancias de un servicio según la demanda.
- **Gestión de red:** Conectar contenedores entre sí, incluso si están en diferentes máquinas.
- **Descubrimiento de servicios:** Permitir que los contenedores se encuentren y se comuniquen entre sí de forma automática.
- **Balanceo de carga:** Distribuir el tráfico entre múltiples instancias de un servicio.
- **Alta disponibilidad:** Reiniciar automáticamente los contenedores fallidos o reubicarlos en nodos saludables.

Sin un orquestador, gestionar una infraestructura de contenedores a gran escala sería un caos.

## Herramientas de Orquestación

Existen varias plataformas de orquestación, pero las más populares son:

- **Docker Swarm:** Una herramienta de orquestación nativa de Docker. Es más sencilla de configurar y usar que Kubernetes, lo que la hace una excelente opción para proyectos pequeños o medianos y para quienes ya están familiarizados con el ecosistema de Docker.
- **Kubernetes (K8s):** El orquestador de facto de la industria. Es mucho más potente, flexible y complejo. Se ha convertido en el estándar para la gestión de clústeres a gran escala y se utiliza en la mayoría de las plataformas de nube pública.

## Docker Swarm

**Docker Swarm** es la herramienta de orquestación integrada en Docker Engine. Convierte un grupo de máquinas con Docker en un clúster de Swarm, que se gestiona como una sola entidad virtual. La configuración es simple y rápida, ideal para empezar con la orquestación.

### Conceptos Clave

- **Nodo (Node):** Una máquina física o virtual que ejecuta Docker. En un clúster de Swarm, los nodos pueden ser de dos tipos:

  - **Manager:** Gestiona el estado del clúster, programa las tareas (contenedores) y orquesta el despliegue. Un clúster debe tener al menos un nodo `manager`.
  - **Worker:** Ejecuta las tareas asignadas por los nodos `manager`.

- **Servicio (Service):** La definición de una tarea que se debe ejecutar en el clúster. Un servicio define qué [[Images#Construyendo Imágenes con Dockerfiles|imagen]] usar, cuántas réplicas (instancias) debe haber y qué puertos exponer.
- **Réplica (Replica):** Una instancia en ejecución de un servicio. Swarm se asegura de que el número de réplicas definido se mantenga siempre en el clúster.

### `docker swarm`

Inicializar un clúster Swarm: Este comando convierte la máquina actual en un nodo `manager` y proporciona un token para que otros nodos se unan.

```docker
docker swarm init --advertise-addr <IP_del_nodo_manager>
```

Unir un nodo al clúster.

```docker
docker swarm join --token <TOKEN_DE_UNIÓN> <IP_del_manager>:2377
```

Desplegar un servicio: Esto crea un servicio llamado `mi-servicio` con 3 réplicas que ejecutan la imagen de Nginx.

```docker
docker service create --name mi-servicio --replicas 3 -p 80:80 nginx
```

Escalar un servicio.

```docker
docker service scale mi-servicio=5
```

Listar servicios y nodos.

```docker
docker service ls
docker node ls
```

Eliminar un servicio.

```docker
docker service rm mi-servicio
```

## Kubernetes

**Kubernetes** no es un reemplazo de Docker; en cambio, es una plataforma que utiliza Docker como su motor de contenedores por defecto. La relación es la siguiente: Docker crea el contenedor y Kubernetes lo orquesta a escala.

Mientras que Docker se encarga del **"qué"** (crear un contenedor con una aplicación), Kubernetes se encarga del **"cómo"** y el **"dónde"** (cómo desplegarlo, cómo gestionarlo, dónde ejecutarlo, cómo escalar y balancear la carga).

### Conceptos Clave de Kubernetes

Kubernetes es un sistema mucho más complejo con su propia terminología:

- **Pod:** La unidad más pequeña de despliegue en Kubernetes. Un `Pod` es una o más instancias de contenedores (por lo general, uno) que comparten el mismo espacio de nombres de red y de almacenamiento.
- **Despliegue (Deployment):** Un `Deployment` es una abstracción que describe el estado deseado de la aplicación (por ejemplo, "quiero 5 réplicas del `Pod` con esta imagen"). Kubernetes se encarga de que ese estado se mantenga.
- **Servicio (Service):** Un `Service` es una forma de exponer una aplicación (`Pod`) al exterior. Se encarga del balanceo de carga y de proporcionar un punto de acceso estable a un grupo de Pods.
- **Clúster:** Un conjunto de nodos (máquinas) que ejecutan `Pods` y `Deployments`.

### ¿Cuándo usar qué?

| Característica           | Docker Swarm                                         | Kubernetes                                                      |
| ------------------------ | ---------------------------------------------------- | --------------------------------------------------------------- |
| **Curva de aprendizaje** | Baja. Muy fácil de empezar.                          | Alta. Requiere entender muchos conceptos nuevos.                |
| **Complejidad**          | Sencillo. Ideal para despliegues básicos.            | Muy complejo. Ofrece control granular sobre cada aspecto.       |
| **Escalabilidad**        | Ideal para aplicaciones de tamaño pequeño a mediano. | Diseñado para escalar a cientos o miles de nodos.               |
| **Ecosistema**           | Pequeño. Mayormente enfocado en Docker.              | Grande y maduro. Respaldado por una comunidad masiva y la CNCF. |

En resumen, si necesitas una solución de orquestación rápida y sencilla para tus contenedores, Docker Swarm es un excelente punto de partida. Si tu objetivo es la gestión de infraestructuras a gran escala, con alta disponibilidad y resiliencia, Kubernetes es la opción estándar de la industria.
