---
Fecha de creación: 2026-07-13T18:21:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-07-13
---
# Active Directory

**Active Directory** es el servicio de directorio de Microsoft: una base de datos centralizada y jerárquica que almacena información sobre los **objetos** de una red (usuarios, grupos, computadoras, impresoras, servidores) junto con las reglas de quién puede acceder a qué. Corre como un rol dentro de Windows Server, bajo el nombre técnico **AD DS (Active Directory Domain Services)**.

## El Problema que Resuelve

Por ejemplo, si se tiene una empresa con 300 empleados y 300 computadoras. Sin AD, cada computadora tendría su propia lista de usuarios y contraseñas independiente. Si, por ejemplo, Ana necesita acceder a 5 computadoras distintas, necesitaría 5 cuentas separadas, y el administrador tendría que crear, modificar y eliminar cuentas una por una, en cada máquina.

- **Sin Active Directory**
	Cada equipo administra sus propios usuarios locales. No hay una fuente única de verdad. Cambiar una contraseña significa cambiarla en cada máquina donde el usuario tiene cuenta. Imposible de escalar más allá de unos pocos equipos.
- **Con Active Directory**
	Un usuario se crea **una sola vez** en el directorio y puede iniciar sesión en cualquier equipo unido al dominio, con las mismas credenciales. Los permisos, políticas y recursos se administran de forma centralizada.

## Los 4 Pilares de Active Directory

1. **Autenticación.** Verifica que un usuario es quien dice ser (usuario + contraseña, o tarjeta inteligente) mediante el protocolo Kerberos.
2. **Autorización.** Define qué puede hacer un usuario ya autenticado: a qué carpetas, impresoras o aplicaciones tiene acceso.
3. **Directorio centralizado.** Una base de datos jerárquica (NTDS.dit) con todos los objetos de la red: usuarios, grupos, equipos, políticas.
4. **Políticas de grupo (GPO).** Reglas que se aplican automáticamente a usuarios y computadoras: fondo de pantalla, restricciones, software.

> [!NOTE] **Analogía**
> Active Directory es como el registro civil de un país. Cada ciudadano (usuario) tiene un único documento de identidad válido en todo el territorio (dominio), y ese documento determina en qué lugares puede entrar y qué puede hacer.

---

## Controladores de Dominio (_Domain Controllers_)

Un **Controlador de Dominio (DC)** es un servidor Windows Server que tiene instalado el rol AD DS y aloja una copia de la base de datos del directorio. Es la máquina que efectivamente "presta" el servicio de Active Directory a toda la red.

### ¿Qué hace un DC?

- Autentica a los usuarios cuando inician sesión (verifica usuario y contraseña).
- Aloja y replica la base de datos de objetos del dominio (NTDS.dit).
- Aplica las políticas de grupo (GPO) a usuarios y equipos.
- Normalmente también actúa como servidor DNS del dominio.

### ¿Por qué se recomienda al menos dos DC?

Un solo controlador de dominio es un **punto único de falla**: si se apaga o se corrompe, nadie puede iniciar sesión ni acceder a recursos del dominio. Por eso, en un entorno real se instalan al menos **dos controladores de dominio** que se replican entre sí.

|Escenario|Con 1 DC|Con 2+ DC|
|---|---|---|
|El DC se apaga o falla|Nadie puede iniciar sesión|El segundo DC sigue autenticando usuarios|
|Mantenimiento / reinicio|Interrumpe todo el dominio|Sin interrupción, se mantiene otro DC activo|
|Carga de autenticación|Todo recae en una máquina|Se distribuye entre los DCs|

---

## DNS: la columna vertebral de Active Directory

**DNS** traduce nombres a direcciones IP. Active Directory depende completamente de DNS, tanto que **no puede funcionar sin él**. De hecho, al instalar el primer controlador de dominio, si no existe un servidor DNS configurado, el propio asistente instala uno automáticamente.

### ¿Por qué AD necesita DNS?

- **Localización de servicios:** cuando un equipo cliente quiere iniciar sesión, pregunta a DNS "¿dónde está el controlador de dominio?". AD registra esta información como **registros SRV** (Service Records) especiales dentro de DNS.
- **Nombre de dominio:** el nombre del dominio de Active Directory (ejemplo: `empresa.local`) es, en realidad, una zona DNS.
- **Replicación entre DCs:** los controladores de dominio se encuentran entre sí consultando DNS.

> [!IMPORTANT] **Regla práctica:**
> El DC apunta su propia configuración de DNS hacia sí mismo (o hacia otro DC con DNS). Si un cliente tiene configurado un DNS externo (como el de su router doméstico) en lugar del DNS del dominio, **no podrá encontrar el controlador de dominio** y fallará al iniciar sesión o unirse al dominio.

**Ejemplo de registros SRV creados por AD:**

| Registro                             | Propósito                                               |
| ------------------------------------ | ------------------------------------------------------- |
| `_ldap._tcp.dc._msdcs.empresa.local` | Localiza los controladores de dominio disponibles       |
| `_kerberos._tcp.empresa.local`       | Localiza el servicio de autenticación Kerberos          |
| `_gc._tcp.empresa.local`             | Localiza el Catálogo Global (Global Catalog) del bosque |

---

## Dominios, Árboles y Bosques

Active Directory se organiza en una jerarquía de tres niveles. Entender esta estructura es clave antes de crear usuarios u objetos, porque define los límites de seguridad y administración.

### A. Dominio (Domain)

La unidad básica de Active Directory: un límite administrativo y de seguridad que agrupa usuarios, computadoras y recursos que comparten la misma base de datos de directorio.  
  
**Ejemplo:** `empresa.local`. Todos los objetos dentro de este dominio comparten las mismas políticas de contraseñas por defecto y confían en los mismos DCs.

### B. Árbol (Tree)

Un conjunto de uno o más dominios que comparten un **espacio de nombres contiguo** (mismo sufijo de dominio) y una relación de confianza automática entre ellos.  
  
**Ejemplo:** `empresa.local` como raíz, con `ventas.empresa.local` y `soporte.empresa.local` como dominios hijos.

### C. Bosque (Forest)

El contenedor más grande en Active Directory: agrupa uno o más árboles, que **no necesitan compartir el mismo nombre**, pero sí comparten el esquema, el catálogo global y una relación de confianza entre todos ellos.  
  
**Ejemplo:** una empresa que compró otra compañía puede tener `empresa.local` y `empresaadquirida.com` en el mismo bosque.

### Jerarquía visual

1. **Bosque (nivel más amplio):** comparte esquema y catálogo global entre todos los árboles.
2. **Árbol**: agrupa dominios que comparten espacio de nombres (ej. `empresa.local` y sus subdominios).
3. **Dominio**: unidad administrativa con su propia base de datos y políticas.
4. **Unidad Organizativa (OU)**: subdivisión dentro de un dominio (siguiente sección).
5. **Objetos**: usuarios, grupos, equipos — lo que realmente se administra día a día.

---

## Unidades Organizativas (OU)

Una **Unidad Organizativa (Organizational Unit, OU)** es un contenedor dentro de un dominio que permite agrupar usuarios, equipos, grupos u otras OUs de forma lógica, reflejando la estructura real de la organización (departamentos, sucursales, roles).

### ¿Para qué sirven las OU?

- **Organización lógica:** agrupar objetos según departamento, ubicación o función (ej. _Ventas_, _Soporte_, _Gerencia_).
- **Delegación de administración:** se puede asignar a un usuario permisos para administrar solo una OU específica (por ejemplo, que el jefe de Soporte pueda resetear contraseñas de su propio equipo, sin acceso al resto del dominio).
- **Aplicación de políticas de grupo (GPO):** las GPOs se vinculan a nivel de OU, lo que permite aplicar reglas distintas a distintos grupos (ej. restringir instalación de software solo en la OU de Practicantes).

---

## Usuarios y Objetos de Active Directory

Todo lo que se administra dentro de Active Directory es, técnicamente, un **objeto**: una entrada en la base de datos del directorio con un conjunto de atributos (nombre, contraseña, dirección de correo, descripción, etc.).

### Tipos de objetos principales

#### Usuario

Representa a una persona. Tiene credenciales (usuario/contraseña) y puede iniciar sesión en el dominio.

#### Equipo (Computer)

Representa una computadora unida al dominio. También tiene una cuenta propia con contraseña, gestionada automáticamente.

#### Grupo

Conjunto de usuarios o equipos usado para asignar permisos en bloque, en lugar de uno por uno.

#### Unidad Organizativa

Contenedor administrativo (visto en la sección anterior). También es, técnicamente, un objeto.

#### Impresora publicada

Permite que los usuarios busquen y se conecten a impresoras compartidas a través del directorio.

#### Recurso compartido

Referencias a carpetas compartidas en la red, publicadas para que sean fáciles de encontrar.
