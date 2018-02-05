---
date: 2018-01-19
title: Instalando DRBD
categories:
  - DRBD
  - Tutorials
description:
type: Document
excerpt-separator: <!--more-->
---

Bienvenidos aventureros.
En este post vamos a aprender a instalar DRBD. Solamente vamos a instalarlo<!--more-->, en los siguientes posts podremos configurarlo y testearlo.
En cada nodo tendremos un disco vacio y completamente dedicado a DRBD, por lo tanto usaremos este disco /dev/vda
  
## Que es DRBD (Distributed Replicated Block Device)?

DRBD es un software que permite hacer replica de los datos de una particion o disco entre varias maquinas a traves de la red.
  
### Entorno

> Nodo1 | drbd1 | 192.168.10.10 | CentOS 7  
> Nodo2 | drbd2 | 192.168.10.11 | CentOS 7  
  
## Instalando
Para comenzar con la instalación lo primero que haremos sera ir a una terminal de linux.  
1 - Ejecutaremos los siguientes comandos para agregar el repositorio ELRepo en ambos nodos:
```
# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org  
# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```  
  
2 - Luego ejecutamos lo siguiente para instalar DRBD y los modulos del kernel necesarios para el funcionamiento de DRBD:
```
# yum install drbd90-utils kmod-drbd90
```  
  
Cuando finalice la instalación comprobaremos si el modulo del kernel fue cargado:
```
# lsmod | grep -i drbd
```  
  
Si no fue cargado automáticamente entonces ejecutamos:
```
# modprobe drbd
```  
  
Nota  
: El comando modprobe cargara el modulo durante la sesión activa, para que el modulo se cargue en el inicio hay que utilizar el servicio **systemd-modules-load** creando un archivo dentro de **/etc/modulesload.d/** y de esta manera el modulo es cargado durante cada inicio del sistema.  
```
# echo drbd > /etc/modules-load.d/drbd.conf
```  
  
## Configurando DRBD
Luego de instalar DRBD de forma correcta lo que haremos sera cambiar la configuración global editando el archivo **/etc/drbd.d/global_common.conf**  
1 - Vamos a hacer un backup del archivo original:
```
# mv /etc/drbd.d/global_common.conf /etc/drbd.d/global_common.conf.orig
```  
  
2 - Creamos un nuevo archivo de configuración global:
```
# vi /etc/drbd.d/global_common.conf  
global {
 usage-count no;
}
common {
 net {
  protocol C;
 }
}
```  
  
3 - Lo siguiente sera crear un archivo de configuración para el recurso **drbd0**:
```
# vi /etc/drbd.d/drbd0.res  
resource drbd0 {
	disk /dev/vda;
	device /dev/drbd0;
	meta-disk internal;
	on drbd1 {
		address 192.168.10.10:7789;
	}
	on drbd2 {
		address 192.168.10.11:7789;
	}
}
```  
Nota
: En el archivo de recurso de arriba hemos creado un nuevo recurso llamado drbd0, donde 192.168.10.10 y 192.168.10.11 son las IP de nuestros nodos y 7789 el puerto utilizado para la comunicación, usando el disco /dev/vda para crear el dispositivo /dev/drbd0.  
  
4 - Inicializar el dispositivo drbd0 en cada nodo:
```
# drbdadm create-md drbd0
```  
  
5 - Iniciar y habilitar el demonio drbd en ambos nodos:
```
# systemctl start drbd
# systemctl enable drbd
```  
  
6 - Definamos el Nodo1 como nodo primario:
```
# drbdadm up drbd0
# drbdadm primary drbd0
```  
Nota
: En caso de tener algún error para definir el nodo como primaro ejecutamos el mismo comando pero con el parámetro --force para forzar esta opcion.  
```
# drbdadm primary drbd --force
```  
  
7 - Para chequear el estado de la sincronización mientras se ejecuta utilizamos el siguiente comando:
```
drbdadm status
```  
  
8 - Ajustes de firewall en ambos nodos:
```
# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="ip_nodo" port port="7789" protocol="tcp" accept'
# firewall-cmd --reload
```  
  
## Testeando DRBD

Para testear DRBD crearemos un filesystem, montaremos el volumen y crearemos archivos en el nodo primario _drbd1_, finalmente cambiaremos el nodo primario a _drbd2_ y comprobaremos que DRBD funciona.  
Con el siguiente comando crearemos un filesystem xfs en /dev/drbd0 y lo montaremos en /mnt/
```
# mkfs.xfs /dev/drbd0
# mount /dev/drbd0 /mnt
```  
  
Crearemos archivos con
```
# touch /mnt/file{1..5}
# ls -l /mnt/
total 0
-rw-r--r--. 1 root root 0 Sep 22 21:43 file1
-rw-r--r--. 1 root root 0 Sep 22 21:43 file2
-rw-r--r--. 1 root root 0 Sep 22 21:43 file3
-rw-r--r--. 1 root root 0 Sep 22 21:43 file4
-rw-r--r--. 1 root root 0 Sep 22 21:43 file5
```  
   
Ahora cambiaremos el nodo primario de _drbd1_ a _drbd2_  
Primero desmontamos el volumen drbd0
```
# umount /mnt
```  
  
Cambiamos el Nodo1 de primario a secundario
```
# drbdadm secondary drbd
```  
  
Ahora cambiamos el Nodo2 de secundario a primario
```
# drbdadm primary drbd
```  
  
Montamos el filesystem y comprobamos que tenga los archivos dentro
```
# mount /dev/drbd0 /mnt
# ls -l  /mnt
total 0
-rw-r--r--. 1 root root 0 Sep 22 21:43 file1
-rw-r--r--. 1 root root 0 Sep 22 21:43 file2
-rw-r--r--. 1 root root 0 Sep 22 21:43 file3
-rw-r--r--. 1 root root 0 Sep 22 21:43 file4
-rw-r--r--. 1 root root 0 Sep 22 21:43 file5
```  
  