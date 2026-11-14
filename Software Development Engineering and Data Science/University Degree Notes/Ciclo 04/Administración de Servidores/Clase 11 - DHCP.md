---
Fecha de creación: 2026-09-21T17:27:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-09-21
---
# DHCP

## ¿Qué es DHCP?

**DHCP (Dynamic Host Configuration Protocol)** entrega automáticamente parámetros de red a los equipos: dirección IP, máscara, puerta de enlace y servidores DNS. Centraliza la configuración y reduce errores manuales.

Al conectar una laptop al Wi-Fi, el punto de acceso permite entrar a la red; DHCP puede entregarle después su configuración IP. Esta clase se centra en **DHCPv4**.

- **En casa.** El router suele incluir el servidor DHCP que configura teléfonos, computadoras y televisores.
- **En una institución.** Un servicio central administra rangos distintos para aulas, oficinas y visitantes.
- **Para equipos conocidos.** Una reserva permite que una impresora reciba una dirección específica mediante DHCP.

---

## Partes de un servicio DHCP

| Parte             | Función                                                       | Ejemplo                                           |
| ----------------- | ------------------------------------------------------------- | ------------------------------------------------- |
| Cliente           | Solicita y aplica la configuración.                           | Una laptop con asignación automática.             |
| Servidor          | Ofrece direcciones y registra concesiones.                    | Ubuntu con ISC DHCP Server.                       |
| Subred o ámbito   | Conjunto de parámetros para una red.                          | `192.168.50.0/24`.                                |
| Pool o rango      | Direcciones disponibles para asignación dinámica.             | `192.168.50.100–192.168.50.199`.                  |
| Exclusión         | Direcciones que no se reparten dinámicamente.                 | Dejar la IP del router fuera del pool.            |
| Reserva           | Asocia un identificador del cliente con una IP.               | MAC de impresora → `192.168.50.50`.               |
| Concesión o lease | Permiso temporal para usar una dirección.                     | Una IP concedida por ocho horas.                  |
| Opciones          | Parámetros adicionales entregados al cliente.                 | Router, DNS y dominio.                            |
| Relay             | Reenvía mensajes entre clientes y un servidor de otra subred. | Router que comunica una VLAN con el DHCP central. |

### IP manual, dinámica y reservada

Una **IP manual** se configura en el equipo y no depende de DHCP. Una **dinámica** procede del pool y puede cambiar. Una **reserva** conserva la asignación centralizada de una IP concreta, pero el cliente sigue necesitando contactar al servicio.

> **Planificación:** en una red `/24` convencional hay 254 direcciones de host; el pool anterior contiene solo 100. La dirección de red `.0`, el broadcast `.255` y las IP manuales no se reparten. En ISC DHCP, el `range` debe definirse de modo que no incluya esas direcciones.

---

## DORA: cómo obtiene una IP el cliente

La adquisición inicial habitual consta de cuatro mensajes. **DORA** resume sus nombres en inglés:

1. **Discover — descubrir.** El cliente sin IP busca servidores mediante broadcast: “¿Quién puede configurarme?”.
2. **Offer — ofrecer.** Un servidor propone una IP y sus parámetros. La oferta todavía no confirma la concesión.
3. **Request — solicitar.** El cliente anuncia la oferta elegida e identifica al servidor; los demás pueden retirar sus propuestas.
4. **ACK — confirmar.** El servidor confirma la concesión. El cliente comprueba y aplica la configuración.

---

## Concesiones, renovación y agotamiento

La IP se presta durante un tiempo. El cliente intenta renovar antes del vencimiento y normalmente conserva su dirección si el servidor confirma.

|Momento|Comportamiento|Ejemplo: concesión de 8 horas|
|---|---|---|
|T1: renovación|Contacta normalmente por unicast al servidor original.|A las 4 horas, con T1 del 50 %.|
|T2: revinculación|Si no renovó, busca por broadcast otro servidor que pueda confirmarla.|A las 7 horas, con T2 del 87,5 %.|
|Vencimiento|Sin confirmación, debe dejar de usar la IP y buscar configuración otra vez.|A las 8 horas.|

Estos porcentajes son los valores predeterminados del protocolo cuando no se indican otros temporizadores. Una renovación exitosa reinicia los plazos de la concesión. [Temporizadores en RFC 2131](https://www.rfc-editor.org/rfc/rfc2131).

- **Red de visitantes.** Concesiones cortas recuperan antes las IP de equipos que se van, pero generan más renovaciones.
- **Oficina estable.** Concesiones largas reducen tráfico, pero retienen más tiempo direcciones de clientes ausentes.

---

## ¿Qué parámetros viajan en DHCP?

|Dato|Opción DHCPv4|Ejemplo|
|---|---|---|
|Dirección ofrecida|Campo `yiaddr` del mensaje, no una opción.|`192.168.50.100`|
|Máscara|1 — subnet mask.|`255.255.255.0`|
|Puerta de enlace|3 — routers.|`192.168.50.1`|
|Servidores DNS|6 — domain name servers.|`192.168.50.10`|
|Dominio|15 — domain name.|`aula.test`|
|Duración|51 — IP address lease time.|`28800` segundos.|
|Tipo de mensaje / servidor|53 / 54.|ACK e identificador del servidor.|

El cliente también puede enviar un identificador (opción 61) y una lista de parámetros solicitados (55). El identificador no siempre coincide con la MAC; esto importa al interpretar concesiones y reservas. [RFC 2132: opciones DHCP](https://www.rfc-editor.org/rfc/rfc2132).

> [!INFO] Entregar `aula.test` como dominio no crea registros DNS
> Para registrar nombres automáticamente se necesita una integración de DNS dinámico (DDNS) configurada por separado.

---

## Configuración con ISC DHCP Server

La práctica principal utiliza **ISC DHCP Server**, cuyo proceso se llama `dhcpd`. ISC es la organización que desarrolló el producto; por eso, al hablar del servidor heredado, el nombre preciso es **ISC DHCP**. Es una implementación clásica y todavía aparece en muchas redes y materiales académicos. Su archivo principal es `/etc/dhcp/dhcpd.conf`.

> [!WARNING] Contexto actual
> ISC DHCP llegó al fin de su ciclo de vida en diciembre de 2022. Ubuntu lo considera obsoleto y sin soporte desde 24.04 LTS. Esta práctica sirve para aprender y mantener instalaciones heredadas; para un despliegue nuevo se recomienda Kea. Realiza el laboratorio con una versión donde el paquete esté disponible, por ejemplo Ubuntu Server 22.04 LTS. [Estado de soporte según ISC](https://kb.isc.org/docs/isc-dhcp-eol-dates).

|Elemento|Valor del ejemplo|
|---|---|
|Interfaz LAN del servidor|`enp0s3`, con IP fija `192.168.50.10/24` ya configurada.|
|Router existente|`192.168.50.1`|
|DNS existente|`192.168.50.10`, BIND9 de la clase anterior.|
|Pool dinámico|`192.168.50.100–192.168.50.199`|
|Reserva de impresora|`192.168.50.50`, fuera del pool.|
|Concesión predeterminada / máxima|8 horas / 8 horas.|

> [!WARNING] Antes de habilitar el servicio
> Utiliza una LAN administrada por ti y coordina el DHCP existente del router o hipervisor. Dos servidores sin coordinación pueden entregar configuraciones incompatibles. Sustituye interfaz, IP y MAC por las reales.

**Instalación y selección de interfaz**

```bash
ip -br address
sudo apt update
sudo apt install isc-dhcp-server
sudo systemctl stop isc-dhcp-server
sudo cp -a /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.backup-$(date +%Y%m%d-%H%M%S)
sudo nano /etc/default/isc-dhcp-server
```

En `/etc/default/isc-dhcp-server`, indica la interfaz conectada a la LAN. El nombre debe coincidir con la salida de `ip -br address`:

```bash
INTERFACESv4="enp0s3"
INTERFACESv6=""
```

Después edita `/etc/dhcp/dhcpd.conf`. Las directivas terminan en punto y coma; los bloques `subnet` y `host` se delimitan con llaves.

**/etc/dhcp/dhcpd.conf**

```json
authoritative;
default-lease-time 28800;
max-lease-time 28800;

option domain-name "aula.test";
option domain-name-servers 192.168.50.10;

subnet 192.168.50.0 netmask 255.255.255.0 {
  range 192.168.50.100 192.168.50.199;
  option subnet-mask 255.255.255.0;
  option broadcast-address 192.168.50.255;
  option routers 192.168.50.1;

  host impresora {
    hardware ethernet 02:00:00:00:00:50;
    fixed-address 192.168.50.50;
  }
}
```

`authoritative` indica que este servidor es la autoridad para las subredes declaradas. `range` define las IP dinámicas; las directivas `option` entregan parámetros al cliente; el bloque `host` crea la reserva. La MAC es ilustrativa y debe reemplazarse.

**Validar y activar**

```bash
sudo dhcpd -t -4 -cf /etc/dhcp/dhcpd.conf

# Continuar solo si la comprobación termina sin errores
sudo systemctl enable --now isc-dhcp-server
sudo systemctl status isc-dhcp-server --no-pager

# Para cargar cambios posteriores, después de validarlos
sudo systemctl restart isc-dhcp-server
```

---

## Ubuntu como cliente DHCP

En el **equipo cliente**, adapta el archivo existente de `/etc/netplan/`. Este ejemplo usa `networkd`; retira de esa interfaz las IP, rutas y DNS manuales que deban sustituirse por DHCP.

**Configuración del cliente con Netplan**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
```

**Aplicación e inspección**

```bash
sudo netplan generate
sudo netplan try
ip -4 address show dev enp0s3
ip route
resolvectl status enp0s3
networkctl status enp0s3

# Solicitar renovación si la interfaz la gestiona systemd-networkd
sudo networkctl renew enp0s3
```

`netplan try` solicita confirmación y puede revertir el cambio si no se confirma. Cambiar la IP puede interrumpir SSH; realiza estos ajustes con acceso de consola disponible. Renovar no implica recibir una IP distinta.

> [!INFO] La configuración DHCP es para el cliente
> El servidor ISC DHCP conserva su IP fija. Si quedó `dhcp4-overrides: use-dns: false` de la clase anterior, elimínalo o ajústalo para aceptar el DNS anunciado.

---

## Diagnóstico y lectura de concesiones

Inspección en el servidor

```bash
sudo journalctl -u isc-dhcp-server --since today --no-pager
sudo tail -n 30 /var/lib/dhcp/dhcpd.leases

# Observar mensajes en la interfaz de la LAN; Ctrl+C termina la captura
sudo apt install tcpdump
sudo tcpdump -ni enp0s3 -vv 'udp port 67 or udp port 68'
```

`dhcpd.leases` conserva información como IP, identificadores, estados y vencimientos. Puede incluir declaraciones antiguas de una misma concesión: contar bloques no equivale a contar clientes activos. No lo edites mientras `dhcpd` esté funcionando.

|Síntoma|Qué revisar|
|---|---|
|No aparece Discover en la captura.|Interfaz, cable o Wi-Fi, VLAN, modo de red de la VM y cliente DHCP.|
|Llega Discover, pero no sale Offer.|Servicio, subred seleccionada, pool libre y errores en los registros.|
|Sale Offer, pero no llega al cliente.|Camino de regreso, relay, filtros y switch virtual.|
|Recibe una IP de otra red.|Otro servidor DHCP, VLAN incorrecta o DHCP del hipervisor.|
|Tiene IP, pero no Internet.|Opción de router, rutas y conectividad del gateway.|
|Llega a servicios por IP, pero no por nombre.|DNS anunciado, acceso al puerto 53 y configuración del resolvedor.|
|La reserva no coincide.|MAC real, aleatorización de MAC, identificador del cliente y concesión anterior.|

---

## Seguridad y disponibilidad

- **Servidor no autorizado.** Un DHCP ajeno puede anunciar DNS o gateways incorrectos. DHCP snooping en switches compatibles permite confiar solo en puertos autorizados.
- **Agotamiento del pool.** Solicitudes abusivas pueden consumir las concesiones. Limita tasas en el acceso y monitorea la ocupación.
- **Reserva no es autenticación.** Una MAC se puede cambiar o suplantar. La reserva organiza direcciones, pero no demuestra identidad ni autoriza acceso.
- **Redundancia coordinada.** La alta disponibilidad necesita compartir estado o un diseño que evite conflictos. Copiar el mismo pool en dos servidores no basta.

Si el servidor cae, los clientes con concesión vigente pueden seguir usando su configuración; los nuevos y quienes llegan al vencimiento tendrán problemas. Conserva respaldos de configuración y concesiones, y supervisa errores y direcciones disponibles.
