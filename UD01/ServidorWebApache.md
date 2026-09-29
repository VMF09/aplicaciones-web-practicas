# Intranet con Apache2: intranet.local

## 1. Crear estructura del sitio
```
bash
sudo mkdir -p /var/www/intranet/facturas
sudo chown -R www-data:www-data /var/www/intranet
ls -R /var/www/intranet
```

> Creamos la raíz de la intranet y el directorio protegido facturas con permisos para Apache.

## 2. Página de inicio
```
sudo nano /var/www/intranet/index.html
<h1>Bienvenidos a la intranet de tu empresa</h1>
<a href="/facturas">Acceder a facturas</a>
```

> Así podemos preparar la estructura básica de la intranet y la página principal con un mensaje y el enlace hacia el área protegida.

## 3. Configurar el VirtualHost intranet.local

Creamos el vhost :
```
sudo nano /etc/apache2/sites-available/intranet.conf
```

> Añadimos el contenido :
```
<VirtualHost *:80>
ServerName intranet.local
DocumentRoot /var/www/intranet
<Directory /var/www/intranet>
Options -Indexes
AllowOverride None
Require all granted
</Directory>
</VirtualHost>
```

> Por ultimo, activamos el sitio con:
```
sudo a2ensite intranet.conf
sudo systemctl reload apache2
```


## 4. Proteger /facturas con un usuario y contraseña

> Primeramente, activamos la auntenticación básica solamente en el directorio de facturas :
```
sudo htpasswd -c /etc/apache2/intranet.htpasswd admin
```

> Después, añadimos dentro del mismo archivo este contenido:
```
<Directory /var/www/intranet/facturas>
AuthType Basic
AuthName "Zona de facturas"
AuthUserFile /etc/apache2/intranet.htpasswd
Require valid-user
</Directory>
```

Y recargamos con :
```
sudo systemctl reload apache2
```

## 5. Ocultar la información del servidor

> Empezamos reduciendo la información que Apache nos muestra sobre su versión y módulos.  
> De esta forma, podemos editar la seguridad global:
```
sudo nano /etc/apache2/conf-available/security.conf
```

> Aseguramos que vemos esto en el archivo :
```
ServerTokens Prod
ServerSignature Off
```

> Una vez listo, activamos y regargamos :
```
sudo a2enconf security
sudo systemctl reload apache2
```
## 6. Activar SSL y crear un certificado autofirmado

> Activamos el módulo SSL de la siguiente forma :
```
sudo s2enmod ssl
```

> Creamos la carpeta de los certificados :
```
sudo mkdir -p /etc/apache2/ssl
```

> Ya creada, generamos el certificado con el siguiente comando :
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/apache2/ssl/intranet.key \
-out /etc/apache2/ssl/intranet.crt \
-subj "/CN=intranet.local"
```

> Esto nos sirve para definir los sitios HTTP/HTTPS, creando el certificado autofirmado, habilitamos HTTPS en la intranet.

## 7. Crear VirtualHost HTTPS para intranet.local

> Para hacer que la intranet también se sirva por el puerto 443 con SSL, primero editamos el vhost :
```
sudo nano /etc/apache2/sites-available/intranet.conf
```

> Seguidamente, añadimos el siguiente bloque ( ojo, los dos bloques VirtualHost tienen que tener </VirtualHost> al final de cada bloque individual, si se añaden al final de todo el texto, no funcionará y nos dará error):
```
<VirtualHost *:443>
ServerName intranet.local
DocumentRoot /var/www/intranet
SSLEngine on
SSLCertificateFile /etc/apache2/ssl/intranet.crt
SSLCertificateKeyFile /etc/apache2/ssl/intranet.key
</VirtualHost>
```

> Recargamos :
```
sudo systemctl reload apache2
```

## 8. Comprobar funcionamiento
> Verificamos que todo responde como debe y comprobamos que el acceso a "facturas" está protegido y nos pide un usuario y contraseña.
