
# CFGS Desarrollo de Aplicaciones Web


- [CFGS Desarrollo de Aplicaciones Web](#cfgs-desarrollo-de-aplicaciones-web)
  - [1. Entorno de Desarrollo](#1-entorno-de-desarrollo)
    - [1.1 Ubuntu Server 24.04.3 LTS](#11-ubuntu-server-24043-lts)
      - [1.1.1 **Configuración inicial**](#111-configuración-inicial)
        - [Nombre y configuración de red](#nombre-y-configuración-de-red)
      - [**Actualizar el sistema**](#actualizar-el-sistema)
        - [**Configuración fecha y hora**](#configuración-fecha-y-hora)
        - [**Cuentas administradoras**](#cuentas-administradoras)
        - [**Habilitar cortafuegos**](#habilitar-cortafuegos)
      - [1.1.2 Instalación del servidor web](#112-instalación-del-servidor-web)
        - [Instalación](#instalación)
        - [Verficación del servicio](#verficación-del-servicio)
        - [Virtual Hosts](#virtual-hosts)
        - [Permisos y usuarios](#permisos-y-usuarios)
      - [1.1.3 PHP](#113-php)
          - [Instalación de PHP en el servidor apache](#instalación-de-php-en-el-servidor-apache)
      - [1.1.4 MySQL](#114-mysql)
      - [1.1.5 XDebug](#115-xdebug)
      - [1.1.6 DNS](#116-dns)
      - [1.1.7 SFTP](#117-sftp)
      - [1.1.8 Apache Tomcat](#118-apache-tomcat)
      - [1.1.9 LDAP](#119-ldap)
    - [1.2 Windows 11](#12-windows-11)
      - [1.2.1 **Configuración inicial**](#121-configuración-inicial)
        - [**Nombre y configuración de red**](#nombre-y-configuración-de-red-1)
        - [**Cuentas administradoras**](#cuentas-administradoras-1)
      - [1.2.2 **Navegadores**](#122-navegadores)
      - [1.2.3 **MobaXterm**](#123-MobaXterm)
      - [1.2.4 **Netbeans**](#124-netbeans)
      - [1.2.5 **Visual Studio Code**](#125-visual-studio-code)
  - [2. GitHub](#2-github)
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

#### **Cuentas administradoras**

> - [X] root(inicio)
> - [X] miadmin/paso
> - [X] miadmin2/paso

#### **Habilitar cortafuegos**
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

### 1.1.2 Instalación del servidor web

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
#### Permisos y usuarios
Al crear la máquina virtual creamos al usuario miadmin con contraseña paso y con privilegios de sudo. Una vez configurada la red y verificado el servicio, creamos el usuario admin2:

```bash
sudo adduser miadmin2                   #Creamos el usuario con contraseña paso e ignoramos el resto de datos que nos piden
```

Una vez creado el usuario tenemos que darle privilegios de sudo, es decir, meterle en el grupo sudoers para que pueda hacer ciertos comandos.

```bash
sudo usermod -aG sudo miadmin2          
# Meter al usuario miadmin2 en el grupo sudo sin quitarle del resto de grupos que pertenece y -G indica los grupos suplementarios a los que quieres añadir el usuario.
# Ahora lo que tenemos que hacer es el meter a miadmin2 en los mismos grupos que miadmin usando este comando y listando
  con cat /etc/group | grep miadmin

**Comandos recomendados**
sudo deluser "nombreusuario"            # Borra el usuario indicado.
su nombreusuario                        # Inicia sesión en el usuario indicado.
exit                                    # Para salir de la sesión actual.
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
### 1.1.3 PHP
#### Instalación de PHP en el servidor apache
Una vez actualizado el sistema y mejorado los paquetes (update y upgrade) debemos de realizar los siguientes pasos:
```bash
# Comprobamos que apache está instalado y activo.
sudo systectl status apache2

# Después añadimos el repositorio PPA de Ondrej para PHP:
sudo apt install software-properties-common
# Puede que ya venga instalado...

#Añadimos el repositorio de ondrej/php y comprobamos si ha sido instalado.
sudo add-apt-repository ppa:ondrej/php -y
ls /etc/apt/sources.list.d/ | grep ondrej
#Actualizamos todos los repositorios.
sudo apt update
```

Ahora instalamos la versión PHP-FPM y los módulos de Apache. En este caso la versión instalada será la PHP 8.3. A mayores, instalaremos otras extensiónes útiles. 

```bash
# En el mismo comando va la instalación de PHP y de las extensiones.
sudo apt install php8.3 libapache2-mod-php8.3 php8.4-fpm -y
```
---

> **Importante**
> Ahora se va a habilitar ciertos módulos de PHP y puede saltar un error.
> En caso de que salte error realizar los siguientes pasos previos.

```bash
# Primero deshabilitamos el mpm_prefork y no saldrá un posible error.
sudo a2dismod mpm_prefork
```

```bash
# Para resolver el error anterior desactivamos el módulo mod_php.
sudo a2dismod php8.3
# Este comando elimina la dependencia con mpm_prefork y ahora si podremos desahabilitarlo.
```

```bash
# Por último, una vez deshabilitado mpm_prefork, activaremos mpm_event y proxy_fcgi
sudo a2enmod mpm_event proxy_fcgi
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


#### Configuración del intérprete PHP.
La configuración que vamos a aplicar en nuestro servicio de PHP es el siguiente:
> display_errors: On
> display_startup: On
> memory_limit: 256M

El display_errors sirve para mostrar los errores de ejecución en la salida (navegador
o consola) y el display_startup_erros para controlar los errores que surgen durante la
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
que dichos valores han cambiado y son los introducidos en el archivo de configuración.ç

> **Consejo:**
>
> En el info.php hay un apartado llamado "Loaded configuration File" que indica la ruta donde
se encuentra el archivo que acabamos de modificar.

### 1.1.4 MySQL
### 1.1.5 XDebug
### 1.1.6 DNS
### 1.1.7 SFTP
### 1.1.8 Apache Tomcat
### 1.1.9 LDAP

## 1.2 Windows 11
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

> Seleccionamos SSH como protocolo de conexión.

> Introducimos en el apartado Remote host la IP del ordenador al que queremos conectarnos. También 
se puede especificar al usuario al que queremos conectarnos en el apartado Specify username, pero no 
es obligatorio.

> Se nos abrirá un terminal donde iniciaremos sesión con un usuario y una contraseña valida.


Para conectar un dispositovo por SFTP para la transferencia de datos debemos realizar los siguientes pasos:
> Pulsamos el botón Session con una pantalla como símbolo.

> Seleccionamos SFTP como protocolo de conexión

> Introducimos la IP del ordenador al que queremos conectarnos en el apartado "Remote hosts" y, esta vez,
si debemos especificar el usuario al que queremos conectarnos en el apartado "Username". Para nosotros es "operadorweb".

> Comprobamos que nos encontramos en el directorio raiz /var/www/html y que podemos crear carpetas y archivos, modificarlos y eliminarlos.
### 1.2.4 **Netbeans**

### 1.2.5 **Visual Studio Code**

# 2. GitHub
# 3.Entorno de Explotación

---

> **Álvaro Allén Perlines**  
> Curso: 2025/2026  
> 2º Curso CFGS Desarrollo de Aplicaciones Web  
> Despliegue de aplicaciones web
