---
Fecha de creación: 2026-08-10T19:20:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-08-10
---
# Servidor de Archivos e IIS

En esta infraestructura, **SRV01** es un Windows Server del dominio `empresa.local`, con IP `192.168.1.10`. Ya tiene Active Directory, DNS y GPO. Ahora también entrega archivos y una intranet.

|Lo que usa el cliente|Servicio que responde|Ejemplo|
|---|---|---|
|Una carpeta de red|File Server mediante SMB|`\\SRV01\RRHH`|
|Un sitio interno|IIS mediante HTTP o HTTPS|`intranet.empresa.local`|

>**La idea central:** Active Directory identifica a los usuarios; DNS encuentra al servidor; NTFS autoriza los archivos; SMB los transporta por la red; IIS publica el contenido web.

---

## Almacenamiento

Es normal separar el sistema y los datos: `C:` contiene Windows y programas; `D:` guarda carpetas de departamentos como `D:\Datos\RRHH`, `Ventas` y `TI`. Así se organizan mejor el espacio y las copias de seguridad.

Una letra como `D:` representa un **volumen**: una parte de un disco que Windows puede usar. Puede provenir de un segundo disco virtual o de espacio separado en un disco existente.

|Concepto|Explicación breve|
|---|---|
|**MBR**|Tabla de particiones antigua; se limita aproximadamente a 2 TB y cuatro particiones primarias.|
|**GPT**|Tabla moderna, preparada para discos grandes y habitual con UEFI. Es la opción normal para un nuevo disco de datos.|
|**FAT32**|Muy compatible, pero no admite archivos mayores de 4 GB ni permisos NTFS. Es común en memorias USB, no en un servidor de archivos.|
|**NTFS**|Sistema de archivos de Windows con permisos, herencia, cuotas, cifrado y archivos grandes. Es el adecuado para estos datos.|
|**ReFS**|Sistema orientado a resiliencia e integridad de datos. Tiene usos especializados, pero NTFS sigue siendo habitual para estas carpetas compartidas.|

---

## Active Directory y NTFS

Active Directory confirma **quién es** el usuario. NTFS decide **qué puede hacer** esa identidad dentro de una carpeta. No son lo mismo, pero trabajan juntos.

- **Grupos de AD.** En vez de dar acceso a cada persona por separado, se usa un grupo como `GG_RRHH`. Los usuarios entran al grupo y el grupo recibe el permiso.
- **ACL.** La lista de control de acceso de una carpeta guarda qué usuarios o grupos tienen permiso. Es la lista de autorizados de NTFS.
- **Herencia.** Una subcarpeta suele recibir permisos de su carpeta padre. Al deshabilitar herencia se pueden conservar como permisos explícitos y adaptar la carpeta a un departamento.

|Permiso NTFS|Qué permite|Uso típico|
|---|---|---|
|**Lectura y ejecución**|Ver, abrir y ejecutar; no cambiar.|Documentos de consulta.|
|**Escritura**|Crear y escribir contenido, con capacidades más limitadas que Modificar.|Carga de archivos.|
|**Modificar**|Leer, crear, cambiar, renombrar y eliminar.|`GG_RRHH → RRHH`.|
|**Control total**|Además de modificar, cambia permisos y toma posesión.|Administradores y SYSTEM.|

Los **permisos especiales** dan un control más fino: listar una carpeta, crear archivos pero no carpetas, eliminar subcarpetas o cambiar permisos. Una denegación explícita suele ganar sobre una concesión; por eso se usa solo si realmente hace falta.

---

## SMB

**SMB** (_Server Message Block_) es el protocolo con el que Windows comparte archivos e impresoras. Normalmente usa el puerto TCP **445**.

Una carpeta local, como `D:\Datos\RRHH`, puede publicarse como el recurso `RRHH`. Desde un cliente se abre con una ruta **UNC**: `\\SRV01\RRHH`. El nombre del servidor puede resolverse mediante DNS.

- **Permiso de compartición.** Controla el acceso al recurso SMB. En un laboratorio puede ser amplio para dejar el detalle a NTFS; en producción también debe diseñarse con cuidado.
- **Permiso NTFS.** Controla las operaciones sobre archivos y carpetas. Se aplica tanto localmente como al entrar por SMB.
- **Acceso efectivo.** Por red se aplican ambas capas. El resultado nunca puede ser más permisivo que la restricción que imponga una de ellas.

>**Ejemplo:** si el recurso compartido deja leer, pero NTFS da Modificar, el usuario por red quedará en lectura. Si el recurso es amplio y NTFS da Modificar a `GG_RRHH`, solo ese grupo podrá trabajar en RRHH.

---

## Unidades de red y GPO

Una **unidad de red** asigna una letra local a una ruta UNC. Por ejemplo, el cliente puede mostrar `RRHH (R:)`, aunque los archivos siguen en `\\SRV01\RRHH`.

Las **GPO** permiten aplicar ese mapeo a muchos usuarios sin configurar cada equipo. El criterio puede ser su pertenencia a un grupo: quien pertenece a `GG_RRHH` recibe `R:`; quien pertenece a Ventas no. La GPO facilita el acceso, pero no sustituye los permisos NTFS.

>[!NOTE] **Relación completa:** usuario de AD → grupo de AD → GPO que muestra la unidad → SMB que conecta al recurso → NTFS que autoriza cada acción.

---

## IIS: publicar una intranet

**IIS** (_Internet Information Services_) es el servicio web de Windows Server. Entrega archivos HTML, imágenes y aplicaciones a los navegadores.

- **Sitio web.**
	Reúne una carpeta física, un binding y una configuración. La carpeta puede ser `C:\WebSites\Intranet`.
- **Contenido estático.**
	Archivos como `index.html`, CSS e imágenes. IIS los devuelve directamente al navegador.
- **Binding.**
	Regla que indica a qué IP, puerto y hostname debe responder un sitio.
- **Application Pool.**
	Aísla aplicaciones web y define la identidad con que se ejecutan. Es relevante cuando el sitio ejecuta código dinámico.

`localhost` y `127.0.0.1` representan al propio servidor; sirven para comprobar que IIS responde localmente. La IP comprueba conectividad. El hostname es el nombre cómodo que usarán los clientes.

---

## HTTP, HTTPS, DNS y hostnames

**DNS** relaciona `intranet.empresa.local` con `192.168.1.10` mediante un registro **A**. Solo entonces el navegador sabe a qué servidor conectarse.

### 1. DNS

El cliente pregunta por `intranet.empresa.local` y obtiene la IP de SRV01.

### 2. Solicitud web

Con HTTP, normalmente por TCP 80, el navegador envía una solicitud como `GET / HTTP/1.1` y el encabezado `Host`.

### 3. IIS responde

IIS usa IP, puerto y hostname para elegir el sitio y devuelve una respuesta: `200 OK`, `404 Not Found` o `403 Forbidden`.

**HTTPS** usa normalmente TCP 443. Antes de enviar contenido, TLS cifra la comunicación y el navegador valida un certificado. El nombre del certificado, el registro DNS y el hostname del binding deben coincidir. Con SNI, varios sitios HTTPS pueden compartir IP y puerto porque IIS distingue el hostname solicitado.
