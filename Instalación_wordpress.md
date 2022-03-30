
# Pasos previos

- Creamos una maquina virtual con vagrant con apache2

- Instalamos el servicio de mariasql

    apt update
	apt install mariadb-server

- Configurar mysql

	mysql_secure_installation
	systemctl status mariadb


- Instalar librerías básicas de php

	apt install apache2 libapache2-mod-php php php-mysql


# Configurar entorno

- Instalar dependecias necesarias de php

    apt install php-curl php-gd php-xml php-mbstring php-xmlrpc php-zip php-soap php-intl
    apt install php7.4-mysqli

- Cambios el orden de los ficheros de html por el de php, para que primero trate de abrir el fichero php y sino el html

    nano /etc/apache2/mods-enabled/dir.conf
 
- Reiniciar apache
    
    systemctl restart apache2

-   Entramos en mysql y creamos una base de datos para wordpress

    mysql -u root –p
    CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
    Ahora creamos un usuario especial para WordPress:

    CREATE USER 'wordpressuser'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

-   Agregamos privilegios al usuario para acceder a la base de datos:
    
    GRANT ALL ON wordpress.* TO 'wordpressuser'@'%';

-   Para confirmar los cambios escribimos:
    
    FLUSH PRIVILEGES;

    Si queremos ver los usuarios creados y las bases de datos, los comandos son:

    SHOW DATABASES;
    select host, user, password from mysql.user;

- Creamos el archivo de configuración de host virtual

    Creamos carpeta para worpress mkdir /var/www/wordpress

    Agregamos la propiedad del directorio con la variable de entorno $USER:
        sudo chown -R $USER:$USER /var/www/wordpress

    Creamos el archivo de configuración para el sitio WordPress:
        nano  /etc/apache2/sites-available/wordpress.conf

        <VirtualHost *:80>
        ServerAdmin webmaster@wordpress.miservidor.com
        DocumentRoot /var/www/wordpress
        ServerName wordpress.miservidor.com
        ServerAlias www.wordpress.miservidor.com
        <Directory />
            Options FollowSymLinks
            AllowOverride None
        </Directory>
        <Directory /var/www/wordpress>
            AllowOverride All
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

    Habilitamos la función mod_write y el nuevo host virtual:
        sudo a2enmod rewrite
        sudo a2ensite wordpress
        sudo systemctl restart apache2

# Instalar wordpress

- Toca hacer la configuración e instalación de WordPress, lo haremos en una carpeta temporal y posteriormente lo pasaremos  a  su carpeta final:

    cd /tmp
    curl -O https://wordpress.org/latest.tar.gz

- Extraemos el archivo:

    tar xzvf latest.tar.gz

    Creamos el archivo .htaccess y lo guardamos:

    sudo nano /tmp/wordpress/.htaccess

- Cambiamos el nombre del archivo de muestra wp-config-sample.php:

    mv /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php

- Creamos la carpeta upgrade:

    mkdir /tmp/wordpress/wp-content/upgrade

- Copiamos la configuración en la carpeta de nuestro nombre de dominio:

    sudo cp -a /tmp/wordpress/. /var/www/wordpress

- Agregamos la propiedad a los usuarios y grupos de www-data:

    sudo chown -R www-data:www-data /var/www/wordpress

- Creamos permisos a las carpetas:

    sudo find /var/www/wordpress/ -type d -exec chmod 755 {} \;
    sudo find /var/www/wordpress/ -type f -exec chmod 644 {} \;

- Generamos las claves salt de WordPress:

    curl -s https://api.wordpress.org/secret-key/1.1/salt

    Y el resultado lo incluimos en el fichero de configuración.
        sudo nano /var/www/wordpress/wp-config.php

- En el fichero de configuración añadimos el nombre de la base de datos creada, el user y la password

- Al final del fichero de configuración, incluimos el siguiente comando:

    define('FS_METHOD', 'direct');

- Desde la url, ejecutamos la instalación, según nuestro virtual host sería: wordpress.miservidor.com:puerto/wp-admin/install.php

- En la instalación añadir un titulo y un usuario para admin

- Ya podemos entrar en wordpress

    Consola admin: wordpress.miservidor.com:puerto/wp-admin
    Pagina test: wordpress.miservidor.com:puerto