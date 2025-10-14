
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
      - [1.2.3 **FileZilla**](#123-filezilla)
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
sudo hostnamectl          #Comprobar el nombre, el sistema operativo, la arquitectura, etc...

free -h                   #Comprobar la RAM total, en uso y libre. Parámetro -h para que salga  en Gb

lsblk
sudo fdisk -l /dev/sda    #Comprobar las distintas particiones del disco, su tamaño y su raíz

ip a
ip r                      #Comprobar IP y GW como se ve en las capturas de abajo

sudo resolvectl status    #Comprobar el DNS (educa.jcyl.es)

# Falta codigo
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


#### **Actualizar el sistema**
```bash
sudo apt update
sudo apt upgrade
```
##### **Configuración fecha y hora**

[Establecer fecha, hora y zona horaria](https://somebooks.es/establecer-la-fecha-hora-y-zona-horaria-en-la-terminal-de-ubuntu-20-04-lts/ "Cambiar fecha y hora")

##### **Cuentas administradoras**

> - [X] root(inicio)
> - [X] miadmin/paso
> - [X] miadmin2/paso

##### **Habilitar cortafuegos**

como activar cortafuegos

#### 1.1.2 Instalación del servidor web

##### Instalación

Para instalar un servidor web vamos a descargar y configurar Apache. El primer paso es actualizar el SO y también los paquetes instalados. A continuación instalamos Apache2 y abrimos el puerto 80 que es el utilizados por Apache. Al abrir el puerto se abrirá tanto el normal como el (v6) y, por recomendación de seguridad, lo borraremos.

```bash

sudo apt update                         #Actualizamos OS
sudo apt upgrade                        #Actualizamos paquetes instalados

sudo apt install apache2                #Instalamos Apache2 en la máquina
  
sudo ufw allow 80                       #Habilitamos el puerto 80 desde el cortafuegos.

sudo ufw status numbered                
sudo ufw delete 'numeropuerto'          #Borramos el puerto 80 (v6) usando su número de indetificación

```
##### Verficación del servicio

Para verificar que Apache se ha instalado y está funcionando correctamente tenemos dos formas: entrando a un navegador desde el ordenador anfitrión y buscando la página con al dirección IP de la máquina. Si aparece una página como la de abajo es que Apache esta funcionando correctamente y que ambas máquinas están en la misma red. La otra forma es mediante los siguientes comandos:

```bash

sudo service apache2 {opcion}
sudo systemctl {opcion} apache2

```
##### Virtual Hosts
##### Permisos y usuarios
Al crear la máquina virtual creamos al usuario miadmin con contraseña paso y con privilegios de sudo. Una vez configurada la red y verificado el servicio, creamos el usuario admin2:

```bash
sudo adduser miadmin2                   #Creamos el usuario con contraseña paso e ignoramos el resto de datos que nos piden
```

Una vez creado el usuario tenemos que darle privilegios de sudo, es decir, meterle en el grupo sudoers para que pueda hacer ciertos comandos.

```bash
sudo usermod -aG sudo miadmin2          
#Meter al usuario miadmin2 en el grupo sudo sin quitarle del resto de grupos que pertenece y -G indica los grupos suplementarios a los que quieres añadir el usuario.
```

#### 1.1.3 PHP
###### Instalación de PHP en el servidor apache
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

```
---
> **Importante**
> 
> 
#### 1.1.4 MySQL
#### 1.1.5 XDebug
#### 1.1.6 DNS
#### 1.1.7 SFTP
#### 1.1.8 Apache Tomcat
#### 1.1.9 LDAP

### 1.2 Windows 11
#### 1.2.1 **Configuración inicial**
##### **Nombre y configuración de red**
##### **Cuentas administradoras**
#### 1.2.2 **Navegadores**
#### 1.2.3 **FileZilla**
#### 1.2.4 **Netbeans**
#### 1.2.5 **Visual Studio Code**

## 2. GitHub
## 3.Entorno de Explotación

---

> **Álvaro Allén Perlines**  
> Curso: 2025/2026  
> 2º Curso CFGS Desarrollo de Aplicaciones Web  
> Despliegue de aplicaciones web
