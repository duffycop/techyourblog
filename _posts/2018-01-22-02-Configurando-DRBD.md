---

title: "02 - Configurando DRBD"
date: 2018-01-22
excerpt-separator: <!--more-->
categories:
  - DRBD

---


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
  
