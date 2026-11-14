---
Fecha de creación: 2026-10-12T17:58:00
Materia:
  - Adminstración de Servidores
Fecha de clase: 2026-10-12
---
# Servicio Web con Apache y Certificados SSL

## ¿Qué es un servicio web?

Un **servicio web** recibe solicitudes de clientes mediante HTTP o HTTPS y devuelve recursos: páginas, archivos, datos JSON, imágenes o respuestas generadas por una aplicación.

El navegador suele resolver primero el nombre con DNS, abre una conexión con el servidor y envía una solicitud. En HTTP se usa normalmente TCP 80; en HTTPS, TCP 443. HTTPS agrega TLS para autenticar al servidor y proteger los datos en tránsito.

- **Contenido estático.** HTML, CSS, JavaScript, imágenes y descargas que el servidor entrega desde archivos.
- **Contenido dinámico.** Una aplicación calcula la respuesta según datos, usuario, ruta o reglas del negocio.
- **API.** Expone operaciones y datos para aplicaciones web, móviles u otros sistemas.

|Componente|Responsabilidad|Ejemplo|
|---|---|---|
|Cliente|Solicita recursos y presenta la respuesta.|Navegador o aplicación móvil.|
|DNS|Traduce un nombre a una dirección IP.|`web.aula.test → 192.168.50.10`|
|Servidor web|Atiende HTTP/HTTPS, archivos y proxy inverso.|Apache HTTP Server.|
|Servidor de aplicaciones|Ejecuta la lógica del sistema.|Tomcat o Node.js.|
|Base de datos|Conserva y consulta información persistente.|MySQL o PostgreSQL.|

---

## Apache HTTP Server

**Apache HTTP Server** es un servidor web modular. Puede publicar archivos, alojar varios sitios mediante Virtual Hosts, terminar conexiones TLS, redirigir solicitudes y trabajar como proxy inverso frente a aplicaciones.

**Instalación e inspección**

```bash
sudo apt update
sudo apt install apache2
sudo systemctl enable --now apache2
sudo systemctl status apache2 --no-pager
apache2ctl -v
sudo apache2ctl configtest
sudo ss -lntp '( sport = :80 or sport = :443 )'
curl -I http://127.0.0.1
```

`systemctl` controla el servicio; `configtest` revisa la sintaxis antes de recargar; `ss` confirma qué proceso escucha y `curl -I` solicita únicamente los encabezados HTTP.

|Ruta en Ubuntu|Uso|
|---|---|
|`/var/www/html/`|Raíz documental predeterminada.|
|`/etc/apache2/apache2.conf`|Configuración general.|
|`/etc/apache2/sites-available/`|Definiciones disponibles de Virtual Hosts.|
|`/etc/apache2/sites-enabled/`|Sitios habilitados mediante enlaces administrados por `a2ensite`.|
|`/etc/apache2/mods-available/`|Módulos disponibles; se habilitan normalmente con `a2enmod`.|
|`/var/log/apache2/`|Registros de acceso y errores.|

---

## Publicar un sitio con Virtual Host

Un **Virtual Host** permite servir varios nombres desde una misma dirección IP. Apache compara el nombre solicitado con `ServerName` y `ServerAlias` para elegir la configuración.

**Directorio y archivo de prueba**

```bash
sudo install -d -o www-data -g www-data -m 0755 /var/www/aula/public
sudo nano /var/www/aula/public/index.html
sudo nano /etc/apache2/sites-available/aula.conf
```

**/etc/apache2/sites-available/aula.conf**

```html
<VirtualHost *:80>
    ServerName web.aula.test
    DocumentRoot /var/www/aula/public

    <Directory /var/www/aula/public>
        Options -Indexes
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/aula-error.log
    CustomLog ${APACHE_LOG_DIR}/aula-access.log combined
</VirtualHost>
```

**Habilitar y verificar**

```bash
sudo a2ensite aula.conf
sudo apache2ctl configtest
sudo apache2ctl -S
sudo systemctl reload apache2
curl -I -H 'Host: web.aula.test' http://127.0.0.1
```

`apache2ctl -S` muestra cómo Apache interpretó los Virtual Hosts. `reload` carga cambios sin interrumpir deliberadamente las conexiones activas; úsalo solo después de una validación correcta.

> [!INFO] El nombre debe resolver hacia el servidor
> En un laboratorio puede existir en DNS interno o temporalmente en `/etc/hosts`; para un certificado público debe usarse un dominio controlado y verificable.

---

## HTTPS, TLS y el certificado

Aunque se dice habitualmente “certificado SSL”, los sistemas actuales usan **TLS**, sucesor de SSL. El certificado vincula una identidad —normalmente un nombre DNS— con una clave pública y está firmado por una autoridad en la que el cliente decide confiar.

1. **Negociación.** Cliente y servidor acuerdan una versión de TLS y algoritmos compatibles.
2. **Identidad.** El servidor presenta su certificado y los certificados intermedios necesarios.
3. **Validación.** El cliente comprueba firma, cadena de confianza, fechas y nombre solicitado.
4. **Claves de sesión.** Ambas partes establecen secretos temporales para proteger la conexión.
5. **HTTP protegido.** Solicitudes y respuestas viajan cifradas y con controles de integridad.

|Elemento|Función|Cuidado principal|
|---|---|---|
|Clave privada|Demuestra control de la identidad del servidor.|No compartir; permisos restrictivos.|
|Certificado del servidor|Contiene identidad, clave pública y vigencia.|Debe incluir el nombre en SAN.|
|Cadena intermedia|Conecta el certificado con una raíz confiable.|Entregar la cadena completa.|
|Autoridad certificadora|Firma certificados después de validar identidad o control.|El cliente debe confiar en su raíz.|

---

## Certificado autofirmado para una red de laboratorio

Un certificado **autofirmado** usa su propia clave para firmarse. Cifra la conexión, pero un navegador no lo confía automáticamente. Es útil en pruebas controladas; para eliminar la advertencia debe distribuirse una CA interna confiable o usarse un certificado público válido.

**Crear clave y certificado**

```bash
sudo install -d -m 0755 /etc/ssl/localcerts
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/web-aula-test.key \
  -out /etc/ssl/localcerts/web-aula-test.crt \
  -subj "/CN=web.aula.test" \
  -addext "subjectAltName=DNS:web.aula.test"
sudo chmod 600 /etc/ssl/private/web-aula-test.key
openssl x509 -in /etc/ssl/localcerts/web-aula-test.crt -noout -subject -issuer -dates -ext subjectAltName
```

`-nodes` crea una clave sin frase de paso para que Apache pueda arrancar sin intervención. Por eso el archivo debe quedar protegido. El **SAN** contiene el nombre que los clientes validan; confiar solo en `CN` ya no es suficiente.

Guarda el siguiente bloque como `/etc/apache2/sites-available/aula-ssl.conf`:

**Virtual Host HTTPS**

```html
<VirtualHost *:443>
    ServerName web.aula.test
    DocumentRoot /var/www/aula/public

    SSLEngine on
    SSLCertificateFile /etc/ssl/localcerts/web-aula-test.crt
    SSLCertificateKeyFile /etc/ssl/private/web-aula-test.key

    ErrorLog ${APACHE_LOG_DIR}/aula-ssl-error.log
    CustomLog ${APACHE_LOG_DIR}/aula-ssl-access.log combined
</VirtualHost>
```

Activar TLS y comprobar

```bash
sudo a2enmod ssl
sudo a2ensite aula-ssl.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -vkI https://web.aula.test
openssl s_client -connect web.aula.test:443 -servername web.aula.test -showcerts </dev/null
```

> [!WARNING] `curl -k` omite la validación y sirve solo para diagnóstico
> No demuestra que la cadena sea confiable. Una prueba real debe ejecutarse sin `-k` desde un cliente que confíe en la CA correspondiente.

---

## Certificado público con Let's Encrypt

**Let's Encrypt** es una autoridad certificadora pública y automatizada. Certbot utiliza el protocolo ACME para demostrar control del dominio, solicitar el certificado, configurar Apache y gestionar renovaciones.

|Requisito|Por qué importa|
|---|---|
|Dominio registrado|El certificado se emite para nombres, no para un nombre ficticio de laboratorio.|
|DNS correcto|Los registros deben apuntar al servidor que responderá la validación.|
|TCP 80 público para HTTP-01|La autoridad consulta un recurso temporal para comprobar el control.|
|Virtual Host con `ServerName`|Certbot identifica el sitio que debe modificar.|
|Hora correcta|Las fechas de validez dependen del reloj del sistema.|

**Certbot con Apache**

```bash
sudo snap install --classic certbot
sudo certbot --apache -d ejemplo.com -d www.ejemplo.com
sudo certbot certificates
sudo certbot renew --dry-run
systemctl list-timers | grep -i certbot
```

El complemento de Apache localiza el Virtual Host, instala las directivas TLS y recarga el servicio. Sustituye los dominios por nombres que controles. `renew --dry-run` simula una renovación y ayuda a detectar problemas antes del vencimiento.

---

## Publicar Apache con UFW

UFW debe permitir únicamente los servicios que realmente se publican. Apache instala perfiles de aplicación; conviene inspeccionarlos antes de utilizarlos.

**Perfiles y reglas web**

```bash
sudo ufw app list
sudo ufw app info 'Apache'
sudo ufw app info 'Apache Full'
sudo ufw app info 'Apache Secure'

# Conservar primero el acceso administrativo requerido
sudo ufw allow proto tcp from 192.168.50.0/24 to any port 22 comment 'SSH administracion'

# Durante emisión HTTP-01 y operación HTTPS
sudo ufw allow 'Apache Full'
sudo ufw enable
sudo ufw status verbose
```

|Perfil|Puertos habituales|Uso|
|---|---|---|
|`Apache`|TCP 80|Solo HTTP.|
|`Apache Secure`|TCP 443|Solo HTTPS.|
|`Apache Full`|TCP 80 y 443|HTTP y HTTPS; útil para redirección y desafíos HTTP-01.|

---

## Apache como proxy inverso

Un **proxy inverso** recibe las conexiones públicas y las reenvía a una aplicación interna. Apache puede centralizar TLS, nombres, registros y cabeceras, mientras Tomcat o Node escuchan solo en una dirección privada.

**Módulos de proxy**

```bash
sudo a2enmod proxy proxy_http headers
sudo apache2ctl configtest
sudo systemctl reload apache2
```

**Fragmento de Virtual Host para Node.js**

```bash
ProxyPreserveHost On
ProxyPass        /api/ http://127.0.0.1:3000/
ProxyPassReverse /api/ http://127.0.0.1:3000/

RequestHeader set X-Forwarded-Proto "https"
```

`ProxyPass` envía la solicitud; `ProxyPassReverse` ajusta cabeceras de redirección devueltas por la aplicación. La aplicación debe configurar explícitamente qué proxies considera confiables antes de aceptar cabeceras reenviadas.

> [!WARNING] Evita convertir Apache en un proxy abierto
> Los destinos deben ser internos y fijos. Si la aplicación usa WebSocket, puede necesitar configuración adicional y pruebas del protocolo de actualización.

---

## Aplicaciones Java con Tomcat

**Apache Tomcat** no es Apache HTTP Server. Tomcat es un contenedor para aplicaciones Java basadas en Jakarta Servlet, JSP y tecnologías relacionadas. Puede ejecutar una API REST con Spring MVC, un portal institucional, un sistema de inventario o una aplicación empaquetada como `.war`.

**Servicio y despliegue**

```bash
# El nombre del paquete y la unidad depende de la versión de Ubuntu
apt-cache policy tomcat10
sudo apt install default-jdk tomcat10
sudo systemctl enable --now tomcat10
sudo systemctl status tomcat10 --no-pager
sudo journalctl -u tomcat10 -n 50 --no-pager
sudo ss -lntp 'sport = :8080'

# Ejemplo: desplegar un WAR como /inventario
sudo install -o tomcat -g tomcat -m 0640 inventario.war /var/lib/tomcat10/webapps/inventario.war
curl -I http://127.0.0.1:8080/inventario/
```

El nombre del archivo suele definir la ruta: `inventario.war` se publica como `/inventario`. Tomcat puede expandir el WAR, crear sesiones y administrar el ciclo de vida de la aplicación.

---

## Aplicaciones y APIs con Node.js

**Node.js** ejecuta JavaScript fuera del navegador. Puede desplegar APIs con Express o Fastify, servicios en tiempo real con WebSocket, servidores para aplicaciones renderizadas en el servidor y tareas que integran otros sistemas.

Preparar y comprobar una aplicación

```bash
node --version
npm --version
cd /srv/node-aula
npm ci --omit=dev
node --check app.js
sudo systemctl enable --now node-aula.service
sudo systemctl status node-aula.service --no-pager
sudo journalctl -u node-aula.service -f
curl -i http://127.0.0.1:3000/health
```

`npm ci` instala exactamente las dependencias del archivo de bloqueo; `--omit=dev` omite dependencias usadas solo en desarrollo. La unidad `node-aula.service` debe crearse para la aplicación con un usuario sin privilegios, directorio de trabajo y variables de entorno controladas.

---

## MySQL y PostgreSQL en la capa de datos

Las bases de datos no reemplazan al servidor web. Guardan usuarios, productos, pedidos, contenido y relaciones que las aplicaciones consultan mediante credenciales. En un sitio sencillo pueden compartir host; en producción suelen separarse por seguridad, capacidad o disponibilidad.

|Motor|Ejemplos de uso|Puerto habitual|
|---|---|---|
|MySQL|CMS, comercio electrónico, catálogos y aplicaciones LAMP.|TCP 3306|
|PostgreSQL|Sistemas transaccionales, datos geográficos con PostGIS, APIs y consultas complejas.|TCP 5432|

**MySQL**

```bash
sudo apt install mysql-server
sudo systemctl enable --now mysql
sudo systemctl status mysql --no-pager
sudo journalctl -u mysql -n 50 --no-pager
sudo mysql -e "SELECT VERSION(); SHOW VARIABLES LIKE 'bind_address';"
sudo ss -lntp 'sport = :3306'
```

**PostgreSQL**

```bash
sudo apt install postgresql
sudo systemctl enable --now postgresql
sudo systemctl status postgresql --no-pager
pg_lsclusters
sudo -u postgres psql -c 'SELECT version();'
sudo ss -lntp 'sport = :5432'
```

Para acceso remoto no basta con UFW: el motor debe escuchar en la interfaz adecuada y autorizar al usuario y origen. En PostgreSQL intervienen `listen_addresses` y `pg_hba.conf`; en MySQL, `bind-address` y los permisos de cuenta.

**UFW si la aplicación está en otro host**

```bash
# Solo el servidor de aplicaciones 192.168.50.20
sudo ufw allow proto tcp from 192.168.50.20 to any port 3306 comment 'MySQL desde app'
sudo ufw allow proto tcp from 192.168.50.20 to any port 5432 comment 'PostgreSQL desde app'
```
