---
Fecha de creación: 2026-09-07T18:56:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-09-07
---
# SSH, SCP y SFTP

**SSH** permite controlar un servidor remoto mediante un canal cifrado. Sobre esa misma tecnología funcionan **SCP**, orientado a copiar archivos directamente, y **SFTP**, pensado para administrar archivos durante una sesión interactiva.

## SSH

Abre una terminal remota segura y también puede ejecutar un comando sin mantener una sesión interactiva.

## SCP

Copia archivos o directorios entre equipos con una sintaxis parecida a `cp`.

## SFTP

Ofrece órdenes para explorar, subir, descargar, renombrar y eliminar archivos remotos.

|Elemento|Valor habitual|Función|
|---|---|---|
|Servicio|`sshd`|Proceso que escucha y acepta conexiones SSH en el servidor.|
|Puerto|`TCP 22`|Puerto predeterminado para SSH y sus subsistemas.|
|Cliente|`ssh`|Programa que inicia la conexión desde el equipo administrador.|
|Servidor|`OpenSSH Server`|Implementación común del servicio SSH en Linux.|

---

## ¿Cómo Funciona SSH?

El cliente inicia una conexión TCP, verifica la identidad criptográfica del servidor, negocia el cifrado y finalmente autentica al usuario. Después, el tráfico de la terminal viaja protegido contra lectura y modificación durante el tránsito.

1. **Conexión.** El cliente contacta la dirección del servidor en el puerto configurado, normalmente el 22.
2. **Verificación del servidor.** La clave de host identifica al equipo remoto. En la primera conexión, su huella se acepta y se guarda en `~/.ssh/known_hosts`.
3. **Negociación segura.** Ambos extremos acuerdan algoritmos y crean claves temporales para cifrar la sesión.
4. **Autenticación del usuario.** El servidor valida una contraseña o, preferiblemente, una clave pública autorizada.
5. **Sesión.** El usuario recibe una terminal o solicita otra función, como SFTP o la ejecución de un comando.

> **No confundir:** la clave de host identifica al _servidor_; la clave pública del usuario identifica al _cliente o usuario autorizado_.

---

## Autenticación: Contraseña y Claves

|Método|Cómo funciona|Consideración|
|---|---|---|
|Contraseña|El usuario escribe un secreto que el servidor valida.|Es sencilla, pero queda expuesta a intentos de fuerza bruta si no se protege el servicio.|
|Clave pública|El cliente demuestra que posee la clave privada correspondiente a una clave pública autorizada.|Facilita accesos robustos y automatización sin transmitir la clave privada.|

```bash
# Crear un par de claves moderno
ssh-keygen -t ed25519 -C "ana@cliente"

# Instalar la clave pública en la cuenta remota
ssh-copy-id ana@192.168.50.20

# Conectarse usando una clave privada concreta
ssh -i ~/.ssh/id_ed25519 ana@192.168.50.20
```

---

## Comandos Básicos de SSH

La forma general es `ssh [opciones] usuario@servidor`. El servidor puede indicarse mediante una dirección IP o un nombre que DNS pueda resolver.

```bash
# Conexión básica
ssh ana@192.168.50.20

# Usar un puerto distinto al predeterminado
ssh -p 2222 ana@servidor.empresa.local

# Ejecutar una orden remota y regresar al cliente
ssh ana@192.168.50.20 "hostname && uptime"
>> web01
>> 10:45:20 up 8 days, 3:12, 2 users, load average: 0.04, 0.06, 0.01

# Mostrar detalles para diagnosticar una conexión
ssh -v ana@192.168.50.20
```

|Opción|Significado|Ejemplo breve|
|---|---|---|
|`-p`|Indica el puerto del servidor SSH.|`ssh -p 2222 usuario@host`|
|`-i`|Selecciona un archivo de identidad o clave privada.|`ssh -i llave usuario@host`|
|`-v`|Muestra información de diagnóstico.|`ssh -v usuario@host`|
|`-J`|Usa un servidor intermedio o _jump host_.|`ssh -J salto usuario@destino`|

> **Cerrar la sesión:** se puede ejecutar `exit` o presionar Ctrl + D. Esto termina la terminal remota y devuelve el control al equipo cliente.

---

## SCP: Copia Segura y Directa

**SCP** copia datos entre una ruta local y una ruta remota. La presencia de `usuario@servidor:` identifica el lado remoto; sin los dos puntos, la ruta se interpreta como local.

```bash
# Sintaxis general
scp [opciones] origen destino

# Subir un archivo local al servidor
scp reporte.pdf ana@192.168.50.20:/home/ana/documentos/

# Descargar un archivo remoto al directorio actual
scp ana@192.168.50.20:/var/log/app.log .

# Copiar un directorio completo de forma recursiva
scp -r sitio/ ana@192.168.50.20:/srv/www/

# Conectar a un puerto SSH diferente
scp -P 2222 respaldo.tar.gz ana@servidor:/backups/
```

> [!IMPORTANT] **Mayúscula importante:**
> En `scp`, el puerto se indica con `-P` mayúscula. La opción `-p` minúscula conserva fechas y permisos del archivo.

---

## SFTP: Gestión Interactiva de Archivos

**SFTP** significa _SSH File Transfer Protocol_. No es “FTP con cifrado añadido”: es un protocolo diferente que funciona como subsistema de SSH y permite mantener una sesión para realizar varias operaciones.

```bash
sftp ana@192.168.50.20
>> Connected to 192.168.50.20.

pwd                 # directorio remoto actual
>> Remote working directory: /home/ana

lpwd                # directorio local actual
ls                  # listar archivos remotos
put informe.pdf     # subir un archivo
get reporte.csv     # descargar un archivo
mkdir entregas      # crear un directorio remoto
bye                 # cerrar la sesión
```

|Remoto|Local|Acción|
|---|---|---|
|`pwd`|`lpwd`|Muestra el directorio actual.|
|`ls`|`lls`|Lista el contenido.|
|`cd`|`lcd`|Cambia de directorio.|
|`get archivo`|—|Descarga desde el servidor.|
|`put archivo`|—|Sube hacia el servidor.|

---

## SCP y SFTP: ¿Cuál Conviene?

|Criterio|SCP|SFTP|
|---|---|---|
|Modo de uso|Un comando de copia.|Sesión interactiva o transferencia automatizada.|
|Operaciones|Copiar archivos y directorios.|Listar, navegar, subir, descargar, renombrar y borrar.|
|Caso típico|Enviar rápidamente un archivo a una ruta conocida.|Administrar varios archivos sin abrir una terminal del sistema.|
|Seguridad de transporte|Utiliza SSH.|Utiliza SSH.|
|Puerto habitual|TCP 22.|TCP 22.|

> **Elección sencilla:** usa SCP cuando conoces el origen y el destino de una copia puntual; usa SFTP cuando necesitas explorar y realizar varias operaciones durante la misma conexión.

---

## Seguridad y Diagnóstico Básico

- **Menor privilegio.** Conectarse con una cuenta normal y usar `sudo` solo cuando una tarea administrativa lo requiera.
- **Verificar la huella.** Una advertencia de clave de host cambiada puede indicar una reinstalación, un cambio legítimo o un posible ataque.
- **Limitar exposición.** Aplicar firewall, permitir solo redes necesarias y revisar intentos fallidos en los registros.
- **Mantener actualizado.** Actualizar OpenSSH y el sistema reduce la exposición a vulnerabilidades conocidas.

Consultas frecuentes en el servidor:

```bash
# Estado del servicio en Ubuntu/Debian
sudo systemctl status ssh

# Confirmar que existe un proceso escuchando en TCP 22
sudo ss -tlnp | grep ':22'

# Consultar eventos recientes del servicio
sudo journalctl -u ssh --since today
```

|Mensaje o síntoma|Causa probable|Qué revisar|
|---|---|---|
|`Connection timed out`|No se alcanza el servidor o un firewall descarta el tráfico.|IP, rutas, puerto y reglas de red.|
|`Connection refused`|El equipo responde, pero nada acepta conexiones en ese puerto.|Estado de `sshd` y puerto configurado.|
|`Permission denied`|Falló la autenticación o la cuenta no permite ese acceso.|Usuario, contraseña, claves y permisos de `~/.ssh`.|
|Clave de host cambiada|El servidor presenta una identidad distinta a la registrada.|Confirmar el cambio con el administrador antes de aceptar.|

> [!WARNING] No se debe ignorar automáticamente una alerta de identidad.
> El cifrado protege la conexión solo si también se confirma que el servidor es el correcto.
