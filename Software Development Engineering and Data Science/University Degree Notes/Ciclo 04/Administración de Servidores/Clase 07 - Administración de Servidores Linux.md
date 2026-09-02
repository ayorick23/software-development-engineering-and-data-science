---
Fecha de creación: 2026-08-24T18:56:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-08-24
---
# Administración de Servidores Linux

**Linux** es una familia de sistemas operativos muy usada en servidores por su estabilidad, seguridad, flexibilidad y porque permite administrar casi todo desde una terminal.

Una **distribución** combina el núcleo Linux con programas, instalador y repositorios. Ubuntu Server es una opción amigable para aprender; Debian, Rocky Linux y Red Hat son también comunes en entornos de trabajo.

|Elemento|Qué hace|Ejemplo|
|---|---|---|
|**Kernel**|Gestiona hardware, memoria y procesos.|El núcleo Linux|
|**Shell**|Interpreta las órdenes escritas por el administrador.|`bash`|
|**Paquete**|Programa instalable con sus archivos y dependencias.|`nginx`, `git`|
|**Servicio**|Programa que se ejecuta en segundo plano.|SSH o servidor web|

### Ubuntu Server vs. Windows Server

|Aspecto|Ubuntu Server|Windows Server|
|---|---|---|
|**Administración habitual**|Terminal, SSH y archivos de configuración.|Interfaz gráfica, Server Manager y PowerShell.|
|**Software**|Paquetes y repositorios con `apt`.|Roles, características e instaladores.|
|**Identidades y permisos**|Usuarios, grupos y permisos `rwx`.|Active Directory, grupos y permisos NTFS.|

---

## Instalación y Primer Acceso

Para esta clase se utilizará **Ubuntu Server**. Descarga la imagen ISO desde la [página oficial de Ubuntu Server](https://ubuntu.com/download/server) . Para aprender sin modificar el equipo principal se puede crear una máquina virtual. En un servidor real, revisa siempre el disco que se va a particionar: instalar puede borrar datos.

1. **Elegir idioma, teclado y red.** Si es posible, usa una IP estable para un servidor.
2. **Crear el usuario administrador.** Elige una contraseña robusta y no compartas esa cuenta.
3. **Configurar almacenamiento.** El sistema usa particiones; Linux las monta dentro de un único árbol que inicia en `/`.
4. **Instalar OpenSSH.** Permite administrar el equipo desde otra computadora de forma cifrada.
5. **Actualizar al entrar.** Las actualizaciones corrigen errores y vulnerabilidades.

Actualizar Ubuntu/Debian:

```bash
# Actualiza la lista de paquetes y luego instala actualizaciones
sudo apt update
sudo apt upgrade

# Instala el servicio para acceso remoto seguro
sudo apt install openssh-server
```


---

## Línea de Comandos

La terminal recibe una orden, la ejecuta y muestra el resultado. Su forma general es `comando [opciones] [objetivo]`. Por ejemplo, `ls -l /home` lista con detalle el contenido de `/home`.

- **Usuario normal.** Trabaja sin privilegios administrativos. Es la cuenta recomendada para las tareas cotidianas.
- **root.** Puede cambiar cualquier parte del sistema. Un error con root puede afectar todo el servidor.
- **sudo.** Ejecuta una orden concreta con privilegios, dejando registro y evitando iniciar sesión como root.

Orientarse y consultar ayuda:

```bash
pwd          # muestra la carpeta actual
ls -la       # lista también archivos ocultos y permisos
cd /etc      # cambia de directorio
man ls       # abre el manual de ls (q para salir)
ip address   # muestra interfaces y direcciones IP
```

---

## Sistema de Archivos

En Linux no se usan letras como `C:`. Todo parte de la raíz `/`. Los directorios organizan los archivos y también separan las responsabilidades del sistema.

|Ruta|Propósito|
|---|---|
|`/home`|Carpetas personales de los usuarios.|
|`/etc`|Configuración del sistema y de servicios.|
|`/var`|Datos que cambian: registros, cachés y contenido web.|
|`/var/log`|Registros para diagnosticar eventos y errores.|
|`/tmp`|Archivos temporales; no debe usarse para datos importantes.|

Crear y revisar archivos:

```bash
mkdir -p ~/proyecto/logs     # crea una ruta, incluso si falta una carpeta
touch ~/proyecto/notas.txt   # crea un archivo vacío
cp notas.txt respaldo.txt    # copia un archivo
cat /etc/hostname            # muestra el nombre del equipo
grep "error" /var/log/syslog # busca una palabra en un registro
```

---

## Usuarios, Grupos y Sudo

Un **usuario** identifica a una persona o proceso. Un **grupo** reúne cuentas con una necesidad común. Asignar permisos al grupo es más ordenado que configurarlos persona por persona.

Los datos básicos de las cuentas se consultan en `/etc/passwd`; la información protegida de contraseñas está en `/etc/shadow`, accesible solo por administración. No se editan manualmente durante una práctica básica.

 Crear una cuenta de soporte:

```bash
sudo adduser ana             # crea usuario, carpeta /home/ana y contraseña
sudo groupadd soporte        # crea un grupo
sudo usermod -aG soporte ana # agrega ana al grupo sin quitar los demás
id ana                       # verifica UID, grupo principal y grupos extra
sudo usermod -aG sudo ana    # da privilegios: úsalo solo si corresponde
```

>[!IMPORTANT] **Principio de mínimo privilegio:** cada cuenta recibe solo el acceso necesario para su trabajo. Una cuenta que consulta registros no necesita administrar usuarios.

---

## Permisos

Todo archivo y directorio tiene propietario, grupo y permisos para tres categorías: **u** (usuario propietario), **g** (grupo) y **o** (otros). Las letras **r**, **w** y **x** significan leer, escribir y ejecutar.

- **Propietario.** Normalmente es quien creó el archivo. Puede modificarlo según sus permisos y, con privilegios, cambiar propietario o grupo mediante `chown`.
- **Grupo.** Sirve para compartir acceso entre varias cuentas. Una carpeta de soporte puede pertenecer al grupo `soporte`.
- **Otros.** Incluye a cualquier cuenta que no sea el propietario ni pertenezca al grupo. En datos internos suele tener permiso `---`.

| En archivo                 | En directorio                           | Valor octal |
| -------------------------- | --------------------------------------- | ----------- |
| **r**: leer contenido      | **r**: listar nombres                   | 4           |
| **w**: modificar contenido | **w**: crear o borrar elementos         | 2           |
| **x**: ejecutar            | **x**: entrar o atravesar el directorio | 1           |

Al ejecutar `ls -l`, los primeros diez caracteres describen el tipo y los permisos. En `-rw-r-----`, el guion inicial indica un archivo; los siguientes tres caracteres (`rw-`) son del propietario, los tres siguientes (`r--`) son del grupo y los últimos tres (`---`) son de otros.

```bash
ls -l informe.txt
-rw-r----- 1 ana soporte 1240 ago 20 10:15 informe.txt
# ana puede leer y modificar; soporte solo puede leer; otros no acceden
stat informe.txt # muestra propietario, grupo, fechas y permisos con más detalle
```

### Permisos Numéricos

La forma numérica de `chmod` usa **4** para lectura, **2** para escritura y **1** para ejecución. Se suman los permisos para propietario, grupo y otros, en ese orden.

|Modo|Significado|Uso recomendado|
|---|---|---|
|`600`|`rw-------`|Archivo privado, por ejemplo una clave o configuración sensible.|
|`640`|`rw-r-----`|Propietario modifica; su grupo solo consulta.|
|`644`|`rw-r--r--`|Archivo público de lectura, como una página HTML estática.|
|`700`|`rwx------`|Carpeta o script que solo usa su propietario.|
|`750`|`rwxr-x---`|Carpeta privada con acceso de consulta para el grupo.|

Cambiar permisos y propiedad:

```bash
chmod 640 informe.txt       # propietario: rw- | grupo: r-- | otros: ---
chmod u+x respaldo.sh       # agrega ejecución solamente al propietario
chmod g-w informe.txt       # quita escritura al grupo
sudo chown ana:soporte informe.txt # cambia propietario y grupo
```

Compartir una carpeta de forma controlada:

```bash
sudo mkdir /srv/proyecto
sudo chown root:soporte /srv/proyecto # propietario root; grupo soporte
sudo chmod 2770 /srv/proyecto         # rwx para propietario y grupo; nada para otros
ls -ld /srv/proyecto
drwxrws--- root soporte /srv/proyecto

# el 2 activa setgid: los nuevos archivos heredan el grupo soporte
```
