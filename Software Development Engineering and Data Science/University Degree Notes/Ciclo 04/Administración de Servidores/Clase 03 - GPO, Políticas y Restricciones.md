---
Fecha de creación: 2026-07-20T18:21:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-07-20
---
# GPO, Políticas y Restricciones

Una **política** expresa la regla de la organización; la **configuración** define el valor técnico que Windows debe aplicar; y la **GPO (Group Policy Object)** reúne esas configuraciones para distribuirlas mediante un vínculo de Active Directory.

| Concepto                            | Pregunta que responde                                 | Ejemplo                                                 |
| ----------------------------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| **Regla organizacional (Política)** | ¿Qué exige la empresa a nivel de negocio?             | Las contraseñas deben ser robustas para evitar hackeos. |
| **Configuración técnica**           | ¿Qué valor específico aplica Windows en el sistema?   | Longitud mínima: 12 caracteres y requerir complejidad.  |
| **GPO (El contenedor)**             | ¿Cómo agrupamos y distribuimos estas configuraciones? | Un objeto GPO llamado `DOM - Seguridad de Cuentas`.     |
| **Vínculo (Link)**                  | ¿A qué nivel de la estructura se va a aplicar?        | Vinculado a la OU llamada `Usuarios_Finanzas`.          |
| **Objeto objetivo**                 | ¿Quién termina procesando y sufriendo la regla?       | El usuario o computadora dentro de esa OU.              |

### ¿Dónde se pueden vincular las GPO? (La jerarquía del alcance)

Las GPO no se asignan directamente a una persona o computadora. Se **vinculan** a contenedores de Active Directory en cuatro niveles:

- **Directiva Local (L):** Cada equipo con Windows tiene una directiva local en su propio disco duro. Se aplica solo a ese equipo y es la base de todo. Si el equipo se une a un dominio, las políticas del dominio sobreescriben a las locales.
- **Sitio (S):** Representa la ubicación física de red (por ejemplo, _"Oficina Monterrey"_ o _"Sucursal CDMX"_). Es ideal para políticas basadas en ancho de banda.
- **Dominio (D):** Se aplica a los usuarios y computadoras del dominio. Es apropiado para reglas globales, como la política de contraseñas.
- **Unidad Organizativa (OU):** El nivel más específico para dirigir GPO a departamentos, usuarios o equipos.

> [!IMPORTANT] Crear vs. vincular
> Una GPO sin vínculo existe, pero no se aplica. Un mismo objeto GPO puede vincularse a varias OU para reutilizar configuraciones.

---

## Anatomía interna de una GPO: ¿Cómo se divide?

Cualquier GPO que edites con la consola de Windows siempre estará dividida en dos grandes mitades: **Configuración del Equipo (Computer)** y **Configuración del Usuario (User)**. Es fundamental entender esta separación porque define _cuándo_ y _a quién_ se aplica la regla.

### Configuración del equipo (_Computer Configuration_)

**Se aplica a la máquina física**, sin importar quién inicie sesión. Windows procesa esta parte **al encender la computadora**, antes de que veas la pantalla para poner tu usuario y contraseña.

_Ejemplo:_ Activar el Firewall de Windows, definir que el equipo se apague a las 10 PM, instalar un antivirus o bloquear los puertos USB.

### Configuración del usuario (_User Configuration_)

**Sigue al usuario humano** a donde quiera que vaya. Windows procesa esta parte **durante el inicio de sesión** de la persona (cuando se dibuja su escritorio).

_Ejemplo:_ Establecer una foto de fondo de pantalla corporativa, quitar el acceso al Panel de Control del usuario, o mapear una carpeta compartida que le pertenece.

- **Arranque.** Se leen las políticas del equipo.
- **Inicio de sesión.** Se leen las políticas del usuario.
- **Actualización.** Las directivas se renuevan periódicamente.

### Directivas (Policies) vs. Preferencias (Preferences)

Dentro de ambas configuraciones, encontrarás dos carpetas principales: Directivas y Preferencias. Esta distinción es crucial para el control del entorno:

|Característica|Directivas (Policies)|Preferencias (Preferences)|
|---|---|---|
|**Propósito**|Imponer una configuración estricta y obligatoria (modo "dictador").|Establecer un valor inicial que facilita el trabajo, pero con flexibilidad.|
|**Control del usuario**|La opción en Windows se bloquea en gris; el usuario **no puede cambiarla**.|El usuario **puede cambiarla** si lo desea (ej. cambiar su fondo de pantalla temporalmente).|
|**Si la GPO deja de aplicarse**|Windows revierte el cambio y limpia el registro del sistema (vuelve al estado original).|El cambio persiste en el equipo (se le conoce como efecto "tatuaje" o persistencia), a menos que configures explícitamente lo contrario.|
|**Ejemplo práctico**|Bloquear el acceso al Administrador de Tareas para estudiantes.|Crear un acceso directo al sitio web de la escuela en el escritorio de todos.|

---

## Alcance, orden, herencia y precedencia

Cuando varias GPO contienen valores contradictorios, Windows las procesa en el orden **LSDOU** (Local, Site, Domain, Organizational Unit). En general, la política procesada más tarde y más específica gana.

1. **1. Local (L):** Se aplica primero. Tiene las configuraciones base de la máquina individual. Es la menos prioritaria.
2. **2. Sitio (S):** Se aplica segundo. Afecta a todos los equipos en la ubicación física de red.
3. **3. Dominio (D):** Se aplica tercero. Afecta a toda la organización. Reemplaza cualquier conflicto con políticas locales o de sitio.
4. **4. Unidad Organizativa Padre (OU):** Se aplican después. Por ejemplo, una política vinculada a la OU principal `Equipos`.
5. **5. Unidad Organizativa Hija (OU):** Se aplica al final. Es la más cercana al objeto. Por ejemplo, una política vinculada a la OU `Equipos_Laboratorios`. Al aplicarse al final, sobrescribe todo lo anterior.

---

## Aplicación selectiva: seguridad y filtros WMI

Vincular una GPO a una OU hace candidatos a todos sus objetos. Para aplicar una política solo a una parte, Windows ofrece filtrado de seguridad, filtros WMI y condiciones de preferencias:

|Mecanismo|¿Cómo funciona?|Caso de uso ideal|
|---|---|---|
|**Vínculo de OU (El filtro base)**|Define el límite geográfico o de organigrama general.|Aplicar políticas a todos los equipos dentro de la OU `Laboratorios`.|
|**Security Filtering (Filtrado de Seguridad)**|Usa grupos de seguridad de Active Directory para decidir quién aplica y quién no.|Una GPO vinculada a todo el dominio, pero que solo afecta al grupo `GG-Directores-Finanzas`.|
|**Filtros WMI**|Realiza consultas de hardware y sistema operativo al cliente antes de aplicar la GPO.|Aplicar una configuración solo si la computadora es una Laptop (tiene batería) o si tiene Windows 11 instalado.|
|**Item-level targeting (Destinatarios)**|Filtros ultra-específicos exclusivos de la pestaña **Preferencias**.|Mapear una unidad de red `Z:` solo si el usuario tiene una dirección IP de la sucursal Norte.|

### Cómo funciona el Security Filtering (Filtrado de Seguridad)

Por defecto, una GPO usa el grupo **Authenticated Users**, por lo que usuarios y equipos del alcance pueden aplicarla.

Si quieres restringirla a un grupo selecto (por ejemplo, `GG-Restringir-USB`):

1. Selecciona la GPO en la consola GPMC.
2. En la pestaña **Scope** (Alcance), busca la sección **Security Filtering**.
3. Elimina el grupo `Authenticated Users`.
4. Agrega el grupo `GG-Restringir-USB`.
5. **¡Cuidado!** A partir de una actualización de seguridad de Microsoft, las computadoras también necesitan poder _leer_ el contenido de la GPO. Debes ir a la pestaña **Delegation** (Delegación), hacer clic en Add y agregar al grupo `Domain Computers` (o dejar `Authenticated Users`) con el permiso exclusivo de **Read** (pero NO de "Apply Group Policy"). Si olvidas esto, la GPO no se procesará en absoluto.

### Filtros WMI (Windows Management Instrumentation)

Un filtro WMI es una consulta que se ejecuta en el cliente: si devuelve verdadero, la GPO se aplica; si devuelve falso, se ignora.

**Ejemplo de filtro WMI:**

```sql
SELECT * FROM Win32_OperatingSystem WHERE ProductType = 1
```

`ProductType = 1` identifica sistemas cliente como Windows 10 u 11; los servidores devuelven 2 o 3 y quedan fuera del alcance.

---

## Crear y administrar GPO con GPMC

La consola recomendada es **Group Policy Management** (`gpmc.msc`). Desde ella se crean GPO, se administran vínculos, delegación, filtros, copias de seguridad, reportes y resultados.

### Flujo seguro de implementación

1. **Definir el objetivo:** escribir qué riesgo o necesidad resuelve la política y a quién afecta.
2. **Preparar el alcance:** ubicar objetos en una OU de prueba o crear el grupo de filtrado correspondiente.
3. **Crear la GPO sin vincular:** usar un nombre descriptivo, por ejemplo `USR - Laboratorio - Restringir panel`.
4. **Editar únicamente lo necesario:** evitar mezclar configuraciones sin relación.
5. **Vincular a prueba:** aplicar primero a pocos usuarios o equipos representativos.
6. **Validar:** usar `gpupdate`, `gpresult`, RSOP y el Visor de eventos.
7. **Documentar y desplegar:** registrar responsable, cambio, alcance, fecha y plan de reversión.
8. **Respaldar:** crear una copia desde GPMC antes y después de cambios importantes.

---

## Políticas de seguridad esenciales

### Contraseñas y bloqueo de cuentas del dominio

Para cuentas del dominio, la política general de contraseña, bloqueo y Kerberos debe definirse en una GPO vinculada al **nivel del dominio**. Vincularla solo a una OU de usuarios no crea políticas de contraseña distintas para esas cuentas.

| Configuración             | Qué controla                                 | Criterio de diseño                                                              |
| ------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------- |
| Longitud mínima           | Número mínimo de caracteres.                 | Favorecer frases largas y combinarla con protección contra contraseñas débiles. |
| Historial                 | Impide reutilizar contraseñas recientes.     | Evita alternar inmediatamente entre valores conocidos.                          |
| Edad mínima/máxima        | Cuándo puede o debe cambiarse.               | Definir según el estándar de la organización y el riesgo, no por costumbre.     |
| Umbral de bloqueo         | Intentos fallidos antes de bloquear.         | Equilibrar ataques de fuerza bruta y bloqueos maliciosos.                       |
| Duración/restablecimiento | Cuándo se desbloquea o reinicia el contador. | Coordinar ambos valores y monitorear eventos.                                   |

---

## Restricciones del entorno y de los dispositivos

Las restricciones deben responder a un riesgo concreto. Bloquear indiscriminadamente dificulta el trabajo, aumenta soporte y puede llevar a los usuarios a buscar alternativas inseguras.

|Necesidad|Área de GPO|Consideración|
|---|---|---|
|Ocultar o impedir Panel de control y Configuración|User Configuration → Administrative Templates → Control Panel|Aplicar solo a perfiles que no administran el equipo.|
|Impedir acceso a CMD|User Configuration → Administrative Templates → System|No sustituye control de aplicaciones; PowerShell y otras vías pueden seguir disponibles.|
|Bloquear almacenamiento USB|Computer Configuration → Administrative Templates → System → Removable Storage Access|Probar periféricos necesarios y definir excepciones.|
|Restringir instalación de dispositivos|Computer Configuration → Administrative Templates → System → Device Installation|Puede filtrar por identificadores de clase o dispositivo.|
|Bloquear pantalla|User Configuration → Administrative Templates → Control Panel → Personalization|Definir tiempo, protector seguro y contraseña al reanudar.|
|Mapear unidad de red|User Configuration → Preferences → Windows Settings → Drive Maps|Usar grupos e Item-level targeting.|
|Controlar aplicaciones|AppLocker o Windows Defender Application Control|Comenzar en modo auditoría para descubrir bloqueos inesperados.|
|Configurar actualizaciones|Computer Configuration → Administrative Templates → Windows Components → Windows Update|Separar anillos de prueba y producción.|

---

## Restricciones de logon, equipos y horarios

Controlar el acceso físico y lógico de los usuarios es vital para la seguridad. Sin embargo, los administradores novatos suelen confundir **quién** controla el acceso (Active Directory) y **desde dónde** se autoriza (las GPOs de los equipos). A continuación aclaramos esta diferencia clave.

### Diferenciando los controles de acceso en Windows

|Control|¿Dónde se configura?|¿Qué problema resuelve?|Ejemplo Práctico|
|---|---|---|---|
|**Logon Hours (Horas de Inicio)**|En la pestaña _Account_ del usuario en **dsa.msc** (ADUC).|Define las horas y días en que la cuenta del usuario está autorizada a validarse en el dominio.|Impedir que un cajero inicie sesión un domingo por la tarde.|
|**Log On To (Iniciar sesión en)**|En la pestaña _Account_ del usuario en **dsa.msc** (ADUC).|Define una lista blanca de nombres de computadoras donde este usuario tiene permitido sentarse.|Permitir que la recepcionista solo inicie sesión en la computadora `PC-RECEPCION`.|
|**User Rights (Asignación de Derechos)**|Dentro de una GPO en: `Configuración de Equipo → Directivas → Configuración de Windows → Configuración de Seguridad → Directivas Locales → Asignación de derechos de usuario`.|Define qué identidades (usuarios o grupos) tienen permitido interactuar técnicamente con el sistema operativo de esa máquina.|Permitir únicamente al grupo `GG-Soporte-RDP` conectarse vía Escritorio Remoto (RDP) a esta máquina.|

### ¿Cómo configurar el horario de inicio de sesión?

1. Abre **Active Directory Users and Computers** (`dsa.msc`).
2. Busca la cuenta del estudiante u operador, haz clic derecho y selecciona **Properties**.
3. Ve a la pestaña **Account** y haz clic en el botón **Logon Hours**.
4. Verás una cuadrícula de 7x24 (días de la semana y horas). Selecciona las celdas y haz clic en **Logon Permitted** (Azul) para permitir o **Logon Denied** (Blanco) para prohibir.
5. Para hacerlo masivamente por PowerShell o CMD, puedes usar el comando `net user`:

	```powershell
	# Permitir al usuario 'aperez' iniciar sesión de Lunes a Viernes, de 7 AM a 6 PM
	net user aperez /domain /times:L-V,07:00-18:00
	
	# Consultar la cuenta para verificar sus horarios configurados
	net user aperez /domain
	```
