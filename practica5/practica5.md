# SWAP2017
## Practica 4

### Conexión de las maquinas

| Maquina 1 Maestro  | Maquina 2 Esclavo | 
| ---------- | ---------- |
| 10.0.2.4   | 10.0.2.15  |

### Creación de una base de datos simple.

Primero ejecutamos mysql -u root, nos pedirá la contraseña, una vez puesta, ya estaremos en la terminal de mysql y debemos poner:

    mysql> create database contactos;
    mysql> use contactos;
    mysql> create table datos(nombre varchar(100),tlf int);
    mysql> insert into datos(nombre,tlf) values ("pepe",95834987);

Una vez realizado, ya tendremos una base de datos llamada contactos con una entrada.


### Replicar una BD MySQL con mysqldump.

Una vez realizada la configuración anterior en la Maquina 1 , se va a proceder a replicar la base de datos a la máquina 2, para ello usaremos mysqldump.

    mysql -u root –p
    mysql> FLUSH TABLES WITH READ LOCK;
    mysql> quit

    mysqldump ejemplodb -u root -p > /tmp/ejemplodb.sql

    mysql -u root –p
    mysql> UNLOCK TABLES;
    mysql> quit

Ya tendremos la copia de seguridad en /tmp/ejemplobd.sql, ahora debemos copiarlo a Máquina 2.

    scp maquina1:/tmp/ejemplodb.sql /tmp/

Una vez que tenemos el fichero en la Maquina 2, realizaremos:

    mysql -u root –p
    mysql> CREATE DATABASE ‘ejemplodb’;
    mysql> quit
    mysql -u root -p ejemplodb < /tmp/ejemplodb.sql

Ya tendremos las bases de datos totalmente replicadas.

### Replicación de BD mediante una configuración maestro-esclavo

Para evitar problemas de automatizacion del paso anterior, vamos a proceder a hacer una configuración maestro esclavo, donde la Máquina 1 será el maestro y la máquina 2 el esclavo.

####Configuración en maquina 1 (Maestro)

File '/etc/mysql/mysql.conf.d/mysqld.cnf':

    #bind-address 127.0.0.1
    log_error = /var/log/mysql/error.log
    server-id = 1
    log_bin = /var/log/mysql/bin.log

Y ahora reiniciamos el servicio.

####Configuración en maquina 2 (Esclavo)

File '/etc/mysql/mysql.conf.d/mysqld.cnf':

    #bind-address 127.0.0.1
    log_error = /var/log/mysql/error.log
    server-id = 2
    log_bin = /var/log/mysql/bin.log
    
Y ahora reiniciamos el servicio.

#### Mysql en Maestro
Ahora debemos ir a la máquina maestro y vamos a crear el esclavo,

    mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
    mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
    mysql> FLUSH PRIVILEGES;
    mysql> FLUSH TABLES;
    mysql> FLUSH TABLES WITH READ LOCK;

#### Mysql en esclavo
Ahora debemos ir a la máquina esclavo y añadimos,

    mysql> CHANGE MASTER TO MASTER_HOST='10.2.0.4',MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',MASTER_LOG_FILE='bin.000002', MASTER_LOG_POS=442, MASTER_PORT=3306;
    mysql> START SLAVE;

Para que el esclavo pueda acceder hay que desbloquear las tablas en el maestro:
    mysql> UNLOCK TABLES;

Una vez realizado mostramos en el esclavo su status y si seconds_behind_master es distinto de null esta funcionando, en mi caso obtengo,

    mysql> SHOW SLAVE STATUS\G

![esclavo](img/fin.PNG)

Ahora insertamos unos valores en el maestro y comprobamos si se clonan, en mi caso 1 de ellos no se clona debido a que era el que restaure en el ejercicio anterior.

![esclavo](img/fin2.PNG)

Como se puede apreciar funcionan perfectamente.
