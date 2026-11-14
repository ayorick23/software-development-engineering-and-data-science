---
Fecha de creación: 2026-10-05T17:49:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-10-05
---
# Protección de Red con Firewall

## ¿Qué protege un firewall?

Un **firewall** aplica reglas al tráfico de red para decidir qué comunicaciones se permiten y cuáles se bloquean. Reduce la superficie expuesta, pero no vuelve seguro por sí solo a un servicio vulnerable.

En un servidor, el firewall puede actuar sobre paquetes que **entran**, **salen** o **atraviesan** el equipo. Cada decisión usa datos como dirección de origen y destino, protocolo, puerto, interfaz y estado de la conexión.

- **Entrada.** Protege los servicios locales. Por ejemplo, permite HTTPS y limita SSH a la red administrativa.
- **Salida.** Controla conexiones iniciadas por el servidor, como actualizaciones, DNS o acceso a una API externa.
- **Reenvío.** Se aplica cuando el equipo funciona como router y mueve paquetes entre redes diferentes.

---

## Cómo se decide el destino de un paquete

|Criterio|Pregunta sencilla|Ejemplo|
|---|---|---|
|Origen|¿De dónde viene?|`192.168.50.0/24`|
|Destino|¿A qué equipo o dirección llega?|`192.168.50.10`|
|Protocolo|¿Cómo se transporta?|TCP, UDP o ICMP|
|Puerto|¿Qué servicio busca?|TCP 22 para SSH|
|Interfaz|¿Por qué conexión física o virtual pasa?|`enp0s3`|
|Estado|¿Inicia una conexión o responde a una existente?|`NEW` o `ESTABLISHED`|
|Acción|¿Qué debe hacer el firewall?|Permitir, descartar, rechazar o registrar|

### Permitir, descartar y rechazar

**Allow/accept** deja continuar el paquete. **Deny/drop** lo descarta sin responder. **Reject** lo bloquea y comunica el rechazo. Descartar revela menos información, pero puede hacer más lento el diagnóstico; rechazar produce un error inmediato y claro.

---

## Firewall con estado

Los firewalls modernos suelen mantener información de las conversaciones mediante **seguimiento de conexiones** o _connection tracking_. Así reconocen que un paquete de regreso pertenece a una comunicación ya permitida.

|Estado conceptual|Significado|Decisión habitual|
|---|---|---|
|`NEW`|Intenta iniciar una comunicación.|Evaluar el servicio, origen y política.|
|`ESTABLISHED`|Pertenece a una conexión reconocida.|Permitir la respuesta de una conexión autorizada.|
|`RELATED`|Está vinculado con otra conexión conocida.|Permitir si el protocolo y la política lo requieren.|
|`INVALID`|No puede asociarse correctamente a una conexión.|Normalmente descartar y, si conviene, registrar.|

Si un servidor consulta una página web, la respuesta puede volver aunque la política de entrada sea restrictiva: el firewall reconoce la conexión iniciada desde dentro. Esto no equivale a permitir que Internet inicie conexiones nuevas hacia cualquier puerto.

> [!INFO] TCP tiene una sesión visible mediante banderas y secuencias
> UDP no establece sesión, pero el firewall conserva temporalmente una relación entre paquetes para tratar respuestas esperadas.

---

## Diseñar una política antes de escribir reglas

Una política segura parte de **negar por defecto las conexiones entrantes** y permitir solo lo necesario. La lista debe nacer del propósito real del servidor, no de puertos abiertos “por si acaso”.

1. **Inventariar servicios.** Identifica qué procesos escuchan, en qué direcciones y puertos, y quién debe usarlos.
2. **Definir orígenes.** Distingue Internet, LAN, red administrativa, VPN y contenedores.
3. **Establecer mínimos.** Permite solo los protocolos y destinos imprescindibles.
4. **Conservar la administración.** Antes de activar reglas remotas, autoriza SSH desde un origen controlado y mantén acceso de consola.
5. **Observar y revisar.** Comprueba reglas, sockets y registros; elimina permisos que dejaron de ser necesarios.

|Servidor|Tráfico entrante típico|Decisión prudente|
|---|---|---|
|Web público|TCP 80 y 443 desde Internet|Permitir web; restringir SSH a administración.|
|DNS autoritativo|UDP y TCP 53|Permitir ambos transportes a los clientes previstos.|
|Base de datos interna|Puerto del motor desde servidores de aplicación|No exponer a Internet; limitar por subred u origen.|
|Servidor de archivos|Puertos del protocolo desde la LAN|Permitir únicamente desde la red local necesaria.|

---

## UFW en Ubuntu Server

**UFW (Uncomplicated Firewall)** ofrece una interfaz sencilla para administrar un firewall local. La decisión final ocurre en **Netfilter**, el subsistema de filtrado del kernel de Linux; UFW traduce instrucciones fáciles de leer a reglas que el sistema puede aplicar.

- **Netfilter.** Es la capacidad del kernel que inspecciona y decide el destino del tráfico.
- **UFW.** Es la herramienta de administración que simplifica políticas y reglas comunes.
- **Reglas persistentes.** UFW conserva su configuración para aplicarla en los siguientes arranques.

**Instalación e inspección inicial**

```bash
sudo apt update
sudo apt install ufw
sudo ufw status verbose
sudo ufw show added
sudo ufw app list
sudo ufw app info OpenSSH
```

UFW suele estar instalado, pero inicialmente inactivo. `status verbose` muestra estado y políticas; `show added` presenta las reglas agregadas en su forma original. Los perfiles de aplicación se guardan en `/etc/ufw/applications.d/`. Consulta también la [documentación oficial de firewall en Ubuntu Server](https://documentation.ubuntu.com/server/how-to/security/firewalls/).

> [!WARNING] No mezcles cambios manuales de distintos administradores de firewall sin comprender cómo interactúan
> Una regla añadida fuera de UFW puede existir aunque no aparezca de la misma forma en `ufw status`.

---

## Políticas y reglas importantes de UFW

Este conjunto representa un servidor administrado por SSH desde `192.168.50.0/24` y con un sitio web público. Debe adaptarse a la red real.

**Política base y servicios autorizados**

```bash
# Definir políticas predeterminadas
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Permitir SSH solo desde la red administrativa
sudo ufw allow proto tcp from 192.168.50.0/24 to any port 22 comment 'SSH administracion'

# Permitir el servicio web público
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'

# Revisar antes de activar
sudo ufw show added
sudo ufw --dry-run enable
sudo ufw enable
sudo ufw status verbose
```

`--dry-run` muestra la operación prevista, pero no prueba que una sesión remota sobrevivirá a una política incorrecta. Si administras por SSH, permite primero el acceso correcto y conserva otra consola disponible.

**Consultar, borrar y recargar**

```bash
sudo ufw status numbered
# Borra la regla que actualmente ocupa el número 3
sudo ufw delete 3
# Alternativa: borrar repitiendo la regla original
sudo ufw delete allow 80/tcp
sudo ufw reload
sudo ufw disable
```

---

## Reglas según servicio, origen e interfaz

**Ejemplos de alcance**

```bash
# Usar un perfil instalado por la aplicación
sudo ufw allow 'Nginx Full'

# DNS para una LAN: se requieren UDP y TCP
sudo ufw allow from 192.168.50.0/24 to any port 53 proto udp
sudo ufw allow from 192.168.50.0/24 to any port 53 proto tcp

# PostgreSQL solo desde un servidor de aplicaciones
sudo ufw allow proto tcp from 192.168.50.20 to any port 5432

# HTTP únicamente al entrar por la interfaz indicada
sudo ufw allow in on enp0s3 to any port 80 proto tcp

# Limitar intentos repetidos de nuevas conexiones SSH
sudo ufw limit 22/tcp comment 'Limite SSH'
```

Un perfil como `Nginx Full` es cómodo, pero conviene inspeccionarlo con `sudo ufw app info 'Nginx Full'`. Si la aplicación usa puertos personalizados, el perfil puede no coincidir con la configuración real.

> **`limit` no reemplaza una defensa de acceso.** Reduce intentos repetidos desde una misma dirección según el mecanismo de UFW, pero no sustituye llaves SSH, autenticación robusta, actualizaciones ni herramientas de bloqueo adaptativo.

### Tráfico hacia el host y tráfico reenviado

`ufw allow` se refiere normalmente al propio servidor. Un equipo que funciona como router necesita `ufw route ...`, reenvío habilitado y, según el diseño, NAT. Abrir un puerto local no autoriza automáticamente el tránsito entre dos interfaces.

---

## Registros y observación

Los registros ayudan a reconocer bloqueos, errores de diseño y actividad inusual. No deben activarse sin medida: una red ruidosa puede producir demasiados eventos y ocultar lo importante.

**Estado, registro y sockets**

```bash
sudo ufw logging low
sudo ufw status verbose
sudo journalctl -k --grep='UFW' --since today
sudo tail -f /var/log/ufw.log
sudo ss -luntp
```

La disponibilidad de `/var/log/ufw.log` depende del sistema de registro configurado. El diario del kernel suele ser una fuente útil. `ss -luntp` responde otra pregunta: qué procesos escuchan realmente.

|Dato de registro|Qué indica|
|---|---|
|`IN` / `OUT`|Interfaz de entrada y de salida.|
|`SRC` / `DST`|Dirección de origen y destino.|
|`PROTO`|Protocolo, como TCP, UDP o ICMP.|
|`SPT` / `DPT`|Puerto de origen y de destino, cuando aplica.|
|`UFW BLOCK`|El paquete coincidió con una decisión de bloqueo registrada.|

---

## Diagnóstico por capas

**Comandos de comprobación**

```bash
# Direcciones, rutas y procesos que escuchan
ip -br address
ip route
sudo ss -luntp

# Política del firewall
sudo ufw status numbered
sudo ufw status verbose

# Pruebas desde otro equipo autorizado
nc -vz 192.168.50.10 22
curl -I http://192.168.50.10

# Observar tráfico en el servidor; Ctrl+C termina
sudo tcpdump -ni any 'tcp port 22 or tcp port 80'
```

| Síntoma                                            | Qué comprobar                                                                     |
| -------------------------------------------------- | --------------------------------------------------------------------------------- |
| Conexión rechazada inmediatamente                  | Proceso escuchando, dirección de enlace y posible regla `reject`.                 |
| La conexión agota el tiempo                        | Ruta, firewall, seguridad del proveedor, NAT y llegada de paquetes con `tcpdump`. |
| Funciona desde el servidor, no desde la LAN        | Que el servicio no escuche solo en `127.0.0.1` y que exista una regla de entrada. |
| Funciona en la LAN, no desde Internet              | Dirección pública, NAT del router, reglas perimetrales y política del proveedor.  |
| Una regla parece correcta, pero falla              | Origen real, IPv4/IPv6, interfaz, protocolo y prioridad de otras capas.           |
| El puerto está permitido, pero la aplicación falla | Estado y registros de la aplicación; el firewall no corrige errores internos.     |

---

## Límites y defensa por capas

- **Vulnerabilidades.** Si un servicio permitido tiene una falla, el firewall no la repara. Mantén paquetes y aplicaciones actualizados.
- **Identidad.** Permitir una IP no autentica a la persona. Usa llaves, MFA cuando aplique y cuentas con privilegios mínimos.
- **Contenido permitido.** El tráfico malicioso puede viajar por HTTPS autorizado. Se necesitan controles de aplicación, validación y monitoreo.
- **Varias fronteras.** Un firewall de nube, router o hipervisor puede bloquear antes que UFW. Todas las capas deben coincidir.

La protección sólida combina firewall, exposición mínima, actualizaciones, configuración segura, cifrado, autenticación, copias de seguridad y observación. También conviene enlazar servicios internos solo a la dirección que necesitan: no todo proceso debe escuchar en `0.0.0.0` o `::`.

> [!INFO] UFW es principalmente un firewall del host
> En contenedores, máquinas virtuales y plataformas de nube pueden existir rutas y reglas adicionales; comprueba el comportamiento real de esa arquitectura.
