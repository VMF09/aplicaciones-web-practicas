# Guión de documentación

Ubuntu Server, Apache2 y MariaDB

Aplicaciones Web - 2.º SMR - UD1

Nombre: Verónica Muñoz de Francisco    
Curso: 2º SMX Vesprada    
Fecha: 14/10/2026

## 1. Instalar un servidor Ubuntu sin interfaz gráfica

### 1.1. Detalles del SO escogido.

Se Utiliza Ubuntu Server 24.10LTS de 64 Bits, la edición Server y sin entorno gráfico.  Es GNU/Linux con soporte de larga duración, administrable desde la terminal y también con menor consumo de recursos que una edición de escritorio.  Tiene un total de 4GB RAM, disco VDI dinámico de 30GB y dos CPU.  Se instala desde su iso correspondiente descargado desde la página oficial.

### 2.2 Configuración de VirtualBox.

1. Archivo > Herramientas de red del anfitrión para crear un adaptador Host-only ( normalmente es vboxnet0 ), red 192.168.56.0/24; DHCP puede quedar desactivado porque la IP se fijará con netplan.
2. Creamos la VM Linux, Ubuntu de 64 bits, asignamos al menos 2GB de RAM y creamos un disco VDI dinámico de como mínimo 25GB.
3. Con nuestra VM apagada, activamos Adaptador 1 en NAT para Internet y Adaptador 2 en Adaptador solo anfitrión, selecionando vboxnet0.
4. Montamos la ISO y arrancamos la máquina para su instalación.

### 1.3 Proceso de instalación.

Elegimos el idioma y teclado, Ubuntu Server ( no minimized ), disco completo y nuestro perfil de usuario.  También dejamos las interfaces con DHCP inicialmente.  En SSH Setup, marcamos Install OpenSSH server; no selecionamos snaps opcionales.  Tras la instalación, iniciamos sesión e identificamos interfaces:  la NAT suele ser 10.0.2.x y Host-Only utilizará 192.168.56.x.

Comandos que debemos utilizar :

```bash
ip a
ping -c 3 archive.ubuntu.com
ls /etc/netplan/
sudo nano /etc/netplan/50-cloud-init-yaml
```

Ejemplo del contenido YAML ( esto no es un comando, es para sustituir nombres de interfaz y ficheros ) :

```yaml
network:
 version: 2
 ethernets:
   enp0s3:
     dhcp4: true
   enp0s8:
     dhcp4: false
     addresses:
       - 192.168.56.10/24
```

Usamos espacios y también respetamos la indentación.  Después de aplicar y verificar, desde el anfitrión nos conectamos por SSH y aceptamos la huella inicial escribiendo 'yes'.

```bash
sudo netplan apply
ip a
```

En PowerShell ( si estás en Windows 11, si no, en la terminal de Linux ) :

```bash
ssh vbox_vmf@192.168.65.10
sudo apt update
sudo apt install openssh-server
```

## 2. Gestionando un servidor por SSH.

### 2.1 ¿Qué es SSH?

SSH ( secure shell ) es un protocolo cifrado para iniciar sesión y ejecutar comandos en otro equipo por red.  También permite administrar Ubuntu Server desde el anfitrión sin usar la consola gráfica de VirtualBox.

Comando que siempre se utiliza :

```bash
ssh usuario@IP_SERVIDOR
```

### 2.3 Configuración SSH.

La VM utiliza dos interfazes, la NAT ( por ejemplo, enp0s3 ) recibe internet, normalmente con DHCP y una IP 10.0.2.x Host-Only ( por ejemple, enp0s8 ) crea la reed privada entre anfitrión y la VM, esto se fijaría con Netplan, por ejemplo nuestra IP 192.168.56.10/24.  Desde el anfitrión se accede por la IP Host-only.  Los nombres reales tienen que ser consultados con ip a y también pueden cambiar según la VM.

Comandos utilizados :

```bash
ip a
ls /etc/netplan/
sudo nano /etc/netplan/50-cloud-init.yaml
sudo netplan apply
systemctl is-active ssh
systemctl is-enabled ssh
ssh vmf@192.168.56.10
```

"active" es lo que describe el estado actual, "enabled" nos indica que systemd lo arrancará automáticamente.  Si OpenSSH Server no se instaló, debemos utilizar sudo apt update y sudo apt install openssh-server.

## 3. Servidor web Apache2.

### 3.1. ¿Qué es Apache2?

Apache HTTP Server es un servidor wed de código abierto mantenido por Apache Software Foundation, este recibe peticiones HTTP y devuelve recursos como HTML e imágenes o contenido generado por módulos.  Es la A de LAMP :   Linux, Apache, MariaDB / MySQL y PHP.

### 3.2 Instalación de Apache 2.

Comandos que se utilizan :

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl status apache2
sudo ufw allow 'Apache'
ip a
curl -I localhost
```

La página predeterminada puede comprobarse a través de http://192.168.56.10.  Si esta no carga, hay que revisar la IP, firewall y su estado.  configtest valida su sitaxis, access.log y error.log ayudan con su diagnóstico.

```bash
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2
sudo systemctl status apache2
sudo apachectl configtest
sudo journalctl -u apache2 -f
sudo tail -f /var/log/apache2/access.log
sudo tail -f /var/log/apache2/error.log
```

### 3.3 Ficheros y directorios más importantes.

/etc/apache2/apache2.conf : eso es la configuración global.
/etc/apache2/ports.conf :  aquí encontramos puertos de escucha, por ejemplo 80.
/etc/apache2/sites-available/ :  estos son los VirtualHosts disponibles.
/etc/apache2/sites-enabled/ :  aquí tenemos enlaces simbólicos de sitios activos.
/etc/apache2/mods-available/ y mods-enabled/ :  módulos disponibles y activos.
/etc/apache2/conf-available/ y conf-enabled/ :  fragmentos de configuración.
/var/www/html/ :  encontramos la raíz web predeterminada.
/var/log/apache2/ :  access.log y error.log.

available significa disponible; enabled significa habilitado mediante enlace simbólico. a2ensite/a2dissite y a2enmod/a2dismod activan/desactivan sitios y módulos. Tras todos los cambios validamos y recargamos.

```bash
sudo a2enmod rewrite
sudo a2dismod rewrite
sudo apache2ctl -M
sudo systemctl reload apache2
```

### 3.4 VirtualHost :   qué es, configuración básica por IP y puerto, habilitar y deshabilitar.

Un VirtualHost define cómo atender un sitio según dirección, puerto y nombre solicitado, e indica su DocumentRoot. Cada sitio suele tener un .conf en sites-available; a2ensite lo activa y a2dissite lo desactiva. Apache debe escuchar en todo puerto utilizado.

```bash
sudo mkdir -p /var/www/192.168.56.10
sudo nano /etc/apache2/sites-available/192.168.56.10.conf
```

Contenido de ejemplo del archivo ( esto no es un comando, sino el contenido del fichero ) :

```apache
<VirtualHost 192.168.56.10:80>
   ServerName 192.168.56.10
   DocumentRoot /var/www/192.168.56.10
   ErrorLog ${APACHE_LOG_DIR}/ip-error.log
   CustomLog ${APACHE_LOG_DIR}/ip-access.log combined
</VirtualHost>
```

Para usar el puerto 8080, añadir Listen 8080 en ports.conf y crear otro VirtualHost con *:8080. Si varios sitios comparten IP y puerto, ServerName distingue las peticiones. Si UFW está activo, permitir el puerto.

Ejemplo de VirtualHost para el puerto 8080 ( esto es el contenido del fichero, no un comando ):

```apache
<VirtualHost *:8080>
   ServerName sitio-puerto.local
   DocumentRoot /var/www/sitio-puerto
</VirtualHost>
```

Comandos utilizados :

```bash
sudo nano /etc/apache2/ports.conf
sudo nano /etc/apache2/sites-available/puerto8080.conf
sudo a2ensite 192.168.56.10.conf
sudo a2ensite puerto8080.conf
sudo a2dissite 000-default.conf
sudo ufw allow 8080/tcp
sudo apachectl configtest
sudo systemctl reload apache2
curl -I http://192.168.56.10/
curl -I http://192.168.56.10:8080/
```

### 3.5. VirtualHost por nombre y fichero hosts.

Utilizar nombres ficticios como sitio-alumnno.local nos permite elegir el sitio por ServerName.  Para resolverlo sin ningún DNS añadimos en el archivo hosts del equipo que ejecuta el navegador la IP del servidor y el nombre.  En Windows el archivo está en C:\Windows\System32\drivers\etc\hosts; esto requiere permisos de administrador.  En Ubuntu cliente sería /etc/hosts.  La modificación de este archivo solo afecta a ese cliente.

```bash
sudo mkdir -p /var/www/sitio-alumno
sudo nano /var/www/sitio-alumno/index.html
sudo nano /etc/apache2/sites-available/sitio-alumno.conf
```

Contenido del VirtualHost ( esto es un archivo ) :  

```apache
<VirtualHost *:80>
   ServerName sitio-alumno.local
   DocumentRoot /var/www/sitio-alumno
   <Directory /var/www/sitio-alumno>
       Require all granted
       AllowOverride None
   </Directory>
</VirtualHost>
```

Comandos utilizados :

```bash
sudo chown -R www-data:www-data /var/www/sitio-alumno
sudo chmod -R 755 /var/www/sitio-alumno
sudo a2ensite sitio-alumno.conf
sudo a2dissite 000-default.conf
sudo apachectl configtest
sudo systemctl reload apache2
curl -H "Host: sitio-alumno.local" http://127.0.0.1/
sudo nano /etc/hosts
```

Añadimos al hosts del cliente una línea tal como esta :

```text
192.168.56.10 sitio-alumno.local
```

Después comprobamos http://sitio-alumno.local en nuestro navegador.

### 3.6 Directorio protegido por contraseña: Auth-Basic.

Auth-Basic nos pide usuario y contraseña.  Basic codifica credenciales en Base64, no las cifra.  En un servidor real debe usarse HTTPS.  Debemos guardar el archivo de contraseñas fuera de la raíz web.

```bash
sudo apt update
sudo apt install apache2-utils -y
sudo mkdir -p /var/www/sitio-alumno/privado
sudo htpasswd -c /etc/apache2/.htpasswd alumno
sudo htpasswd /etc/apache2/.htpasswd otro_usuario
sudo chown root:www-data /etc/apache2/.htpasswd
sudo chmod 640 /etc/apache2/.htpasswd
sudo nano /etc/apache2/sites-available/sitio-alumno.conf
```

Incluimos estas directivas en nuestro VirtualHost, permitimos AuthConfig con AllowOverride solo si se configura la autenticación en .htaccess :

```apache
<Directory /var/www/sitio-alumno/privado>
   AuthType Basic
   AuthName "Zona privada"
   AuthUserFile /etc/apache2/.htpasswd
   Require valid-user
</Directory>
```

De otro modo, para usar .htaccess hay que permitirlo en la configuraciñon del sitio con AllowOverride AuthConfig dentro de <Directory /var/www/sitio-alumno/privado>, y guardamos allí las directivas siguientes :

```apache
AuthType Basic
AuthName "Zona privada"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

Comandos utilizados :

```bash
sudo apachectl configtest
sudo systemctl reload apache2
curl -i http://sitio-alumno.local/privado/
curl -i -u alumno http://sitio-alumno.local/privado/
```

Sin ninguna credencial se espera HTTP 401, con credenciales válidas se sirve el contenido. curl -u solicita la contraseña.  Para deshabilitar este sitio utilizamos :  sudo a2dissite sitio-alumno.conf, validamos configtest y recargamos.

## 4. Base de datos MariaDB

### 4.1 ¿Por qué un SGBD como MariaDB? ¿Qué es?

Una base de datos organiza información en tablas para almacenarla, consultarla y actualizarla, WordPress guarda entradas, páginas, comentarios y también configuración.  MariaDB es un sitema gestor de bases de datos relacional :  almacena y protege los datos y también responde consultas.  En LAMP el recorrido es navegador > Apache > PHP > MariaDB.

### 4.2 Instalación de MariaDB

#### Apartado 1. Preparación del sistema.

```bash
sudo apt update
systemctl is-active apache2
```

Explicación corta :   la actualización nos permite consultar versiones recientes de los paquetes disponibles.  Comprobamos que Apache confirma que se parte de la práctica anterior y que el servidor web sigue operativo.

[Comprobación de Apache activo](<img/systemctl is-active apache2.png>)

#### Apartado 2. Instalación de MariaDB.

```bash
sudo apt install mariadb-server -y
mariadb --version
```

Explicación corta :   mariadb-server instala el servicio que ejecuta el motor de base de datos.  mariadb --version comprueba que el cliente de línea de comandos está disponible y muestra la versión.

[Versión de MariaDB instalada](<img/version.png>)

**¿Qué diferencia hay entre mariadb-server y mariadb-client? ¿Cuál has instalado?**

*mariadb-server ejecuta el motor y ofrece el servicio.  mariadb-client proporciona herramientas para poder conectarse y enviar consultas.  El que se instaló fue mariadb-server.*

#### Apartado 3. Comprobación del funcionamiento.

### 3.1 Estado del servicio.

Comandos que utilizamos :

```bash
sudo systemctl status mariadb
systemctl is-enabled mariadb
```

Explicación corta :   active indica que el servicio está ejecutándose en este momento.  enabled indica que se iniciará automáticamente al arrancar el equipo.

[Estado del servicio y entrada en MariaDB](<img/estado y entrada.png>)

### 3.2 Entrar en la base de datos.

Comandos utilizados :

```bash
sudo mariadb
exit
```

Explicación corta :   sudo mariadb abre una sesión administrativa local.  Poder llegar al prompt ya nos confirma que el servidor responde a una conexión.

**¿Qué diferencia hay entre active y enabled? ¿Qué pasaría si MariaDB estuviera active pero no enabled y se reiniciara el servidor?**
*active es el estado actual y enabled hace que systemd arranque el servicio al iniciar el sistema.  Tras su reinicio, MariaDBB no se iniciará automáticamente y WordPress no podría consultar la base hasta poder arrancarlo.*

#### Apartado 4. Comandos principales.

```bash
sudo systemctl start mariadb :   arranca el servicio
sudo systemctl stop mariadb :   detiente el servicio
sudo systemctl restart mariadb :   reinicia el servicio
sudo systemctl status mariadb :   muestra el estado y mensajes recientes
sudo systemctl enable mariadb :   configura el inicio automático al encender el equipo
systemctl is-active mariadb :   muestra si el servicio está activo
```

Explicación corta :   estos comandos permiten controlar el ciclo de vida del servicio sin entrar en el cliente SQL.  enable configura el arranque automático, no sustituye a start para iniciarlo ahora.

#### Apartado 5. ¿Qué pasa si MariaDB está parado?

```bash
sudo systemctl stop mariadb
systemctl is-active mariadb
sudo mariadb
sudo systemctl start mariadb
systemctl is-active mariadb
```

Explicación :   la prueba nos muestra cñomo distinguir un fallo de conexión causado por el servicio detenido y cómo debemos restaurar la disponibilidad.  Esta detención es intencionada para poder hacer la práctica, una vez hemos terminado, volvemos a arrancar MariaDB.

**Si WordPress no pudiera conectar con la base de datos, ¿qué dos comandos usarías para saber si MariaDB está parado?**
*systemctl is-acitve mariadb nos ofrece el estado breve y sudo systemctl status mariadb nos muestra los detalles y mensajes recientes.  Si está parado, sudo systemctl start mariadb lo arrancaría.*

[Error al detener MariaDB y recuperación del servicio](<img/error y recuperación.png>)

#### Apartado 6. Instalación de PHP

WordPress está desarrollado en PHP, se instala el intérprete, el módulo que integra PHP con Apache y las extensiones necesarias para conectarse a MariaDB y cubrir funciones habituales de WordPress.

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y
php -v
sudo systemctl restart apache2
sudo apache2ctl -M
php -m | grep -i mysql
```

php instala el lenguaje, libapache2-mod-php integra PHP con Apache, php-mysql permite conexión con MariaDB, php-curl realiza las conexiones HTTP, php-gd procesa imágenes, php-mbstring trata texto multibyte, php-xml procesa XML, php-zip maneja ZIP, php-intl aporta las funciones de internacionalización.  Se esperan mpm_prefork_module y php_module, php -m debería mostrar mysqli, mysqlnd y pdo_mysql.

[Versión de PHP](<img/php -v.png>)

Explicación corta :   el reinicio permite a Apache cargar el módulo PHP recién instalado.  Las extensiones MySQL permiten que las aplicacones PHP, como WordPress, se comuniquen con MariaDB.

```bash
echo '<?php phpinfo();' | sudo tee /var/www/html/info.php
```

Abrimos http://192.168.56.10/info.php en el navegador para poder comprobar que Apache ejecuta PHP.  El archivo de prueba expone información detallada del entorno, por lo que se elimina obligatoriamente al finalizar la comprobacion con los siguientes comandos :

```bash
sudo rm /var/www/html/info.php
ls /var/www/html
```
[Comprobación de PHP mediante phpinfo](<img/php.info.png>)

**¿Por qué es peligroso dejar info.php en un servidor real? ¿Para qué sirve php-mysql?**
*info.php nos revela información detallada de versiones y configuraciones, debe eliminarse tras la prueba.  php.mysql nos instala las extensiones para que PHP pueda comunicarse con MariaDB/MySQL.*

#### Apartado 7. Comprobación final.

```bash
systemctl is-active apache2 mariadb
php -v
```

Explicación corta :   la verificación conjunta nos confirma que los servicios que forman la pila están en marcha y que PHP está correctamente instalado.  Antes de terminar, tomamos una instantánea de la VM como punto de partidaa para instalar WordPress.

Estado final de LAMP :

Pieza de LAMP :   Programa > Estado esperado
Linux :   Ubuntu Server 24.10 > Instalado
Apache :   Apache2 > Funcionando
MariaDB :   MariaDB 10.11 > Funcionando
PHP :   PHP 8.2 > Funcionando y preparado para MariaDB

## Conclusiones y problemas encontrados :

Durante la instalación y la comprobación del entorno LAMP ( Apache + PHP + MariaDB ), hemos conseguido que el servidor mostrara correctamente la página info.php, con esto hemos verificado que PHP estaba instalado y funcionando.  Finalmente, el servidor nos respondió como debía y pudimos validar la configuración del sistema sin errores.

Encontré un error durante la instalación, este fue que el archivo info.php devolvía error 404.  Aunque el archivo estaba en /var/www/html, Apache no estaba usando esa carpeta como DocumentRoot para las peticiones al puerto 80.  Para poder solucionarlo, he revisado la configuración con apache2ctl -S y descubrí que el VirtualHost por defecto era intranet.local, cuyo DocumentRoot era /var/www/intranet.  Moví info.php a esa carpeta y el servidor empezó a funcionar correctamente. También tenía un error de multiples VirtualHosts activos a causa de practicas anteriores, esto hacía que la IP sin dominio no usara /var/www/html.  Para solucionar esto, identifiqué cuál era el sitio por defecto y ajusté la ubicación del archivo para que coincidiera con el DocumentRoot real.  Otro error que encontré fue que Apache no estaba interpretando PHP.  Solucioné esto confirmando que libapache2-mod-php estaba instalado y reinicié Apache para asegurar que el módulo estuviera cargado.
