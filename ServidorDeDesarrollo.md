- [1. Servidor de Desarrollo](#1-servidor-de-desarrollo)
    - [1.1 Ubuntu Server 24.04.3 LTS](#11-ubuntu-server-24043-lts)
        - [Configuración](#configuración)
        - [Cuentas administradoras](#cuentas-administradoras)
        - [Servicios](#servicios)
    - [1.2 Apache HTTP](#12-Apache-HTTP)
        - [Instalación](#instalación)
        - [Cuentas](#cuentas)
        - [Conexión segura (HTTPS)](#protocolo-https)
    - [1.3 PHP](#13-php)
        - [Instalación](#instalacion)
        - [Configuración del interprete de PHP](#configuracion-del-interprete-de-php)
        - [Módulos de PHP](#modulos-de-php)
            - [MariaDB](#mariadb)
            - [XDebug](#xdebug)
            - [DNS](#dns)
            - [SFTP](#sftp)
            - [Apache Tomcat](#apache-tomcat)
            - [LDAP](#ldap)


## 1. Servidor de Desarrollo

### 1.1 Ubuntu Server 24.04.3 LTS

#### Configuración

**Configuración principal**
> **Nombre de la máquina**: alp-used\
> **Memoria RAM**: 2G\
> **Particiones**: 150G(/) y resto (350GB) (/var)\
> **Configuración de red interface**: xxxx \
> **Dirección IP** :10.199.11.90/22\
> **GW**: 10.199.8.90/22\
> **DNS**: 10.151.123.21 o 10.151.126.21

Para comprobar todos estos valores debemos usar los siguientes comandos:
```bash
hostname
sudo hostnamectl                                # Comprobar el nombre, el sistema operativo, la arquitectura, etc...
sudo hostnamectl set-hostname "nombre"          # Cambiar el nombre de la máquina.
sudo nano /etc/hosts                            # Modificar las siguientes lineas de este archivo.

127.0.0.1 localhost
127.0.1.1 alp_used2                             # En esta línea modificamos el nombre por el nuevo.

free -h                                         # Comprobar la RAM total, en uso y libre. Parámetro -h para que salga  en Gb

lsblk
sudo fdisk -l /dev/sda                          # Comprobar las distintas particiones del disco, su tamaño y su raíz

ip a
ip r                                            # Comprobar IP y GW como se ve en las capturas de abajo

sudo resolvectl status                          # Comprobar el DNS (educa.jcyl.es)

date                                            # Comprobar la fecha y hora
```

**Configuración de red**
Editar el fichero de configuración del interface de red  **/etc/netplan**.
En este caso los datos introducidos son los mios personales pero cada uno 
puede configurarlo acorde con sus preferencias.

```bash

# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      addresses:
       - 10.199.11.90/22
      nameservers:
       - 10.151.123.21
       - 10.151.126.21
       search: [educa.jcyl.es]
      routes:
       - to: default
       via: 10.199.8.1
  version: 2
````


**Actualizar el sistema**
```bash
sudo apt update
sudo apt upgrade
```


**Configuración fecha y hora**

Para comprobar la fecha y la hora, y modificarlas, usaremos los siguientes comandos: 
```bash
# Comprobar la hora y fechar en formato: día YYYY-MM-DD HH:MM:SS 
# También muestra la zona horaría en la que está trabajando el sistema.
timedatectl

# Para modificar la zona horaria debemos usar el siguiente comando:
sudo timedatectl set-timezone Europe/Madrid         # En caso de querer poner la zona horaria de Madrid UTG+1.
```

#### Cuentas administradoras 

**Cuentas administradoras**

> root(inicio)\
> miadmin/paso\
> miadmin2/paso

#### Servicios 

**SSH**\
El protocolo de red SSH permite controlar y modificar servidores remotos de manera segura a través de Internet. Utiliza criptografía para
encriptar las conexiones entre dispositivos, garantizando así la seguridad de los datos transmitidos.

En el momento que estamos instalando el sistema operativo Ubuntu Server 24.04.3 LTS el instalador nos pregunta si queremos el servicio SSH.
Por tanto tenemos dos opciones para obtener el servicio SSH en nuestra máquina:

**1ª Opción:** preinstalarlo durante la instalación del sistema operativo.

**2ª Opción:** usar el siguiente comando:
```bash
# Actualizamos el sistema operativo.
sudo apt update

# Instalamos el servicio SSH.
sudo apt install openssh-server -y
```

En cualquiera de los dos casos debemos de comprobar que se ha instalado el servicio correctamente:
```bash
sudo systemctl status ssh

# En caso de necesitar iniciarlo o reiniciarlo tenemos este comando
sudo systemctl [start|restart] ssh

# Para que el servicio se inicie cada vez que se encienda la máquina tenemos este comando
sudo systemctl enable ssh
```

Conexión SSH
Para conectarnos con la máquina con SSH deberemos abrir la consola de comandos en un dispositivo de la misma red
y escribir el siguiente comando:
```bash
ssh 'usuario'@'ip'
```

Nos pedirá la contraseña del usuario y nos iniciará sesión. En ese momento podremos gestionar en remoto desde esa consola.
Hay que tener en cuenta que los límites de que se puede hacer los marcan los privilegios que tenga el usuario conectado.

**UFW**\
Para comprobar el estado del cortafuegos y saber si está activado o desactivado, debemos usar el siguiente comando:

```bash
sudo systemctl status ufw                           # Mostrar el status del cortafuegos.
sudo systemctl start|restart ufw                    # Arrancar el cortafuegos.
```

De la misma manera podemos comprobar el SSH:

```bash
sudo systemctl status ssh                           # Mostrar el status del servicio SSH.
sudo systemctl start|restart ssh                    # Arrancar el servicio SSH.

# A mayores tenemos que comprobar el puerto del cortafuegos ufw 22.
sudo ufw status numbered

# Normalmente va a estar activo el puerto 22 y el puerto 22 v6. Éste último hay que borrarlo.
sudo ufw delete "numeropuerto"
```

### 1.2 Apache HTTP

#### Instalación

Para instalar un servidor web vamos a descargar y configurar Apache. El primer paso es actualizar el SO y 
también los paquetes instalados. A continuación instalamos Apache2 y abrimos el puerto 80 que es el utilizados 
por Apache. Al abrir el puerto se abrirá tanto el normal como el (v6) y, por recomendación de seguridad, lo borraremos.

```bash

sudo apt update                         #Actualizamos OS
sudo apt upgrade                        #Actualizamos paquetes instalados

sudo apt install apache2                #Instalamos Apache2 en la máquina
  
sudo ufw allow 80                       #Habilitamos el puerto 80 desde el cortafuegos.

sudo ufw status numbered                
sudo ufw delete 'numeropuerto'          #Borramos el puerto 80 (v6) usando su número de indetificación

```
**Verficación del servicio**

Para verificar que Apache se ha instalado y está funcionando correctamente tenemos dos formas: entrando a un 
navegador desde el ordenador anfitrión y buscando la página con al dirección IP de la máquina. Si aparece una 
página como la de abajo es que Apache esta funcionando correctamente y que ambas máquinas están en la misma red. 
La otra forma es mediante los siguientes comandos:

```bash

sudo service apache2 {opcion}
sudo systemctl {opcion} apache2

```

#### Cuentas
Al crear la máquina virtual creamos al usuario miadmin con contraseña paso y con privilegios de sudo. Una vez configurada la red y verificado el servicio, creamos el usuario miadmin2:

```bash
sudo adduser miadmin2                   #Creamos el usuario con contraseña paso e ignoramos el resto de datos que nos piden
```

Una vez creado el usuario tenemos que darle privilegios de sudo, es decir, meterle en el grupo sudoers para que pueda hacer ciertos comandos:

```bash
sudo usermod -aG sudo miadmin2          
# Meter al usuario miadmin2 en el grupo sudo sin quitarle del resto de grupos que pertenece y -G indica los grupos suplementarios a los que quieres añadir el usuario.
# Ahora lo que tenemos que hacer es el meter a miadmin2 en los mismos grupos que miadmin usando este comando y listando
  con cat /etc/group | grep miadmin

**Comandos recomendados**
sudo deluser "nombreusuario"            # Borra el usuario indicado.
su nombreusuario                        # Inicia sesión en el usuario indicado.
exit                                    # Cierra la sesión actual.
```


Ahora vamos a crear el usuario "operadoweb" que será el encargado de subir archivos al servidor. Solo podrá tener acceso a su carpeta ráiz
que es /var/www/html y pertenecerá al grupo www-data

```bash
sudo adduser --home /var/www/html --ingroup www-data --shell /bin/bash operadorweb

# Ahora debemos cambiar el dueño de la carpeta /var/www/html para que pertenezca a operadorweb
sudo chown -R operadorweb:www-data /var/www/html            # Cambia el propietario y el grupo del directorio indicado.
sudo chmod -R 775 /var/www/html                             # Cambia los permisos del usuario propietario, del grupo y del resto de propieatarios.

# Para comprobar que los cambios han surgido efecto debemos hacer lo siguiente:
ls -al /var/www/html

drwxrwxr-x 2 operadorweb www-data  4096 oct  9 10:30 .
drwxr-xr-x 3 root        root      4096 oct  9 10:30 ..
-rwxrwxr-x 1 operadorweb www-data 10671 oct  9 10:30 index.html

```

**Usuarios enjaulados**
El concepto de usuario enjaulado tiene que ver con la seguridad de nuestro servidor web. Cuando "enjaulamos" a un usuario, estamos prohibiendole circular por el árbol de directorios
de nuestro servidor, es decir, solo puede entrar, modificar, leer y borrar en cualquier fichero o directorio dentro de su directorio raíz.

El primer paso es crear un grupo llamado "sftpusers" (el nombre es a gusto del desarrollador) en el cual vamos a introducir a los usuarios que queremos enjaular.
También crearemos un usuario de prueba para comprobar que este método funciona y es seguro.
```bash
# Creamos el grupo.
sudo groupadd sftpusers

# Creamos el usuario con raíz /var/www/nombredeusuario y que pertenezca al grupo creado.
sudo useradd -g www-data -G sftpusers -m -d /var/www/enjaulado1 enjaulado1

# Cambiamos la contraseña del usuario creado ya que no tiene.
sudo passwd enjaulado1
```

Ahora debemos cambiar los permisos del directorio jaula y de los directorios padres de éste.
```bash
# Cambiamos el dueño del directorio /var/www/enjaulado1
sudo chown root:root /var/www/enjaulado1

# Quitamos el permiso de escritura del directorio /var/www/enjaulado1
sudo chmod 555 /var/www/enjaulado1
```

Para terminar, debemos crear la carpeta donde vamos a subir nuestros proyectos y aplicaciones:
```bash
# Creamos la carpeta httpdocs.
sudo mkdir /var/www/enjaulado1/httpdocs

# Le damos permisos de lectura y escritura a todos y de ejeución a root.
sudo chmod 2775 -R /var/www/enjaulado1/httpdocs

# Cambiamos el propietario del directorio /var/www/enjaulado1/httpdocs para que sea el usuario enjaulado1.
sudo chown enjaulado1:www-data -R /var/www/enjaulado1/httpdocs
```

Por último editamos el archivo de configuración /etc/ssh/sshd_config:
```bash
# Realizamos una copia del archivo de configuración por si surgen problemas.
sudo cp sshd_config sshd_config.bk

# Introducimos las siguientes lineas.
sudo nano sshd_config

# Buscamos la siguiente línea, la comentamos y a continuación copiamos estas líneas.

# Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp

Match Group sftpusers
ChrootDirectory %h
ForceCommand internal-sftp -u 2
AllowTcpForwarding yes
PermitTunnel no
X11Forwarding no

# Guardamos el archivo y reiniciamos el servicio ssh
sudo systemctl restart ssh
```

Para comprobar nos iremos al MobaXterm y comprobaremos que, con el usuario enjaulado1 se inicia en el directorio /var/www/enjaulado1 y que no puede acceder al directorio padre.
También podemos comprobar que en la carpeta /var/www/enjaulado1/httpdocs se pueden crear y eliminar archivos.
### Protocolo HTTPS
Es la versión segura del protocolo HTTP, el cual cifra la comunicación entre el navegador y el servidor, 
garantizando confidencialidad, integridad y autenticación de los datos.
#### Instalación
Generar clave privada SSL
```bash
# Generamos la solicitud de certificado.
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/alp-used.key -out /etc/ssl/certs/alp-used.crt

# A continuación vamos a introducir los datos del certificado:
Country Name: ES
State or Province Name: Zamora
Locality Name: Benavente
Organization Name: Instituto
Organizational Unit Name: Informatica
Common Name: alp-used
Email Address: alvaro.allper.1@educa.jcyl.es

# Para comprobar que se ha creado correctamente.
sudo ls /etc/ssl/certs | grep alp-used
sudo ls /etc/ssl/private | grep alp-used

# Reiniciamos el servicio apache
sudo systemctl restart apache2

# Nos situamos en el directorio /etc/apache2/sites-available y hacemos una copia del archivo default-ssl.conf
cd /etc/apache2/sites-available
sudo cp default-ssl.conf alp-used.conf

# Modificamos el archivo copiado alp-used.conf con las siguientes líneas:

SSLCertificateFile      /etc/ssl/certs/alp-used.crt
SSLCertificateKeyFile   /etc/ssl/private/alp-used.key

# Activamos el archivo de configuración modificado y reinicamos el servicio apache.
sudo a2ensite alp-used.conf
sudo systemctl restart apache2

# También debemos activar el puerto 443 en el firewall y borrar el v6.
sudo ufw allow 443
sudo ufw status numbered # Comprobar el número del puerto 443(v6)
sudo ufw delete (nº)
```

**Consejo**
Al entrar con un navegador al servidor, nos indica que no es seguro debido a que la autoridad certificadora no es segura.
El certificado es autofirmado por el usuario, en este caso yo, y no es de confianza para el navegador. Esto se puede se puede resolver
de dos formas.
> 1. Pedir a una autoridad certificadora que firme el certificado para que así sea reconocido por el navegador y sea seguro.\
> 2. Introducir como autoridad certificadora a tu propio usuario en tu dispositivo. No supone un riesgo ya que te estás dando confianza
> a ti mismo.

### 1.3 PHP

#### Instalación
Una vez actualizado el sistema y mejorado los paquetes (update y upgrade) debemos de realizar los siguientes pasos:
```bash
# Comprobamos que apache está instalado y activo.
sudo systectl status apache2

# A continuación instalamos PHP y PHP8.3-FPM con la versión 8.3
sudo apt install php8.3-fpm php8.3

# Y por último instalamos el módulo de libapache2-mod-php8.3 y lo habilitamos.
sudo apt install libapache2-mod-php8.3 -y
sudo a2enmod php8.3
```

Para terminar reiniciaremos Apache y comprobaremos que todo ha salido bien:
```bash
# Reiniciamos servicio Apache
sudo systemctl restart apache2
```

En caso de no haber salido error al reiniciar, creamos en nuestro directorio ráiz del servidor un info.php y le introducimos lo siguiente:
```bash
<?php
phpinfo();
?>
```
Entramos en nuestra página principal y escribimos en la barra de busqueda:
http://direccionip/info.php. Nos debería de salir una página como esta.

**Directorios y ficheros de PHP**

**/etc/php/8.3/fpm/conf.d**: Módulos instalados en esta configuración de php (enlaces simbólicos a /etc/php/8.3/mods-available)
**/etc/php/8.3/fpm/php-fpm.conf** : Configuración general de php-fpm
**/etc/php/8.3/fpm/php.ini** : Configuración de php para este escenario
**/etc/php/8.3/fpm/pool.d** : Directorio con distintos pool de configuración. Cada aplicación puede tener una configuración distinta (procesos distintos) de php-fpm.
  
Por defecto tenemos un pool cuya configuración la encontramos en **/etc/php/8.3/fpm/pool.d/ www.conf**, en este fichero podemos configurar parámetros, los más importantes son:

**[www]**: -es el nombre del pool, si tenemos varios, cada uno tiene que tener un nombre.
**user y group** : Usuario y grupo con el que va a ejecutar los procesos
**listen**: Se indica el socket unix o el socket TCP donde se van a escuchar los procesos:
Por defecto, escucha por un socket unix: listen=/run/php/php8.3-fpm.sock
Si queremos que escuche por TCO; listen=127.0.0.1:9000
Directivas de procesamiento, gestión de procesos:
**pm**: Por defecto es igual a dynamic (el número de procesos se crean y se destruyen de forma dinámica). Otros valores: static o ondemand.
Otras directivas: **pm.max_children** (número máxio de procesos hijo que pueden ser creados al mismo tiempo para manejar solicitudes), **pm.start_servers** (cuantos procesos PHP-FPM se lanzararón al inicio de forma automática),**pm.min_spare_servers**( número mínimo de procesos del servidor inactivos para manejar nuevas solicitudes),...
**pm.status_path=/status**: No es necesario, vamos a activar la URL de status para comprobar el estado del proceso.

#### Configuración del interprete de PHP
La configuración que vamos a aplicar en nuestro servicio de PHP es el siguiente:
> display_errors: On
> display_startup: On
> memory_limit: 256M

El display_errors sirve para mostrar los errores de ejecución en la salida (navegador
o consola) y el display_startup_errors para controlar los errores que surgen durante la
ejecución PHP.

El límite de memoria o memory_limit en el archivo de configuración sirve para establecer
un límite máximo de memoria que puede usar un archivo PHP.

Una vez explicado vamos a ir paso por paso detallando como aplicar esta configuración:

Una vez instalado PHP y comprobado que nos funciona el info.php entramos en el directorio
/etc/php/8.3/fpm donde encontraremos el archivo de configuración php.ini.
```bash
cd /etc/php/8.3/fpm

# Listamos el contenido y encontramos el archivo de configuración php.ini. Lo copiamos.
sudo cp php.ini php.ini.backup

#Modificamos el archivo php.ini cambiando los valores de display_errors, display_startup_errors
y memory_limit con los valores de On, On y 256M. Puedes user ctrl+w para buscar palabras en 
el editor.
sudo nano php.ini

#Una vez modificado y guardado, reiniciamos el servicio PHP.
sudo systemctl restart php8.3-fpm
```

Para comprobar que dicha configuración se ha aplicado vamos a la página info.php y comprobamos
que dichos valores han cambiado y son los introducidos en el archivo de configuración.

> **Consejo:**\
> En el info.php hay un apartado llamado "Loaded configuration File" que indica la ruta donde se encuentra el archivo que acabamos de modificar.

#### Módulos de PHP

##### MariaDB
El gestor de base de datos que hemos escogido, compatible con PHP, es MariaDB.
A continuación detallaremos el proceso de instalación y puesta en marcha de este servicio.

**Instalación**

```bash
# Actualizamos el OS
sudo apt update

# Instalamos MariaDB
sudo apt install mariadb-server -y

# Una vez instalado comprobamos la version de MariaDB instalada.
mariadb --version

mariadb Ver 15.1 Distrib 10.11.13
```
**Configuración**
Una vez instalado el servicio de MariaDB nos vamos a la configuración de esta misma.

```bash
# Nos dirigimos a este archivo y lo modificamos /etc/mysql/mariadb.conf.d/50-server.cnf
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

# Modificamos bind-address = 127.0.0.1 por: 0.0.0.0 y guardamos
# Reiniciamos el servicio
sudo systemctl restart mariadb

# Aunque no nos de error al reiniciar debemos hacer una comprobación para estar seguros de que el proceso está en ejecución.
sudo ss -punta | grep mariadb

# Nos debe salir una linea así: tcp LISTEN 0 80 0.0.0.0:3306 0.0.0.0:* users:(("mariadb",pid=7865,fd=22))
```

Con esto ya tenemos un servidor sql trabajando correctamente en nuestra máquina ubuntu server.
Para iniciar el servicio en modo comando, nos iniciará sin autenticar ningún usuario. 

**Usuarios requeridos**
Superusuarios: root y adminsql con los cuales crearemos, modificaremos y eliminaremos tanto bases de datos como usuarios.
Usuario base: uno por cada base de datos el cual va a realizar las consultas necesarias a la base y enviarselas al PHP. Este usuario no tendrá ciertos privilegios por seguridad.
**-------------------**

Otro paso importante es realizar la prueba de instalación segura de mariadb
```bash
sudo mysql_secure_installation
```
Y realizamos los pasos siguientes de la siguiente manera:

**Instalación de módulos**
Todo esto se puede realizar previo a instalar el mysql ya que esto es una configuración de PHP.
Modelo php8.3-mysql es la extensión que permite a PHP conectarse con el servidor de bases de datos.
```bash
sudo apt install php8.3-mysql
sudo apt install php8.3-intl           # Extensión de internalización básica para SQL.
```
Ahora se listan los módulos mediante el siguiente comando (este paso se debe realizar previo a la instalación y después):
```bash
sudo php -m | grep mysql
# Si se realiza antes de instalar los módulos debería de aparecer vacio.
# En caso de hacerlo después apareceran varios módulos: mysqli, mysqlnd, pdo_mysql.
``` 
**----------------------**

**Conexión con NetBeans**
Para conectar el NetBeans a la base de datos debemos de descargar el driver necesario:
mariadb-java-cliente-3.5.6.

Una vez descargado y guardado en una carpeta llamada lib entraremos en NetBeans e iremos al apartado 'Services'>'Databases'
Haremos click derecho donde pone 'MariaDB(MySQL-compatible)' y se nos abrirá una pestaña de conexión.
En esta pestaña añadiremos el driver descargado y continuaremos a efectuar la conexión.
En cada apartado pondremos lo siguiente:
**Host:** 10.199.11.90 (ip de la máquina)
**Port:** 3306
**Database:** se puede dejar vacio o introducir uno cual sea.
**User Name:** adminsql
**Password:** paso

Antes de continuar con la conexión podemos testearla para confirmar que funciona.
Por último decidimos el nombre de la conexión para facilitarnos su uso y aceptamos.
Ya podemos realizar todas las operaciones que el usuario introducido puede hacer con los permisos que tenga.
##### XDebug
**Verifica si Xdebug está instalado**

```bash
sudo php -v | grep xdebug
```

**Si no aparece, instalálo:**
```bash
sudo apt install php8.3-xdebug
```

Luego se edita el fichero de configuración:

```bash
sudo nano /etc/php/8.3/fpm/conf.d/20-xdebug.ini
```

Y añade

```bash
xdebug.mode=develop,debug
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
xdebug.log=/tmp/xdebug.log
xdebug.log_level=7
xdebug.idekey="netbeans-xdebug"
xdebug.discover_client_host=1
```

Guarda y reinicia el servidor

```bash
sudo systemctl restart apache2
# o si usas php-fpm
sudo systemctl restart php8.3-fpm
```
##### DNS
##### SFTP
##### Apache Tomcat
##### LDAP