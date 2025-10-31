
# CFGS Desarrollo de Aplicaciones Web


- [CFGS Desarrollo de Aplicaciones Web](#cfgs-desarrollo-de-aplicaciones-web)
  - [1. Entorno de Desarrollo](#1-entorno-de-desarrollo)
    - [1.1 Ubuntu Server 24.04.3 LTS](#11-ubuntu-server-24043-lts)
      - [1.1.1 **Configuración inicial**](#111-configuración-inicial)
        - [Nombre y configuración de red](#nombre-y-configuración-de-red)
        - [**Actualizar el sistema**](#actualizar-el-sistema)
        - [**Configuración fecha y hora**](#configuración-fecha-y-hora)
        - [**Cuentas administradoras**](#cuentas-administradoras)
        - [**Gestión SSH**](#gestion-ssh)
        - [**Gestion UFW**](#Gestión-UFW)
      - [1.1.2 Apache HTTP](#112-Apache-HTTP)
        - [Instalación](#instalación)
        - [Verficación del servicio](#verficación-del-servicio)
        - [Virtual Hosts](#virtual-hosts)
        - [Permisos y usuarios](#permisos-y-usuarios)
        - [Conexión segura (HTTPS)](#protocolo-https)
      - [1.1.3 PHP](#113-php)
        - [Instalación de PHP en el servidor apache](#instalación-de-php-en-el-servidor-apache)
      - [1.1.4 MariaDB](#114-mariadb)
      - [1.1.5 XDebug](#115-xdebug)
      - [1.1.6 DNS](#116-dns)
      - [1.1.7 SFTP](#117-sftp)
      - [1.1.8 Apache Tomcat](#118-apache-tomcat)
      - [1.1.9 LDAP](#119-ldap)
    - [1.2 Windows 10](#12-windows-10)
      - [1.2.1 **Configuración inicial**](#121-configuración-inicial)
        - [**Nombre y configuración de red**](#nombre-y-configuración-de-red-1)
        - [**Cuentas administradoras**](#cuentas-administradoras-1)
      - [1.2.2 **Navegadores**](#122-navegadores)
      - [1.2.3 **MobaXterm**](#123-MobaXterm)
      - [1.2.4 **Netbeans**](#124-netbeans)
      - [1.2.5 **Visual Studio Code**](#125-visual-studio-code)
  - [2. GitHub](#2-github)
    - [2.1 Publicación de un repositorio local](#publicación-de-un-repositorio-local)
    - [2.2 Clonar un repositorio](#clonar-un-repositorio)
  - [3.Entorno de Explotación](#3entorno-de-explotación)

|  DAW/DWES Tema2 |
|:-----------:|
|![Alt](images/portada.jpg)|
| INSTALACIÓN, CONFIGURACIÓN Y DOCUMENTACIÓN DE ENTORNO DE DESARROLLO Y DEL ENTORNO DE EXPLOTACIÓN |

## 1. Entorno de Desarrollo

### 1.1 Ubuntu Server 24.04.3 LTS

Este documento es una guía detallada del proceso de instalación y configuración de un servidor de aplicaciones en Ubuntu Server utilizando Apache, con soporte PHP y MySQL

#### 1.1.1 **Configuración inicial**

##### Nombre y configuración de red

> **Nombre de la máquina**: alp-used\
> **Memoria RAM**: 2G\
> **Particiones**: 150G(/) y resto (350GB) (/var)\
> **Configuración de red interface**: xxxx \
> **Dirección IP** :10.199.11.90/22\
> **GW**: 10.199.8.90/22\
> **DNS**: 10.151.123.21\ o 10.151.126.21\

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


### **Actualizar el sistema**
```bash
sudo apt update
sudo apt upgrade
```
#### **Configuración fecha y hora**

[Establecer fecha, hora y zona horaria](https://somebooks.es/establecer-la-fecha-hora-y-zona-horaria-en-la-terminal-de-ubuntu-20-04-lts/ "Cambiar fecha y hora")

Para comprobar la fecha y la hora, y modificarlas, usaremos los siguientes comandos: 
```bash
# Comprobar la hora y fechar en formato: día YYYY-MM-DD HH:MM:SS 
# También muestra la zona horaría en la que está trabajando el sistema.
timedatectl

# Para modificar la zona horaria debemos usar el siguiente comando:
sudo timedatectl set-timezone Europe/Madrid         # En caso de querer poner la zona horaria de Madrid UTG+1.
```

#### **Cuentas administradoras**

> - [X] root(inicio)
> - [X] miadmin/paso
> - [X] miadmin2/paso

#### **Gestión SSH**
El protocolo de red SSH permite controlar y modificar servidores remotos de manera segura a través de Internet. Utiliza criptografía para
encriptar las conexiones entre dispositivos, garantizando así la seguridad de los datos transmitidos.

##### Instalación de SSH
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
##### Conexión SSH
Para conectarnos con la máquina con SSH deberemos abrir la consola de comandos en un dispositivo de la misma red
y escribir el siguiente comando:
```bash
ssh 'usuario'@'ip'
```

Nos pedirá la contraseña del usuario y nos iniciará sesión. En ese momento podremos gestionar en remoto desde esa consola.
Hay que tener en cuenta que los límites de que se puede hacer los marcan los privilegios que tenga el usuario conectado.

#### **Gestión UFW**
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

### 1.1.2 Apache HTTP

#### Instalación

Para instalar un servidor web vamos a descargar y configurar Apache. El primer paso es actualizar el SO y también los paquetes instalados. A continuación instalamos Apache2 y abrimos el puerto 80 que es el utilizados por Apache. Al abrir el puerto se abrirá tanto el normal como el (v6) y, por recomendación de seguridad, lo borraremos.

```bash

sudo apt update                         #Actualizamos OS
sudo apt upgrade                        #Actualizamos paquetes instalados

sudo apt install apache2                #Instalamos Apache2 en la máquina
  
sudo ufw allow 80                       #Habilitamos el puerto 80 desde el cortafuegos.

sudo ufw status numbered                
sudo ufw delete 'numeropuerto'          #Borramos el puerto 80 (v6) usando su número de indetificación

```
#### Verficación del servicio

Para verificar que Apache se ha instalado y está funcionando correctamente tenemos dos formas: entrando a un navegador desde el ordenador anfitrión y buscando la página con al dirección IP de la máquina. Si aparece una página como la de abajo es que Apache esta funcionando correctamente y que ambas máquinas están en la misma red. La otra forma es mediante los siguientes comandos:

```bash

sudo service apache2 {opcion}
sudo systemctl {opcion} apache2

```
#### Virtual Hosts
##### Permisos y usuarios
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

#### Protocolo HTTPS
Es la versión segura del protocolo HTTP, el cual cifra la comunicación entre el navegador y el servidor, 
garantizando confidencialidad, integridad y autenticación de los datos.
##### Instalación
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
> 1. Pedir a una autoridad certificadora que firme el certificado para que así sea reconocido por el navegador y sea seguro.

> 2. Introducir como autoridad certificadora a tu propio usuario en tu dispositivo. No supone un riesgo ya que te estás dando confianza
> a ti mismo. 
### 1.1.3 PHP
#### Instalación de PHP en el servidor apache
Una vez actualizado el sistema y mejorado los paquetes (update y upgrade) debemos de realizar los siguientes pasos:
```bash
# Comprobamos que apache está instalado y activo.
sudo systectl status apache2

# A continuación instalamos PHP y PHP8.3-FPM con la versión 8.3
sudo apt install php8.3-fpm php8.3
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
**/etc/php/8.3/fpm/php.ini** : Configuraicón de php para este escenario
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

#### Configuración del intérprete PHP.
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

> **Consejo:**
>
> En el info.php hay un apartado llamado "Loaded configuration File" que indica la ruta donde
> se encuentra el archivo que acabamos de modificar.

### 1.1.4 MariaDB
El gestor de base de datos que hemos escogido, compatible con PHP, es MariaDB.
A continuación detallaremos el proceso de instalación y puesta en marcha de este servicio.

#### Instalación

```bash
# Actualizamos el OS
sudo apt update

# Instalamos MariaDB
sudo apt install mariadb-server -y

# Una vez instalado comprobamos la version de MariaDB instalada.
mariadb --version

mariadb Ver 15.1 Distrib 10.11.13
```
#### Configuración
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



#### Usuarios

**Usuarios requeridos**
Superusuarios: root y adminsql con los cuales crearemos, modificaremos y eliminaremos tanto bases de datos como usuarios.
Usuario base: uno por cada base de datos el cual va a realizar las consultas necesarias a la base y enviarselas al PHP. Este usuario no tendrá ciertos privilegios por seguridad.
**-------------------**

Otro paso importante es realizar la prueba de instalación segura de mariadb
```bash
sudo mysql_secure_installation
```
Y realizamos los pasos siguientes de la siguiente manera:


#### Módulos
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

#### Conexión con NetBeans
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
### 1.1.5 XDebug
##### ⚙️ Instalación y configuración

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
### 1.1.6 DNS
### 1.1.7 SFTP
### 1.1.8 Apache Tomcat
### 1.1.9 LDAP

## 1.2 Windows 10
### 1.2.1 **Configuración inicial**
#### **Nombre y configuración de red**
#### **Cuentas administradoras**
### 1.2.2 **Navegadores**
Aquí se especifican los navegadores que solemos utilizar para la interpretación y visualización de
nuestras aplicaciones web. También se indican las extensiones instaladas en cada uno.

> **Navegadores y extensiones**
>
> Edge: Color Picker - Native Eyedropper
>
> Chrome: ColorZilla
### 1.2.3 **MobaXterm**
Esta aplicación permite conectarse a un servidor mediante un amplio abanico de protocolos.
En nuestro caso, los dos protocolos que vamos a utilizar son SFTP para la gestión de archivos y 
directorios del servidor apache y SSH para realizar cambios en los ficheros de configuración.

> **Versión**: MobaXterm Personal Edition v25.2 Build 5296

Una vez descargada e instalada la versión siguiendo los pasos (en clase tenemos la versión portable),
iniciamos la aplicación.
Para conectar un dispositivo por SSH debemos realizar los siguientes pasos:
> Pulsamos el botón Session con una pantalla como símbolo.

|![Alt](webroot/images/mobasession.PNG)|

> Seleccionamos SSH como protocolo de conexión.

|![Alt](webroot/images/mobassh.PNG)|

> Introducimos en el apartado Remote host la IP del ordenador al que queremos conectarnos. También 
se puede especificar al usuario al que queremos conectarnos en el apartado Specify username, pero no 
es obligatorio.

|![Alt](webroot/images/mobassh.PNG)|

> Se nos abrirá un terminal donde iniciaremos sesión con un usuario y una contraseña valida.

|![Alt](webroot/images/mobasshdisplay.PNG)|
Para conectar un dispositovo por SFTP para la transferencia de datos debemos realizar los siguientes pasos:

> Pulsamos el botón Session con una pantalla como símbolo.

|![Alt](webroot/images/mobasession.PNG)|

> Seleccionamos SFTP como protocolo de conexión

|![Alt](webroot/images/mobasftp.PNG)|

> Introducimos la IP del ordenador al que queremos conectarnos en el apartado "Remote hosts" y, esta vez,
si debemos especificar el usuario al que queremos conectarnos en el apartado "Username". Para nosotros es "operadorweb".

|![Alt](webroot/images/mobasftpdisplay.PNG)|

> Comprobamos que nos encontramos en el directorio raiz /var/www/html y que podemos crear carpetas y archivos, modificarlos y eliminarlos.
### 1.2.4 **Netbeans**
Como nuestro IDE escogido para el desarrollo de aplicaciones web durante el segundo curso de DAW es NetBeans
vamos a introducir este programa.
De uso completamente gratuito y con una interfaz compleja pero bastante completa tenemos delante un IDE estupendo pero complicado 
en un principio.

#### Creación de proyecto
En este apartado vamos a detallar como se crea un proyecto de PHP enlazado mediante SFTP a un servidor de Ubuntu. La configuración
de dicho servidor está explicada en este mismo documento.
Para crear debemos de pulsar el botón con un cuadrado amarillo y un + verde.

|![Alt](webroot/images/proyectonuevo.PNG)|

Esto nos abrirá una pestaña como la de abajo donde podemos escoger el tipo de proyecto entre un catálogo extenso:

|![Alt](webroot/images/proyectonuevo1.PNG)|

Escogemos el proyecto PHP y se nos abre la ventana de configuración de conexión donde pondremos la url completa de nuestro proyecto y 
donde se guardará. Antes de continuar deberemos de darle a Manage y establecer la conexión

|![Alt](webroot/images/proyectonuevo2.PNG)|

**Pestaña de conexión**
En esta pestaña indicaremos la ip de nuestro servidor Ubuntu, el usuario con el que iniciaremos la conexión y su contraseña que en este caso
es el usuario operadorweb e indicaremos la ruta donde se encuentra el proyecto en cuestion teniendo en cuenta que los proyectos se alojan en
/var/www/html...

|![Alt](webroot/images/proyectoconexion.PNG)|

Para terminar pulsaremos el botón de Test Connection para probar si la conexión se realiza correctamente y cerraremos la pestaña.

|![Alt](webroot/images/proyectoconexion.PNG)|

Al terminar de configurar la conexión y en caso de haber salido bien, nos saldrá un cuadro donde nos muestra la carpeta del proyecto previamente
creada en el servidor y dentro un archivo cualquiera también creado con anterioridad. Esto es muy importante ya que, en caso de no existir la carpeta
o de estar vacía, la conexión nos lleva a errores como creación de archivos basura o creación de directorios donde no deberían de crearse.

|![Alt](webroot/images/proyectonuevo3.PNG)|
|![Alt](webroot/images/proyectonuevo4.PNG)|

**Pestaña de sincronización**
Al finalizar podremos gestionar nuestra conexión pulsando click derecho en la carpeta sources y dandole a syncronize. Importante tener cuidado ya que 
la pestaña de sincronización nos permite borrar, descargar y subir archivos y NetBeans establece bajo su criterio que elementos borrar, cuales subir y cuales 
descargar y suele confundirse. Tener máximo cuidado y, en caso de duda, poner a todos los archivos la opción suspense la cual ignora el archivo.

|![Alt](webroot/images/proyectosincro.PNG)|

#### Versionamiento de un proyecto
Una vez creado el proyecto, lo siguiente es versionar y hacer el primer commit. Para ello haremos click derecho encima del proyecto y pulsaremos 
Versioning>Initialize Repository. Automaticamente se nos pondrán en verde todos los archivos y tendremos que hacer un commit.

|![Alt](webroot/images/git.PNG)|

|![Alt](webroot/images/menugit.PNG)|

Por defecto la rama creada es la master y con ella haremos el commit pulsando botón derecho sobre el proyecto Git>Commit y nos saldrá la siguiente ventana:

|![Alt](webroot/images/menugit.PNG)|
|![Alt](webroot/images/gitcommit.PNG)|

El mensaje recomendado para el primer commit es: Commit inicial o Initial commit. 

|![Alt](webroot/images/gitcommit.PNG)|

Con esto podemos dar por terminado este apartado pero podemos añadir ramas cuyo uso facilita y simplifica el desarrollo.
Tener una rama developer donde desarrolles y pruebes cosas nuevas y otra master donde subas el producto terminado hace más limpio
el desarrollo de tu aplicación.
Para crear una rama debemos hacer click derecho en el proyecto Git>Branch/Tag>Create Branch. Se nos abrirá una ventana
en la cual escribiremos el nombre que queremos para la rama y aceptaremos.

|![Alt](webroot/images/gitbranch.PNG)|
|![Alt](webroot/images/gitdeveloper.PNG)|

# 2. GitHub
Una vez versionado el proyecto y configurado de la forma correcta para su desarrollo, podemos dar un paso más y llevar nuestro repositorio a GitHub.
GitHub es una red social donde usuarios de todo el mundo publican sus proyectos de manera pública o privada. Así cualquier usuario puede ver el código, preguntar
hacer propuestas de cambios y mejoras, descargar el proyecto y probarlo en casa, incluso se puede clonar el repositorio para tenerlo en otro dispositivo. 
Esta última cualidad es de la que va a tratar este apartado.

## Publicación de un repositorio local
Para realizar este paso debemos de tener ciertos pasos previos:
> Crear una cuenta de GitHub
> Crear un repositorio (público o privado)
> Crear un token clásico para realizar las subidas y bajadas de contenido a parte de para realizar la clonación.

**Creación de token**
Como este paso es más complejo que la creación de la cuenta de GitHub y el repositorio, vamos a explicarlo detalladamente.
Desde la pagina de inicio de GitHub pulsamos el circulo con nuestra foto de perfil situado en la esquina derecha superior de la pestaña y entramos en 'Settings'
Una vez dentro vamos al apartado 'Developer settings' situado al final de la lista lateral.
Una vez dentro vamos a 'Personal access tokens'>'Tokens (classic)'

|![Alt](webroot/images/githubtoken.PNG)|

Es importante guardar el código en un lugar seguro ya que una vez creado no se podrá ver.

**Conexión de un repositorio desde NetBeans**
Haciendo click derecho en el proyecto nos dirigimos a 'Git'>'Remote'>'Push'

|![Alt](webroot/images/githubpush.PNG)|

Dentro de la pestaña indicamos la url del repositorio en GitHub y la contraseña del token

|![Alt](webroot/images/githubenlace.PNG)|
|![Alt](webroot/images/gitpestanapush.PNG)|

Indicamos las ramas que queremos publicar y aceptamos.

## Clonar un repositorio
No es muy diferente a publicar uno. Estando fuera de cualquier proyecto pulsamos la pestaña team en NetBeans y vamos a 'Team'>'Git'>'Clone'

|![Alt](webroot/images/gitclone.PNG)|

Se nos abrirá la siguiente pestaña en la que escribiremos el enlace del respositorio en GitHub que queremos clonar en local y la contraseña del token.

|![Alt](webroot/images/gitclonerepo.PNG)|

En el siguiente paso deberemos escoger las ramas que queremos clonar del repositorio de GitHub. En este caso, voy a clonar las dos que hay.

|![Alt](webroot/images/gitclonebranch.PNG)|

Por último indicamos donde queremos guardar el proyecto en local que será donde tenemos todos los proyectos del ciclo formativo.

|![Alt](webroot/images/gitclonedesti.PNG)|

Con esto ya podemos empezar a desarrollar en local, publicar los cambios en GitHub y llevarte el trabajo a donde quieras.
# 3.Entorno de Explotación

---

> **Álvaro Allén Perlines**  
> Curso: 2025/2026  
> 2º Curso CFGS Desarrollo de Aplicaciones Web  
> Despliegue de aplicaciones web
