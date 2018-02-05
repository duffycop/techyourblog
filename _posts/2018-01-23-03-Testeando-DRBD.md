---

title: "03 - Testeando DRBD"
date: 2018-01-23
excerpt-separator: <!--more-->
categories:
  - DRBD

---

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
  

