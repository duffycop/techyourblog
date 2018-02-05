---

title: "01 - Instalando DRBD"
date: 2018-01-19
excerpt-separator: <!--more-->
categories:
  - DRBD
  - Tutorials

---
Bienvenidos aventureros.
En este post vamos a aprender a instalar DRBD. Solamente vamos a instalarlo<!--more-->, en los siguientes posts podremos configurarlo y testearlo.
En cada nodo tendremos un disco vacio y completamente dedicado a DRBD, por lo tanto usaremos este disco /dev/vda
  
## Pero, que es DRBD (Distributed Replicated Block Device)?

DRBD es un software que permite hacer replica de los datos de una particion o disco entre varias maquinas a traves de la red.
  
## Entorno

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
  
