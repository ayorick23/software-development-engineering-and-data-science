---
Fecha de creación: 2026-09-14T18:12:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-09-14
---
# DNS en Ubuntu Server

## ¿Qué es DNS?

**DNS (_Domain Name System_)** es un sistema distribuido y jerárquico que asocia nombres con información: direcciones IP, servidores de correo y otros servicios.

Funciona como un directorio repartido entre organizaciones: usamos nombres fáciles de recordar y DNS permite encontrar sus direcciones. Ningún servidor conserva todos los nombres de Internet.

- **Dominio y subdominio.** Un dominio es una parte del árbol de nombres. `example.com` contiene nombres como `www.example.com` o `ventas.example.com`.
- **Zona y delegación.** Una zona es la porción del árbol administrada por un conjunto de servidores autoritativos. Delegar un subdominio transfiere su administración a otra zona.
- **Resolvedor recursivo.** Busca respuestas por cuenta del cliente y conserva resultados en caché. Puede pertenecer al proveedor de Internet, a la empresa o a un operador público.
- **Servidor autoritativo.** Publica los datos de las zonas a su cargo. Su respuesta procede de esos datos, no de una búsqueda recursiva para el cliente.

En `www.example.com.`, leído de derecha a izquierda, el punto final representa la **raíz**, `com` es el dominio de nivel superior o **TLD**, y `example` y `www` son etiquetas sucesivas. El nombre completo se llama **FQDN**. Véase [RFC 1034: conceptos de DNS](https://www.rfc-editor.org/rfc/rfc1034).

> **DNS, DHCP y HTTP cumplen funciones diferentes.** DHCP entrega parámetros de red; DNS permite consultar nombres; HTTP/HTTPS transporta el contenido web. DNS no asigna por sí mismo la IP de tu equipo ni descarga la página solicitada.


---

## Recorrido de una Consulta

Al abrir `https://www.example.com/noticias`, DNS consulta solo `www.example.com`. Sin respuestas en caché, el recorrido es:

1. **El equipo consulta sus mecanismos locales.** La aplicación y el sistema pueden tener respuestas guardadas; la resolución del sistema también puede consultar `/etc/hosts` según su configuración.
2. **El cliente pregunta al resolvedor.** Solicita, por ejemplo, un registro A o AAAA. La petición recursiva pide una respuesta final o un error.
3. **El resolvedor consulta la raíz.** Recibe una referencia a los servidores responsables de `.com`; la raíz no le entrega normalmente la IP del sitio.
4. **Consulta los servidores del TLD.** Estos indican los servidores autoritativos de `example.com`.
5. **Consulta al autoritativo.** Obtiene el registro solicitado o un alias que requiere continuar la resolución.
6. **Devuelve y conserva la respuesta.** El resolvedor guarda los datos según su TTL y responde al cliente. Después, el navegador se conecta al servicio web usando la dirección obtenida.

|Transporte|Uso|Dato importante|
|---|---|---|
|UDP 53|Consultas DNS tradicionales frecuentes.|Evita establecer una conexión antes de cada intercambio.|
|TCP 53|También consultas ordinarias; transferencias completas de zona.|Permite reintentar cuando una respuesta UDP está truncada. No debe bloquearse por asumir que DNS solo usa UDP.|

---

## Registros

Cada consulta indica un **nombre** y un **tipo**: un nombre puede tener registros A, MX y otros.

|Tipo|Función|Ejemplo o interpretación|
|---|---|---|
|`A`|Nombre → IPv4.|`web → 192.168.50.20`|
|`AAAA`|Nombre → IPv6.|`web → 2001:db8::20` (dirección de documentación).|
|`CNAME`|Alias → otro nombre.|`www → web.aula.test.`; no apunta directamente a una IP.|
|`MX`|Servidor que recibe correo para el dominio.|`10 mail.aula.test.`; un número menor tiene mayor preferencia.|
|`NS`|Servidores autoritativos de una zona.|`ns1.aula.test.`; también participa en delegaciones.|
|`SOA`|Metadatos administrativos de la zona.|Servidor principal, contacto, número de serie y temporizadores.|
|`PTR`|IP → nombre, mediante una zona inversa.|`192.168.50.20 → web.aula.test.`|
|`TXT`|Texto utilizado por otras aplicaciones.|Verificación de dominios o políticas como SPF; DKIM y DMARC también publican datos mediante TXT.|
|`SRV`|Localización de servicios: destino y puerto.|Incluye prioridad y peso; utilizado, por ejemplo, en descubrimiento de servicios de directorio.|
|`CAA`|Indica qué autoridades pueden emitir certificados.|Ayuda a controlar la emisión; no sustituye el certificado HTTPS.|

---

## Caché, TTL y la Llamada "Propagación"

El **TTL (Time To Live)** indica cuánto puede conservarse un dato en caché: `3600` segundos equivalen a una hora. No representa saltos de red como el TTL de IP.

Una IP guardada a las 10:00 con TTL de una hora puede seguir respondiéndose hasta las 11:00, aunque cambie a las 10:10. Un resolvedor sin esa caché puede obtener la nueva de inmediato.

### TTL corto

Reduce el tiempo de permanencia de datos antiguos, a cambio de más consultas y mayor dependencia de la disponibilidad del DNS.

### TTL largo

Reduce consultas repetidas, pero los cambios pueden tardar más en ser observados por quienes guardaron la respuesta anterior.

### Caché negativa

También se conserva temporalmente que un nombre o un tipo de registro no existe. Crear el registro no borra esas respuestas anteriores.

---

## Cómo Funciona DNS

### ¿Quién administra cada parte?

Las funciones de IANA coordinan la zona raíz y sus delegaciones. Los registros administran TLD como `.com` o `.sv`; los registradores permiten registrar dominios. El titular elige un proveedor de DNS autoritativo y publica sus registros. El alojamiento web puede estar en una empresa distinta. [IANA: administración de la raíz](https://www.iana.org/domains/root).

El titular registra el dominio, configura su delegación NS en el registrador y publica los registros en el proveedor DNS. Si un NS está dentro del dominio delegado, pueden necesitarse direcciones de apoyo (**glue**) para evitar una dependencia circular.

|Situación|Papel de DNS|Implicación|
|---|---|---|
|Abrir una tienda o una aplicación|Encuentra direcciones del servicio.|El servicio puede estar activo y parecer caído si falla su DNS.|
|Enviar correo|MX identifica los receptores; TXT publica políticas.|Una web puede funcionar aunque el correo falle por registros distintos.|
|Usar una CDN|Puede devolver destinos según región, red o política del proveedor.|Dos usuarios pueden recibir IP diferentes para el mismo nombre.|
|Acceder a una intranet|El DNS interno publica nombres privados.|Un resolvedor público normalmente no conoce esos nombres.|
|Ofrecer redundancia|Varios autoritativos sirven una zona; los secundarios reciben sus cambios.|Deben evitarse dependencias de una sola máquina, red o ubicación.|

---

## Seguridad: Autenticidad y Privacidad

|Tecnología o control|Qué aporta|Qué no resuelve por sí solo|
|---|---|---|
|DNSSEC|Firmas y una cadena de confianza para validar autenticidad e integridad de datos firmados.|No cifra consultas ni convierte una página maliciosa en segura.|
|DNS sobre TLS (DoT)|Cifra el enlace DNS con el resolvedor; puerto TCP 853 habitual.|El resolvedor sigue pudiendo conocer las consultas.|
|DNS sobre HTTPS (DoH)|Transporta DNS dentro de HTTPS, normalmente por el puerto 443.|No valida automáticamente los datos de la zona ni cifra toda la cadena entre servidores DNS.|
|ACL y firewall|Limitan quién puede consultar, usar recursión o transferir zonas.|No sustituyen actualizaciones, redundancia ni monitoreo.|

El **envenenamiento de caché** intenta introducir respuestas falsas. El **secuestro de DNS** puede alterar la configuración del cliente, del router o de la cuenta que administra el dominio. Un resolvedor abierto también puede ser utilizado para amplificar ataques: la recursión de un DNS interno debe limitarse a su red.

> [!WARNING] Validar no es firmar.
>  `dnssec-validation auto;` permite que BIND valide respuestas externas firmadas; no firma automáticamente una zona que tú creaste. Un autoritativo público normalmente sirve sus zonas sin ofrecer recursión al público.

---

## Ubuntu Server: Servicio DNS con BIND9

**BIND9** sirve zonas propias y resuelve nombres externos. Esta referencia configura un DNS interno en Ubuntu Server 24.04 LTS; adapta las direcciones y la interfaz a tu red.

|Elemento del ejemplo|Valor|
|---|---|
|Red interna|`192.168.50.0/24`|
|Servidor DNS, con IP fija ya configurada|`192.168.50.10` / `ns1.aula.test`|
|Servidor web / correo|`192.168.50.20` / `192.168.50.30`|
|Zona de ejemplo|`aula.test`, privada y no delegada en Internet.|

**Instalación y archivos de servicio:**

```bash
sudo apt update
sudo apt install bind9 bind9-utils dnsutils

# Respaldo antes de editar una instalación existente
sudo cp -a /etc/bind /etc/bind.backup-$(date +%Y%m%d-%H%M%S)
sudo nano /etc/bind/named.conf.options
```

Adapta el bloque existente de `named.conf.options`, sin duplicar `options`. Usa reenviadores aprobados por tu organización; el ejemplo muestra dos públicos.

**/etc/bind/named.conf.options:**

```json
acl "red_interna" {
    127.0.0.1;
    192.168.50.0/24;
};

options {
    directory "/var/cache/bind";
    listen-on { 127.0.0.1; 192.168.50.10; };
    listen-on-v6 { none; };
    recursion yes;
    allow-query { red_interna; };
    allow-recursion { red_interna; };
    allow-query-cache { red_interna; };
    allow-transfer { none; };
    forwarders { 1.1.1.1; 8.8.8.8; };
    forward only;
    dnssec-validation auto;
};
```

a ACL limita el acceso a la LAN y al equipo local. `forward only` envía las búsquedas externas a los reenviadores: si ninguno responde, no hay búsqueda directa alternativa. Las zonas locales se responden desde BIND. `allow-transfer { none; };` desactiva transferencias hasta definir secundarios autorizados. Este ejemplo escucha solo por IPv4; aun así puede consultar registros AAAA.

La escucha explícita en `127.0.0.1` permite coexistir con el stub habitual de `systemd-resolved` en `127.0.0.53`; no es necesario desactivarlo. Revisa otros procesos que ya usen las direcciones y el puerto 53.

---

## Zona Directa e Inversa

La zona directa responde por nombres; la inversa utiliza `in-addr.arpa` para IPv4. Para la red `192.168.50.0/24`, su nombre es `50.168.192.in-addr.arpa`. IPv6 utiliza `ip6.arpa` y otra representación.

**Archivo de declaración:**

```bash
sudo nano /etc/bind/named.conf.local
```

Añade estas declaraciones sin duplicar zonas existentes. `type primary` identifica al servidor que mantiene los archivos originales.

**/etc/bind/named.conf.local:**

```json
zone "aula.test" {
    type primary;
    file "/etc/bind/db.aula.test";
};

zone "50.168.192.in-addr.arpa" {
    type primary;
    file "/etc/bind/db.192.168.50";
};
```

**Crear o editar la zona directa:**

```bash
sudo nano /etc/bind/db.aula.test
```

/etc/bind/db.aula.test

```
$TTL 3600
$ORIGIN aula.test.
@ IN SOA ns1.aula.test. admin.aula.test. (
    2026091201 ; serial: aumentar con cada cambio
    3600       ; refresh: consulta de cambios del secundario
    900        ; retry: reintento tras un fallo
    604800     ; expire: límite sin actualizar el secundario
    300        ; límite para caché negativa
)
@    IN NS    ns1.aula.test.
@    IN MX 10 mail.aula.test.
ns1  IN A     192.168.50.10
web  IN A     192.168.50.20
mail IN A     192.168.50.30
www  IN CNAME web.aula.test.
```

`@` representa el origen de la zona; `admin.aula.test.` en el SOA equivale a `admin@aula.test`. El serial identifica la versión: fecha más contador es una convención. MX anuncia el destino del correo, pero no instala ese servicio.

**Crear o editar la zona inversa:**

```bash
sudo nano /etc/bind/db.192.168.50
```

**/etc/bind/db.192.168.50:**

```
$TTL 3600
$ORIGIN 50.168.192.in-addr.arpa.
@ IN SOA ns1.aula.test. admin.aula.test. (
    2026091201
    3600
    900
    604800
    300
)
@  IN NS  ns1.aula.test.
10 IN PTR ns1.aula.test.
20 IN PTR web.aula.test.
30 IN PTR mail.aula.test.
```

El propietario `20` se completa como `20.50.168.192.in-addr.arpa.`. Si cambia un archivo de zona, incrementa su serial. Un secundario compara números de serie y puede actualizarse mediante transferencias completas (AXFR) o incrementales (IXFR).

**Validación y carga del servicio:**

```bash
# Comprobar sintaxis y cargar las zonas para validarlas
sudo named-checkconf -z
sudo named-checkzone aula.test /etc/bind/db.aula.test
sudo named-checkzone 50.168.192.in-addr.arpa /etc/bind/db.192.168.50

# Solo después de corregir los errores de validación
sudo systemctl enable --now named
sudo systemctl restart named
sudo systemctl status named --no-pager

# Tras posteriores cambios de registros, ya validados
sudo rndc reload
```

`named` es el servicio de BIND en esta referencia de Ubuntu. `named-checkzone` debe terminar con `OK`; una comprobación correcta no demuestra por sí sola que el firewall o los clientes estén configurados.

**Reglas para una LAN con UFW:**

```bash
sudo ufw allow from 192.168.50.0/24 to 192.168.50.10 port 53 proto udp
sudo ufw allow from 192.168.50.0/24 to 192.168.50.10 port 53 proto tcp
sudo ufw status
```

Estas reglas tienen efecto cuando UFW está activo. Su activación debe contemplar primero el acceso administrativo SSH. Si la salida está restringida, también deben permitirse consultas UDP y TCP 53 a los reenviadores. La configuración de referencia sirve a la LAN; no publica `aula.test` en Internet.

---

## Ubuntu como Cliente DNS

Instalar BIND no cambia automáticamente el DNS de los demás equipos. Los clientes deben usar `192.168.50.10`, configurado manualmente o anunciado por DHCP.

**Inspección y selección temporal con systemd-resolved:**

```bash
ip -br address
resolvectl status

# Sustituir enp0s3 por la interfaz real del cliente
sudo resolvectl dns enp0s3 192.168.50.10
sudo resolvectl domain enp0s3 aula.test '~.'
resolvectl query web.aula.test

# Retirar los ajustes temporales de esa interfaz
sudo resolvectl revert enp0s3
```

`~.` indica una ruta DNS para todos los dominios por esa interfaz; `aula.test` añade un dominio de búsqueda. Son cambios temporales: Netplan o la gestión de red pueden reemplazarlos.

### Configuración persistente con Netplan

Integra estos campos en el archivo existente de `/etc/netplan/`, bajo la interfaz correcta y conservando los demás ajustes. El cliente recibe IP y rutas por DHCP con `networkd`; evita definiciones contradictorias entre archivos.

**Fragmento de referencia para el cliente:**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
      dhcp4-overrides:
        use-dns: false
      nameservers:
        addresses: [192.168.50.10]
        search: [aula.test]
```

**Validar y probar cambios de red:**

```bash
sudo netplan generate
sudo netplan try
resolvectl status
```

`netplan try` permite confirmar la configuración con un tiempo de espera para revertirla si no se confirma. Si también usas DHCPv6 con `networkd`, revisa sus `dhcp6-overrides` y las fuentes DNS de IPv6.

> [!INFO] 127.0.0.53 puede ser normal.
> Es el stub local de `systemd-resolved`, no necesariamente el DNS final; `resolvectl status` muestra los servidores por interfaz. Evita editar a mano un `/etc/resolv.conf` administrado por el sistema. Como segundo DNS interno, utiliza otro servidor que conozca la misma zona: un DNS público no reemplaza esa función.
