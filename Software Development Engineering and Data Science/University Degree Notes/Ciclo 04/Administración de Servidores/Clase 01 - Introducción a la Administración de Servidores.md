---
Fecha de creación: 2026-07-06T21:18:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-07-06
---
# Introducción a la Administración de Servidores

La **administración de servidores** es la gestión integral de equipos informáticos dedicados a alojar datos, aplicaciones y servicios. Sus pilares incluyen la instalación de sistemas operativos (_Windows Server o Linux_), la monitorización de recursos para evitar cuellos de botella, la seguridad mediante cortafuegos y el diseño de copias de seguridad.

---

## Retroalimentación: Hardware de Computadoras

Una computadora es una máquina que procesa información. Para funcionar, necesita componentes específicos que trabajan juntos:

1. Procesador (CPU - Central Processing Unit)
2. Memoria RAM (Random Access Memory)
3. Disco Duro (Storage/Almacenamiento)
4. Placa Madre
5. Fuente de Poder
6. Tarjeta Gráfica

---

## Retroalimentación: Sistemas Operativos

Un **Sistema Operativo (SO)** es software que gestiona el hardware y actúa como intermediario entre tú y la máquina. Sin él, no podrías usar una computadora: tendrías que escribir código binario para cada tarea (encender la pantalla, escribir en el disco, etc.).

### ¿Por qué necesitamos un SO?

El SO resuelve el problema de la complejidad: agrupa las operaciones de hardware en funciones simples que nosotros podemos usar. Ejemplos:

|Sin SO (código máquina)|Con SO (línea sencilla)|
|---|---|
|`10110101 00001100 11001010...` (100+ líneas)|`print("Hola")`|
|`01010011 11110011 00110011...` (cientos de líneas)|`save("archivo.txt")`|

---

## Retroalimentación: Redes Informáticas

Una **red informática** es un conjunto de computadoras conectadas que pueden compartir información y recursos. Sin redes, cada computadora sería una "isla" aislada. Con redes, pueden colaborar y compartir.

Dos tipos principales de redes

### LAN (Local Area Network)

Red de **área local**: conecta dispositivos en una pequeña área geográfica (una oficina, un edificio, una casa).  
  
**Ventajas:**

- Muy rápida (gigabits por segundo).
- Baja latencia (respuesta casi instantánea).
- Permite compartir impresoras, archivos, servicios (DNS, DHCP).

**Ejemplo:** la red WiFi de tu casa es una LAN. Tu teléfono, laptop y smart TV están en la misma red local.

### WAN (Wide Area Network)

Red de **área amplia**: interconecta múltiples redes locales (LANs) a través de grandes distancias geográficas (ciudades, países, continentes).  
  
**Ejemplo:** tu sucursal en la ciudad A se conecta con la sucursal en la ciudad B mediante una WAN. **La WAN más grande que existe es Internet.**  
  
**Costo:** las WANs son caras de mantener. Por eso las empresas usan conexiones dedicadas (fibra óptica renta mensual) o se conectan a través de Internet.

### Cómo se Estructura una Red LAN Típica

1. **Dispositivos finales (workstations)**  
    Las computadoras, impresoras, teléfonos inteligentes se conectan (por cable o WiFi) al siguiente nivel.
2. **Switch (conmutador)**  
    Un dispositivo "inteligente" que interconecta todos esos dispositivos locales. Permite que se comuniquen entre sí a máxima velocidad. Es como el "hub central" de la red local.
3. **Servidor local**  
    Conectado al switch, provee servicios y recursos a todos en la LAN (almacenamiento compartido, autenticación, etc.).
4. **Router (enrutador)**  
    Conecta la LAN completa hacia el exterior (Internet). Es la "puerta de salida" de tu red. Decide qué tráfico se queda local y qué sale hacia el exterior.

### Direcciones IPv4

Una dirección **IPv4** es un identificador único para un dispositivo en una red informática. Este parámetro le permite acceder a ciertos recursos en la red. Todos los dispositivos en la red tienen una IP, incluyendo computadoras, teléfonos y servidores.

```text
IPv4 Addres: 192.168.120.1
```

|Octeto|Decimal|Binario|Significado|
|---|---|---|---|
|1°|192|11000000|Identifica un dispositivo único en la red|
|2°|168|10101000|
|3°|120|01111000|
|4°|1|00000001|

---

## ¿Qué es un Servidor?

Un **servidor** es un sistema informático (hardware, software, o ambos) que proporciona servicios, datos o funciones a otros equipos llamados **clientes**, normalmente a través de una red.

### Analogía: el servidor como mesero de restaurante

Imagina un restaurante:

- **El cliente** es el comensal: hace un pedido (solicitud HTTP).
- **El servidor** es el mesero: recibe el pedido, va a la cocina (procesamiento), y trae el platillo (respuesta).
- **El servicio** es el platillo (página web, archivo, datos de base de datos, email, etc.).

Un restaurante puede tener un mesero (un servidor pequeño) o 50 meseros (una granja de servidores). Un cliente puede comer en un restaurante o en varios (acceder múltiples servicios).

### ¿Por qué necesitamos servidores?

Sin servidores:

- No habría internet (no tendrías dónde ver YouTube, Facebook, Gmail).
- Las empresas no podrían compartir archivos centralizados.
- No habría correo electrónico corporativo.
- Cada persona tendría que guardar su información localmente (muy inseguro).

> [!NOTE] Un dispositivo informático se considera "servidor" si provee de un servicio o recurso a otros dispositivos.

---

## Tipos de servidores según su función

Un servidor se define por el **rol** o **servicio** que proporciona. El mismo hardware puede ejecutar múltiples roles simultáneamente, aunque no siempre es recomendable en ambientes de producción (por rendimiento y seguridad).

Un servidor puede proveer diversos servicios a otros dispositivos llamados "clientes".

![[Drawing 2026-07-07 19.07.31.excalidraw]]

|Tipo de servidor|Función principal|Ejemplo de software|
|---|---|---|
|**Servidor Web**|Entrega páginas HTML, aplicaciones y contenido vía HTTP/HTTPS. Lo que hace funcionar sitios web.|Apache, Nginx, IIS, Node.js|
|**Servidor DNS**|Traduce nombres de dominio (google.com) a direcciones IP (172.217.0.0). Sin él, no podrías navegar por internet escribiendo "google.com".|BIND, Windows DNS Server|
|**Servidor DHCP**|Asigna automáticamente direcciones IP a los dispositivos de la red. Si no existiera, tendrías que configurar manualmente cada computadora.|ISC DHCP, Windows DHCP Server|
|**Servidor de Archivos**|Almacenamiento centralizado con permisos de acceso. "Nube privada" de la empresa.|Samba, Windows File Services, NFS|
|**Servidor de Base de Datos**|Almacena y gestiona datos estructurados. Las aplicaciones consultan los datos aquí.|SQL Server, MySQL, PostgreSQL, Oracle|
|**Servidor de Correo**|Gestiona el envío, recepción y almacenamiento de emails (SMTP, IMAP, POP3).|Microsoft Exchange, Postfix, Zimbra|
|**Servidor de Aplicaciones**|Ejecuta la lógica de negocio. Intermediario entre el cliente y la base de datos.|Apache Tomcat, JBoss, .NET Core|
|**Servidor Proxy**|Actúa como intermediario. Filtra, cachea, protege. Ejemplo: evita que los empleados accedan a ciertos sitios.|Squid, HAProxy, nginx|

---

## Características Principales

Un servidor confiable debe presentar las siguientes características:

- **Disponibilidad:** El servidor debe estar disponible la mayor parte del tiempo (idealmente 24/7)
- **Confiabilidad:** Los datos deben ser confiables (no corromperse, no perderse).
- **Escalabilidad:** Debe poder crecer (más usuarios, más datos) sin reducir en desempeño.
- **Seguridad:** Datos protegidos contra acceso no autorizado.

### Disponibilidad, Confiabilidad y Recursos

A diferencia de tu PC personal (que puedes apagar cuando terminas), un servidor debe permanecer **activo las 24 horas del día, 365 días al año**. Cualquier minuto de inactividad no planificada cuesta dinero.

|Uptime|Downtime por año|Downtime por mes|Caso de uso|
|---|---|---|---|
|**99%**  <br>("Dos nueves")|3 días, 15 horas|7 horas, 18 min|Servicios web básicos, no críticos|
|**99.9%**  <br>("Tres nueves")|8 horas, 45 min|43 min, 49 seg|Estándar en la industria. Alojamientos web, SaaS.|
|**99.99%**  <br>("Cuatro nueves")|52 min, 33 seg|4 min, 23 seg|Plataformas de e-commerce, servicios en la nube (AWS, Azure).|
|**99.999%**  <br>("Cinco nueves")|5 min, 15 seg|26 segundos|Sistemas de misión crítica: bancos, hospitales, infraestructura de seguridad.|

- **99% ("Dos nueves"):** Permite hasta 3.65 días de caída al año. Es común en servicios web básicos.
- **99.9% ("Tres nueves"):** Permite hasta 8.76 horas de caída al año. Estándar en la industria para alojamientos web y servicios estándar.
- **99.99% ("Cuatro nueves"):** Permite hasta 52 minutos de caída al año. Común en plataformas de comercio electrónico y servicios en la nube profesionales.
- **99.999% ("Cinco nueves"):** Permite 5 minutos de caída al año. Es el estándar para sistemas de misión crítica (bancos, redes de salud, infraestructuras de seguridad).

---

### Entorno de Ejecución

En una infraestructura de red, es posible implementar servidores de dos maneras: a nivel físico y a nivel virtual. Ambos cumplirán la función de proveer servicios a los clientes. La diferencia radica en cómo se hace uso del hardware y sus capacidades.

El **servidor físico** consiste en un dispositivo con hardware especializado y dedicado exclusivamente a proveer servicios en la red. El sistema operativo para entorno de servidor corre directamente en él.

El **servidor virtual** se ejecuta dentro de una máquina virtual en un dispositivo anfitrión que provee el hardware respectivo. Es como tener diversas computadoras o servidores dentro de una sola computadora física.

Existen distribuciones especiales para la ejecución en servidores como:

- **Windows Server**
	Ideal para entornos empresariales y Active Directory.
- **Ubuntu Server**
	Flexible, popular y fácil de administrar.
- **Red Hat Enterprise Linux**
	Robusto, seguro y con soporte empresarial.
- **Debian**
	Estable, ligero y confiable para producción.

---

## Windows Server 2022

**Windows Server** es un sistema operativo de **Microsoft** diseñado específicamente para gestionar **redes**, alojar **aplicaciones** y centralizar recursos en entornos empresariales. Permite administrar usuarios y equipos mediante **Active Directory**, habilitar el acceso remoto seguro y optimizar el almacenamiento de datos.

### ¿Qué es Windows Server?

Es la versión "empresarial" de Windows, diseñada para:

- **Gestionar redes:** miles de usuarios y máquinas conectadas.
- **Alojar aplicaciones:** sitios web, bases de datos, servicios de correo.
- **Centralizar recursos:** almacenamiento, permisos, políticas.
- **Funcionar 24/7:** con máximo uptime.

### Diferencias clave vs. Windows 10/11

|Aspecto|Windows 10/11|Windows Server 2022|
|---|---|---|
|**Propósito**|PC personal, usuario único|Servidor, múltiples usuarios/servicios|
|**Licencia**|Una por máquina|Por procesador o por usuario|
|**Active Directory**|No (solo conectarse a dominio)|Sí (administrar dominios)|
|**Roles y características**|Limitados|Completos (DNS, DHCP, IIS, Hyper-V, etc.)|
|**Conexiones remotas simultáneas**|Máximo 2 – 3|Ilimitadas (RDP)|

### Proceso de Instalación de Windows Server y Windows 10

#### Server Core (sin interfaz)

Solo línea de comandos (PowerShell). Menor consumo de recursos, menor superficie de ataque. **Estándar en producción.** Pero es más difícil para aprender.

#### Desktop Experience (con GUI)

Interfaz gráfica completa, similar a Windows 10. Consume más recursos, pero es más fácil para administradores nuevos.
