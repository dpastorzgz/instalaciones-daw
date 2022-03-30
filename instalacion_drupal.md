
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

-   Entramos en mysql y creamos una base de datos para drupal 

    mysql -u root –p
    CREATE DATABASE drupal  DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
    
    Ahora creamos un usuario especial para drupal :

    CREATE USER 'drupaluser'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

-   Agregamos privilegios al usuario para acceder a la base de datos:
    
    GRANT ALL ON drupal.* TO 'drupaluser'@'%';

-   Para confirmar los cambios escribimos:
    
    FLUSH PRIVILEGES;

    Si queremos ver los usuarios creados y las bases de datos, los comandos son:

    SHOW DATABASES;
    select host, user, password from mysql.user;

- Creamos el archivo de configuración de host virtual

    Creamos carpeta para worpress mkdir /var/www/drupal

    Agregamos la propiedad del directorio con la variable de entorno $USER:
        sudo chown -R $USER:$USER /var/www/drupal

    Creamos el archivo de configuración para el sitio drupal:
        nano  /etc/apache2/sites-available/drupal.conf

        <VirtualHost *:80>
        ServerAdmin webmaster@drupal.miservidor.com
        DocumentRoot /var/www/drupal
        ServerName drupal.miservidor.com
        ServerAlias drupal.miservidor.com
        <Directory />
            Options FollowSymLinks
            AllowOverride None
        </Directory>
        <Directory /var/www/drupal>
            AllowOverride All
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

    Habilitamos la función mod_write y el nuevo host virtual:
        sudo a2enmod rewrite
        sudo a2ensite drupal
        sudo systemctl restart apache2

# Instalar drupal

- Descargamos la última version
    
    wget --content-disposition https://www.drupal.org/download-latest/tar.gz

- Extraemos el paquete en una carpeta temporal

    sudo tar xf drupal-9.3.9.tar.gz -C /tmp/drupal

- Movemos lo descargado a nuestra carpeta creada

    sudo cp -rp ./drupal-9.3.9/* /var/www/drupal

- Agregamos la propiedad a los usuarios y grupos de www-data:

    sudo chown -R www-data:www-data /var/www/drupal

- Continuamos con la instalación desde el navegador. (drupal.miservidor.com:puerto/core/install.php)

    En la primera pantalla debemos seleccionar el idioma
    En la segunda pantalla seleccionamos perfil estándar

    En la siguiente pantalla veremos una revisión de los requisitos necesarios para instalarlo. Podemos pinchar en "continuar de todos modos" o habilitar las recomendaciones que describe. En nuestro caso tuvimos que corregir el tema de las "URL limpias".
    Para configurarlo ahora, tendremos que asegurarnos que el Apache tiene el módulo "rewrite" habilitado. Si no lo tiene, lo podremos activar y reiniciar con:

        sudo a2enmod rewrite
        sudo service apache2 restart 


    También el virtualHost debe tener activado el AllowOverride
    También necesitaremos tener el fichero .htaccess en la raíz de /var/www/drupal para ello copiaremos de la carpeta de descarga de Drupal este fichero, si no lo tenemos copiado ya.
    Desde la carpeta donde está extraído drupal

        sudo cp .htaccess /var/www/drupal/

    En la siguiente pantalla nos mostrará la configuración de base de datos "drupal" para entrar en el panel de administración, así que tendremos que introducir el usuario "drupaluser" y la contraseña "password" que previamente hemos creado. Luego pincharemos en el botón "Guardar y continuar".

    Por último veremos la pantalla de configurar sitio.
    Veremos que tendremos que especificar el nombre del sitio web, y un email para las notificaciones. 
    Así como una cuenta de administrador, con su contraseña y correo. 
    Por último deberemos especificar el país y la zona horaria. 
    Pincharemos en el botón "Guardar y continuar".


