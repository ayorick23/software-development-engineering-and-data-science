---
Fecha de creación: 2026-07-27T19:19:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-07-27
---
# DNS y DHCP en Windows Server

## Servicios que hacen habitable una red

Una red no funciona solamente porque los equipos estén conectados por un cable, un switch o Wi-Fi. Para que las personas puedan trabajar con facilidad, hacen falta servicios que organicen la comunicación. En esta clase estudiaremos dos de los más importantes: **DNS** y **DHCP**.

DNS permite usar nombres comprensibles en lugar de memorizar direcciones numéricas. DHCP permite que los equipos reciban automáticamente la información necesaria para comunicarse. Juntos reducen errores, simplifican la administración y hacen posible que una red crezca sin tener que configurar manualmente cada computadora.

### DNS: el directorio

Responde preguntas como: _“¿Qué dirección tiene `archivos.empresa.local`?”_. Es un servicio de resolución de nombres.

### DHCP: el asignador

Responde preguntas como: _“¿Qué dirección debo usar y qué datos de red necesito?”_. Es un servicio de configuración automática.

---

## Fundamentos: antes de hablar de DNS y DHCP

Para comunicarse dentro de una red TCP/IP, cada dispositivo necesita una identidad de red y debe saber por dónde enviar la información. DNS y DHCP trabajan sobre estos conceptos.

### Dirección IP

Es la dirección lógica de un equipo dentro de una red. En IPv4 tiene cuatro números, por ejemplo `192.168.10.25`. Es útil para las máquinas, pero difícil de recordar para una persona.

### Máscara de subred

Indica qué parte de la dirección IP representa la red y qué parte representa al equipo. Una máscara común es `255.255.255.0`, también escrita como `/24`.

### Puerta de enlace

Es el dispositivo que permite salir de la red local para llegar a otras redes, como Internet. En el laboratorio puede ser `192.168.10.1`.

### Servidor DNS

Es la dirección del equipo al que el cliente preguntará por nombres. Si el cliente no conoce un DNS válido, puede tener conectividad IP y aun así no encontrar los recursos por su nombre.

### Ejemplo de configuración de un cliente

|Parámetro|Valor de ejemplo|Para qué sirve|
|---|---|---|
|Dirección IPv4|`192.168.10.120`|Identifica al cliente dentro de la LAN.|
|Máscara|`255.255.255.0`|Define la red `192.168.10.0/24`.|
|Puerta de enlace|`192.168.10.1`|Permite alcanzar otras redes.|
|DNS preferido|`192.168.10.10`|Permite resolver nombres del dominio y externos.|
|Sufijo DNS|`empresa.local`|Completa nombres cortos como `archivos`.|

---

## DNS

**DNS** significa _Domain Name System_. Su misión es asociar nombres con información de red, principalmente direcciones IP. Gracias a DNS podemos escribir `servidor1.empresa.local` en lugar de aprender una lista de números.

Cuando un usuario abre un recurso por nombre, el equipo consulta primero si ya conoce la respuesta. Si no la tiene, pregunta al servidor DNS configurado. Si el servidor es responsable de ese nombre, responde; si no lo es, puede consultar otros servidores o usar un reenviador.

**Recorrido de una consulta:**

```plain text
Nombre -> Consulta -> Respuesta -> Conexión
```

---

## DNS aplicado a Windows Server

Windows Server incluye el rol **DNS Server**. Puede instalarse desde **Server Manager** o mediante PowerShell. En una infraestructura con Active Directory, suele instalarse junto al controlador de dominio, aunque también puede funcionar en un servidor dedicado.

### Elementos habituales en una empresa

- **Zona del dominio.**
	Por ejemplo, `empresa.local`. Contiene los registros internos de equipos y servicios.
- **Zona inversa.**
	Permite comprobar qué nombre corresponde a una IP y ayuda en diagnóstico.
- **Actualizaciones dinámicas.**
	Permiten que los equipos registren o actualicen sus datos DNS sin crearlos manualmente.
- **Reenviadores.**
	Resuelven consultas externas sin dejar que los clientes dependan de DNS públicos directos.

### Registro de Recursos

| Registro | Tipo      | Descripción                                                            |
| -------- | --------- | ---------------------------------------------------------------------- |
| A        | Host IPv4 | Mapea un FQDN a una dirección IPv4                                     |
| AAAA     | Host IPv6 | Mapea un FQDN a una dirección IPv6 (128 bits)                          |
| PTR      | Puntero   | Se ubica en la Zona Inversa. Mapea un IP a un nombre de dominio (FQDN) |
| CNAME    | Alias     | Crea un apodo o nombre alternativo que apunta a otro registro A.       |
### Instalación básica

1. Abre **Server Manager → Manage → Add Roles and Features**.
2. Selecciona **Role-based or feature-based installation** y el servidor de destino.
3. Marca **DNS Server**, acepta las herramientas requeridas y completa la instalación.
4. En **Tools → DNS**, revisa las zonas existentes o crea las zonas directa e inversa necesarias.

```powershell
# Instalar el rol DNS y sus herramientas de administración
Install-WindowsFeature DNS -IncludeManagementTools

# Consultar un nombre desde PowerShell
Resolve-DnsName archivos.empresa.local
```

---

## DHCP

**DHCP** significa _Dynamic Host Configuration Protocol_. Su objetivo es entregar automáticamente los parámetros que un equipo necesita para trabajar en una red TCP/IP.

Sin DHCP, cada cliente tendría que configurarse manualmente con una IP, máscara, puerta de enlace y DNS. En una oficina con pocos equipos esto ya puede generar errores; en una empresa con cientos de equipos sería lento, difícil de controlar y propenso a direcciones repetidas.

### El proceso DORA

Cuando un equipo inicia y todavía no tiene IP, no sabe a qué servidor DHCP debe dirigirse. Por eso el primer intercambio usa mensajes de difusión en la red local. El proceso se recuerda con las letras **DORA**:

1. **Discover.** El cliente anuncia: “Busco un servidor DHCP”.
2. **Offer.** El servidor ofrece una dirección y parámetros.
3. **Request.** El cliente solicita formalmente la oferta elegida.
4. **Acknowledge.** El servidor confirma la concesión y el tiempo de uso.

---

## DHCP aplicado a Windows Server

Windows Server incluye el rol **DHCP Server**. La consola DHCP permite crear ámbitos, revisar las concesiones activas, definir reservas y configurar opciones. En un dominio Active Directory, el servidor DHCP debe estar **autorizado**; esto ayuda a evitar servidores no aprobados en la red.

### Configuración básica de un ámbito

1. Instala el rol **DHCP Server** desde Server Manager y completa la configuración posterior a la instalación.
2. Autoriza el servidor en Active Directory, si forma parte de un dominio.
3. Abre **Tools → DHCP**, expande IPv4 y selecciona **New Scope**.
4. Define el rango, máscara, exclusiones y duración de la concesión.
5. Configura la puerta de enlace, el DNS interno y el sufijo de dominio.
6. Activa el ámbito y prueba desde un cliente configurado para obtener la dirección automáticamente.

```powershell
# Instalar DHCP y sus herramientas
Install-WindowsFeature DHCP -IncludeManagementTools

# Crear un ámbito IPv4 de ejemplo
Add-DhcpServerv4Scope -Name "LAN-Principal" `
	-StartRange 192.168.10.100 `
	-EndRange 192.168.10.200 `
	-SubnetMask 255.255.255.0
	
# Entregar gateway, DNS y dominio a los clientes
Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 `
	-Router 192.168.10.1 `
	-DnsServer 192.168.10.10 `
	-DnsDomain "empresa.local"
```

---
## Diagnóstico: preguntas y herramientas básicas

Cuando algo falla, no conviene cambiar configuraciones al azar. Primero identifica si el problema es de conectividad, DHCP o DNS. Estas preguntas ayudan a ordenar el diagnóstico.
### Comandos útiles en el cliente

```powershell
# Ver toda la configuración de red recibida
ipconfig /all

# Solicitar una nueva concesión DHCP
ipconfig /release
ipconfig /renew

# Consultar un nombre mediante DNS
nslookup archivos.empresa.local
Resolve-DnsName archivos.empresa.local

# Limpiar caché DNS y solicitar registro dinámico
ipconfig /flushdns
ipconfig /registerdns
```

---

## Conceptos Adicionales

- **Enrutamiento (_Routing_).** Es el proceso de reenviar paquetes de datos a través de diferentes subredes usando la Capa 3 del modelo OSI (IP). En un entorno local, los equipos necesitan conocer la IP de su Puerta de Enlace Predeterminada (Default Gateway) para enviar tráfico fuera de su propia subred.
- **NAT (_Network Address Translation_).** Traduce direcciones IP privadas no enrutables en Internet (definidas bajo RFC 1918, como 10.0.0.0/8 o 192.168.0.0/16) a una o más IP públicas globales. El DHCP entrega a los clientes su IP privada local y la puerta de enlace; cuando esos clientes quieren navegar en Internet, el router/firewall realiza NAT para darles salida.
