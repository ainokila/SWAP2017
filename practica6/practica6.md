# SWAP2017
## Practica 6

### Creación de los discos duros.

En mi caso he añadido dos discos duros de 100Mb en Virtual Box.

### Instalación de mdadm

En caso de tener que instalar mdadm utilizaremos,

    sudo apt-get install mdadm

En mi caso no ha sido necesario.

### Comprobación de etiquetas de los discos.

Para ver que etiquetas se le ha asignado a los discos usaré,

    sudo fdisk -l 

En mi caso son /dev/sdb y /dev/sdc.


### Creación y formateo del raid

Vamos a proceder a crear un raid con los dos discos que se crearon anteriormente,

    sudo mdadm -C /dev/md127 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc

Ahora vamos a proceder a formatearlo con,

    sudo mkfs /dev/md127

### Montaje manual del disco

Para montar el disco de manera manual tenemos que crear la carpeta y usar mount,

    sudo mkdir dat
    sudo mount /dev/md127 /dat

Si reiniciamos se perderá el montaje, para ello añadiremos una linea en /etc/fstab.

### Montaje automático

Primero debemos saber cual es el UUID del md127, para ello ejecutaremos,

    ls -l /dev/disk/by-uuid/

Después simplemente vamos editamos el fichero /etc/fstab y añadimos la linea 

    UUID= nuestra UUID /dat ext2 defaults 0 0

Guardamos y ya siempre que arranque estará montado en /dat.

### Simulación de un fallo en disco sdb

Para simular un fallo ejecutaré,

    sudo mdadm --manage --set-faulty /dev/md0 /dev/sdb

![fallo](img/2.PNG)

Después simplemente lo quitamos,

![remove](img/3.PNG)

Como se puede apreciar /dev/sdb ya no esta operativo,

![detalles](img/4.PNG)

### Añadimos de nuevo el disco

    sudo mdadm --manage --add /dev/md127 /dev/sdb

Como se puede apreciar, vuelve a estar 100% operativo.

![FIN](img/5.PNG)