# SWAP2017
## Practica 2

### Conexión de las maquinas

| Maquina 1  | Maquina 2  |
| ---------- | ---------- |
| 10.0.2.4   | 10.0.2.15  |

### Instalación de rsync
En mi caso, no ha sido necesario instalar rsync, ya que viene instalada en Ubuntu Server 16.04, en caso de necesitar instalarlo se instala con:
	
	sudo apt-get install rsync

Una vez realizado, tenemos que modificar el propietario de el directorio /var/www/ en ambos equipos, en mi caso usaré el usuario ainokila:

	sudo chown ainokila:ainokila -R /var/www/

De esa manera, el usuario ainokila ahora sera el propietario, esto debe aplicarse en ambas maquinas.

Ahora podemos ver si funciona con:

	rsync -avz -e ssh 10.0.2.4:/var/www/ /var/www/

Introducimos la contraseña de SSH y listo.

### SSH sin contraseña

Para poder automatizar el proceso, y que no nos este pidiendo la contraseña cada vez que ejecutamos la orden anterior, vamos a generar un par de claves rsa que serviran para conectarse sin contraseña.

| Maquina 1  | Maquina 2                                  |
| ---------- | -------------------------------------------|
| 		     | ssh-keygen -b 4096 -t rsa                  |
|            | Dejamos la contraseña vacia                |
|            | ssh-copy-id 10.0.2.4                       |
|            | Introducimos por ultima vez contraseña SSH |
|            | Fin                                        |

Una vez realizado eso, ya podemos conectarnos entre las maquinas sin necesidad de poner claves.

### Automatización con Crontab

Una vez que ya no nos pide la contraseña, vamos a proceder a que la máquina 2 realice una copia cada minuto por ejemplo, debemos editar el archivo /etc/crontab:

	sudo nano /etc/crontab

Y debemos añadir esta linea:

	* * * * * ainokila rsync -avz -e ssh 10.0.2.4:/var/www/ /var/www/

Ahora debemos recargar cron, para que vuelva a cargar el fichero de tareas:

	sudo /etc/init.d/cron reload

Ahora nuestro sistema de copia automatica esta funcionando.

![Funcionando Practica 2](./img/P2.png)

