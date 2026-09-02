---
Fecha de creación: 2026-08-31T19:01:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-08-31
---
# Configuración de Red en Linux

La configuración de red define **cómo se identifica** un servidor, **por dónde envía datos** y **cómo encuentra otros equipos**. En Ubuntu Server normalmente queda guardada de forma persistente con **Netplan**.

- **Interfaz.** Adaptador físico o virtual. Sus nombres pueden ser `enp0s3`, `ens18` o `eth0`.
- **Dirección IP.** Identifica la interfaz dentro de una red, por ejemplo `192.168.10.20`.
- **Puerta de enlace.** Router al que se envía el tráfico destinado a otras redes, incluido Internet.
- **DNS.** Traduce nombres como `ubuntu.com` a direcciones IP.

|Necesidad|Elemento|Ejemplo|
|---|---|---|
|Responder dentro de la LAN|IP y prefijo|`192.168.10.20/24` pertenece a 192.168.10.0/24.|
|Llegar a Internet|Ruta por defecto|El tráfico sale hacia `192.168.10.1`.|
|Abrir un sitio por nombre|Servidor DNS|Consulta al DNS institucional o a `1.1.1.1`.|

---

## Interfaces y Estado Actual

Antes de cambiar algo, conviene observar. `ip` es la herramienta moderna para consultar interfaces, direcciones y rutas. El nombre de interfaz **no es universal**: se debe copiar el que muestra el servidor.

```bash
admin@ubuntu:~$ ip -br address  # resumen de interfaces e IP
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             192.168.10.20/24

admin@ubuntu:~$ ip route        # tabla de rutas
default via 192.168.10.1 dev enp0s3 proto static
192.168.10.0/24 dev enp0s3 proto kernel scope link src 192.168.10.20

admin@ubuntu:~$ ip link show enp0s3 # estado de una interfaz
```

|Estado|Qué suele indicar|Qué comprobar|
|---|---|---|
|`UP`|La interfaz está habilitada y tiene enlace lógico.|IP asignada, ruta y conectividad.|
|`DOWN`|Está deshabilitada o no hay enlace.|Cable, adaptador virtual, switch o configuración.|
|Solo `127.0.0.1`|Solo existe la interfaz local `lo`.|La interfaz de red no tiene una IPv4 útil.|

---

## IPv4: Dirección, Prefijo y Asignación

Una IPv4 identifica un host. El sufijo `/24` es el **prefijo CIDR**: indica que los primeros 24 bits corresponden a la red. En `192.168.10.20/24`, la red es `192.168.10.0`.

|Prefijo|Máscara decimal|Uso o alcance típico|
|---|---|---|
|`/24`|`255.255.255.0`|Red pequeña, como 192.168.10.0/24.|
|`/16`|`255.255.0.0`|Red más grande, como 10.20.0.0/16.|
|`/30`|`255.255.255.252`|Enlace punto a punto entre equipos de red.|

- **DHCP.** Entrega automáticamente IP, prefijo, ruta y, con frecuencia, DNS. Es útil para clientes.
- **IP estática.** No debería cambiar. Es preferible para servicios web, DNS, archivos o SSH.

---

## Netplan en Ubuntu Server

Netplan lee archivos **YAML** de `/etc/netplan/` y entrega la configuración al componente de red, habitualmente `systemd-networkd`. La sangría forma parte del formato: usa espacios, nunca tabuladores.

 Localizar, validar y aplicar

```bash
admin@ubuntu:~$ ls -l /etc/netplan/       # archivos como 00-installer-config.yaml
admin@ubuntu:~$ sudo netplan generate     # valida y genera la configuración
admin@ubuntu:~$ sudo netplan try          # aplica temporalmente y pide confirmación
admin@ubuntu:~$ sudo netplan apply        # aplica de inmediato
```

### Ejemplo A: obtener la configuración por DHCP

```bash
# Archivo: /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

### Ejemplo B: servidor con IP estática

```bash
# Archivo: /etc/netplan/01-servidor-estatico.yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 192.168.10.20/24
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses: [192.168.10.2, 1.1.1.1]
        search: [empresa.local]
```

El servidor usa `192.168.10.20`; sale por el router `192.168.10.1`; consulta primero el DNS interno y usa `empresa.local` como sufijo de búsqueda.

---

## Rutas y Resolución DNS

La **tabla de rutas** decide el siguiente salto de cada paquete. Una ruta específica gana a una general; la ruta `default` se usa cuando no hay una más específica.

 Ruta a una red remota en Netplan:

```bash
# Fragmento dentro de enp0s3: 172.16.30.0 se alcanza por otro router
routes:
  - to: default
    via: 192.168.10.1
  - to: 172.16.30.0/24
    via: 192.168.10.254
```

Cómo comprobar DNS:

```bash
admin@ubuntu:~$ resolvectl status             # DNS activo por interfaz
admin@ubuntu:~$ resolvectl query ubuntu.com   # consulta un nombre
admin@ubuntu:~$ getent hosts ubuntu.com       # usa el mecanismo normal del sistema
admin@ubuntu:~$ cat /etc/resolv.conf          # referencia generada; no suele editarse a mano
```

---

## Diagnóstico Ordenado de Conectividad

Diagnosticar por capas evita cambios al azar. Se empieza cerca del servidor y se avanza hacia el destino.

1. **Verificar la interfaz.** ¿Está `UP` y tiene la IP esperada? Consultar `ip -br address`.
2. **Probar la red local.** Comprobar el router con `ping -c 4 192.168.10.1`.
3. **Revisar la salida.** Consultar `ip route` y buscar `default via`.
4. **Probar una IP externa.** Usar `ping -c 4 1.1.1.1`; esto excluye temporalmente DNS.
5. **Probar un nombre.** Usar `resolvectl query ubuntu.com` o `getent hosts ubuntu.com`.

 Herramientas y lo que responden:

```bash
admin@ubuntu:~$ ping -c 4 192.168.10.1   # ¿alcanzo la puerta de enlace?
admin@ubuntu:~$ ping -c 4 1.1.1.1        # ¿hay salida por IP?
admin@ubuntu:~$ tracepath 1.1.1.1        # ¿por qué saltos viaja el tráfico?
admin@ubuntu:~$ ss -tulpn                # ¿qué puertos escucha el servidor?
admin@ubuntu:~$ sudo journalctl -u systemd-networkd # eventos de red
```

---

## Ejemplo de Configuración de una IP Estática

>**Enunciado:** el servidor `web01` está conectado a la red `192.168.50.0/24`. Debe usar la IP fija `192.168.50.20`, salir por el router `192.168.50.1` y consultar los DNS `192.168.50.10` y `1.1.1.1`. La interfaz detectada es `enp0s3`.

1. **Confirmar la interfaz real.** El ejemplo usa `enp0s3`, pero en cada servidor debe verificarse el nombre antes de escribir el archivo.
2. **Ubicar la configuración actual.** Se revisa el contenido de `/etc/netplan/` para editar el archivo existente o crear uno con un nombre claro.
3. **Declarar IP, ruta y DNS.** Se escribe YAML respetando la sangría y usando los valores del enunciado.
4. **Validar antes de aplicar.** `netplan generate` detecta problemas de formato; en una sesión SSH se usa `netplan try`.
5. **Comprobar el resultado.** Se confirma la IP, la ruta por defecto, el alcance del router y la resolución DNS.

 1 y 2. Identificar la interfaz y el archivo Netplan:

```bash
admin@web01:~$ ip -br address
>> enp0s3           UP             192.168.50.101/24

admin@web01:~$ ls -l /etc/netplan/
>> -rw------- 1 root root 180 00-installer-config.yaml

admin@web01:~$ sudo nano /etc/netplan/00-installer-config.yaml
```

3. Contenido del archivo de configuración:

```bash
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.50.20/24]
      routes:
        - to: default
          via: 192.168.50.1
      nameservers:
        addresses: [192.168.50.10, 1.1.1.1]
```

 4 y 5. Validar, aplicar y verificar:

```bash
admin@web01:~$ sudo netplan generate          # valida la sintaxis

admin@web01:~$ sudo netplan try               # confirmar solo si conserva conectividad

admin@web01:~$ ip -br address show enp0s3
enp0s3           UP             192.168.50.20/24

admin@web01:~$ ip route | grep default
default via 192.168.50.1 dev enp0s3

admin@web01:~$ ping -c 4 192.168.50.1       # comprueba el router

admin@web01:~$ resolvectl query ubuntu.com  # comprueba DNS
```
