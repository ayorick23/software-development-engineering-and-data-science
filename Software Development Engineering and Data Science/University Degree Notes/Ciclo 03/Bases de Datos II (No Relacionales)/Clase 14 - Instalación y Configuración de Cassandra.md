---
Fecha de creación: 2026-05-02T13:57:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-05-02
---
[[Clase 13 - Estrategias de Implementación Cassandra|← Clase anterior]]

# Instalación y Configuración de Apache Cassandra

## Entornos NoSQL

Para aplicaciones de Big Data y alta disponibilidad, Cassandra es la opción líder debido a su arquitectura distribuida:

- **Descentralización:** No existen nodos maestros; todos los nodos tienen el mismo rol.
- **Replicabilidad:** Configuración flexible del factor d/replicación por Keyspace.
- Flexibilidad: Instalación nativa vs. Contenedores (Docker).

## Instalación en Linux (Debian/Ubuntu)

Uso de repositorios oficiales **APT** para una gestión de paquetes estable:

```bash
# 1. Añadir repositorio
echo "deb http://wa.apache.org/dist/cassandra/debian 41x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list

# 2. Añadir claves GPG
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -

# 3. Instalar
sudo apt update && sudo apt install cassandra
```

- Ideal para producción
- Servicio Systemd
- Paquetes Firmados

## Instalación en MacOS (HomeBrew)

Homebrew es un gestor de paquetes popular en Mac, que gestiona todas las librerías y paquetes necesarios para instalar/actualizar software en Mac.

**Lema:**

**The Missing Package Manager for macOS (or Linux), (EI manejador de paquetes apple se le Olvido hacer)**

**Comandos:**

```bash
Ibin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

EI estándar de facto en macOS es Homebrew. Cassandra requiere Java 8 o 11 para funcionar correctamente.

**Comandos:**

```bash
# Instalar
brew install cassandra

# Iniciar como Servicio
brew services start cassandra
```

## Instalación en Windows

Históricamente complejo, hoy Windows se maneja principalmente mediante:

- **Docker Desktop:** EI método recomendado (aislamiento total).
- **Paquete Binario:** Ejecutado directamente vía JVM.
- **WSL2:** Permite usar la instalación de Linux dentro de Windows.

_Nota: Requiere configuración manual de variables de entorno de Java si se
usa el binario._

## Cassandra con Docker (Nodo Único)

Levantar una instancia de desarrollo en segundos:

```bash
docker run --name cassandra-dev -d -p 9042:9042 cassandra:latest
```

- `--name`: Alias para el contenedor.
- ``-p 9042``: Puerto nativo de CQL.
- ``-d``: Modo detached (segundo plano).
- ``cqlsh``: ``docker exec -it cassandra-dev cqlsh``

## Clustering Local Multi-Nodo

Orquestación avanzada mediante **Docker Compose**. Permite simular alta disponibilidad en una sola máquina.

### Demo en VS Code

Procederemos a revisar el archivo ``docker-compose.yml``:

- Configuración de Red Interna.
- Definición de Nodo Semilla (Seed).
- Dependencias de arranque entre nodos.
- Mapeo de volúmenes persistentes.

```yaml
services:
	cassandra-seed:
		image: cassandra:latest
		container_name: cassandra-seed
		environment:
			- CASSANDRA_CLUSTER_NAME=UeesC1uster
			- CASSANDRA_SEEDS=cassandra-seed
			- CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFi1eSnitch
			- MAX_HEAP_SIZE=512M
			- HEAP_NEWSIZE=100M
			- JVM_OPTS=-Xms512M -Xmx512M
		ports:
			- "9042:9042"
		volumes:
			- cassandra-seed-data:/var/lib/cassandra
		networks:
			- cassandra-net
			  
	cassandra-node2:
		image: cassandra:latest
		container_name: cassandra-node2
		environment:
			- CASSANDRA_CLUSTER_NAME=UeesC1uster
			- CASSANDRA_SEEDS=cassandra-seed
			- CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFi1eSnitch
			- MAX_HEAP_SIZE=512M
			- HEAP_NEWSIZE=100M
			- JVM_OPTS=-Xms512M -Xmx512M
		depends_on:
			- cassandra-seed
		volumes:
			- cassandra-node2-data:/var/lib/cassandra
		networks:
			- cassandra-net
			  
	cassandra-node3:
		image: cassandra:latest
		container_name: cassandra-node3
		environment:
			- CASSANDRA_CLUSTER_NAME=UeesC1uster
			- CASSANDRA_SEEDS=cassandra-seed
			- CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFi1eSnitch
			- MAX_HEAP_SIZE=512M
			- HEAP_NEWSIZE=100M
			- JVM_OPTS=-Xms512M -Xmx512M
		depends_on:
			- cassandra-seed
		volumes:
			- cassandra-node3-data:/var/lib/cassandra
		networks:
			- cassandra-net
networks:
	cassandra-net:
	
volumes:
	cassandra-seed-data:
	cassandra-node2-data:
	cassandra-node3-data:
```

## Sesión de Debugging: Acceso a Logs

El monitoreo de logs es crítico para diagnosticar fallos de arranque, problemas de memoria o latencia de red.

**Rutas Estándar:**

```bash
/var/log/cassandra/system.log
```

**En Entorno Docker:**

```bash
docker logs cassandra-node1
# Modo Seguimiento (Real-time)
docker logs -f cassandra-node1
```

## ¿Qué es NODETOOL?

Es la navaja suiza para administradores de Cassandra. Permite monitorear, administrar y ejecutar tareas de mantenimiento sobre nodos activos.

- **Gestión Operativa:** Verificación de clúster.
- **Métricas JVM:** Estado de la memoria y Garbage Collection.
- **Mantenimiento:** Compactación y limpieza de datos.

```bash
docker exec -it cassandra-seed nodetool status
```

## Verificación de Estado General

Comandos esenciales de monitoreo (vía Docker):

- ``nodetool status``: Verifica si los nodos están **UN** (Up/Normal) o **DN** (Down/Normal).
- ``nodetool cfstats`` o ``tablestats``: Revisa estadísticas detalladas de lectura/escritura por tabla. (Incluida la latencia)
- ``nodetool info``: Obtiene información sobre la JVM, el protocolo y la carga de datos.

## IDE: DataStax DevCenter

Herramienta visual nativa diseñada específicamente por DataStax para CQL.

- **Específico:** Optimizado para Cassandra.
- **Visual:** Exploración jerárquica de Keyspaces.
- **Debug:** Depuración guiada de scripts CQL.

## IDE: DBeaver (Universal)

La herramienta universal más popular para desarrolladores multi-base de datos.

- **Soporte JDBC:** Requiere instalar el driver oficial de DataStax dentro de la app.
- **Open Source:** Versión gratuita potente.
- **Visualización:** Modo Rejilla, Texto y JSON.

## IDE: DBVisualizer 25.3.2

Suite profesional multi-plataforma de alto rendimiento.

- **Gráficos de Esquema:** Generación automática de diagramas (ERD).
- **Entornos Corporativos:** Gestión de múltiples conexiones masivas.
- **Fetch Size:** Configurable para no saturar memoria en grandes volúmenes.
