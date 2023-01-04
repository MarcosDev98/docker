
# Virtualización

... versión virtual de algún recurso tecnológico, como (...) hardware, un sistema operativo, un dispositivo de almacenamiento o (...) recurso de red. 

La **virtualización** nos permite afrontar en simultaneo los tres problemas del desarrollo de software profesional.

Las máquinas virtuales nos permiten construir con algún tipo de software una version virtual (digital, por software) de una máquina y correr nuestro software ahí.

## Problemas de las VMs
  * ### **Peso:** en el orden de los GBs. Repiten archivos en común. Inicio lento.
  * ### **Costo de administración:** Necesita mantenimiento igual que cualquier computadora.
  * ### **Múltiples formatos:** VDI, VMDK, VHD, raw, etc.

#

# Contenedores

## Containerización
El empleo de **contenedores** para constuir y desplegar software.

## Contenedores:
* **Flexibles:** No importa que aplicación estes construyendo, vas a poder meterla en un contenedor.
* **Livianos:** reutilizan kernel del S.O de la máquina contenedora.
* **Portables:** estan diseñados para funcionar de la misma manera en cualquier máquina.
* **Bajo acomplamineto:** son autocontenidos, un contenedor tiene todo lo que necesita para poder correr el software que esta dentro suyo, y no le importa que otra cosa haya en el S.O o en otros contenedores. 
* **Escalables:** es muy fácil y rápido replicar contenedores.
* **Seguros:** las herramientas que corren contenedores (docker, etc.) se aseguran de que un contenedor solo pueda acceder a aquellas partes del sistema que necesita y que no pueda acceder a otras (otros contenedores, máquina anfitriona, etc.). 

#
# Arquitectura de Docker

![Image Text](arquitectura-docker.png)
Componentes DENTRO del circulo de Docker:

* **Docker daemon**: Es el centro de docker, el corazón que gracias a el, podemos comunicarnos con los servicios de docker.
* **REST API**: Como cualquier API, nos permite comunicarnos con el docker daemon (server) mediante http.
* **Cliente de docker**: Gracias a este componente, podemos comunicarnos con el corazón de docker (Docker Daemon) que por defecto es la línea de comandos.


Dentro de la arquitectura de Docker encontramos:

* **Contenedores**: Es la razón de ser de Docker, es donde podemos encapsular nuestras imagenes para llevarlas a otra computadora, o servidor, etc.
* **Imagenes**: Son las encapsulaciones de x contenedor. Podemos correr nuestra aplicación en Java por medio de una imagen, podemos utilizar Ubuntu para correr nuestro proyecto, etc.
* **Volumenes de datos**: Podemos acceder con seguridad al sistema de archivos de nuestra máquina.
* **Redes**: Son las que permiten la comunicación entre contenedores.

#
![TextImage](docker-architecture.png)
#

## Contenedores:
* Es una agrupación de procesos.
* Es una entidad lógica, no tiene el limite estricto de las máquinas virtuales, emulación del sistema operativo simulado por otra más abajo.
* Ejecuta sus procesos de forma nativa.
* Los procesos que se ejecutan adentro de los contenedores ven su universo como el contenedor lo define, no pueden ver mas allá del contenedor, a pesar de estar corriendo en una maquina más grande.
* No tienen forma de consumir más recursos que los que se les permite. Si esta restringido en memoria ram por ejemplo, es la única que pueden usar.
* A fines prácticos los podemos imaginar cómo maquinas virtuales, pero NO lo son. Máquinas virtuales livianas.
* Docker corre de forma nativa solo en Linux.
* Sector del disco: Cuando un contenedor es ejecutado, el daemon de docker le dice, a partir de acá para arriba este disco es tuyo, pero no puedes subir mas arriba.
* Docker hace que los procesos adentro de un contenedor este aislados del resto del sistema, no le permite ver más allá.
* Cada contenedor tiene un ID único, también tiene un nombre.
#

## Configurar usuario para poder utilizar docker sin permisos de root:

1. Creamos un grupo: 
``` 
  $ sudo groupadd docker
```
2. Agregamos nuestro usuario al grupo:
```
  $ sudo usermod -aG docker <user_name>
```
3. Cerramos la terminal y volvemos a abrir.
4. Activamos los comandos del grupo:
```
  $ newgrp docker
```

* Activar los comandos del grupo
```
  $ newgrp docker
```
#
# Comandos 

* Mostrar contenedores corriendo (activos):
 ```
  $ docker ps
```
* Mostrar todos los contenedores (activos e inactivos):
```
  $ docker ps -a
```
* Mostrar información de un contenedor:
```
  $ docker inspect <container_id>
  $ docker inspect <container_name>
```
* Correr un contenedor y nombrarlo como querámos:
```
  $ docker run --name <name> hello-world
```
* Renombrar un contenedor:
```
  $ docker rename <container_name> <new_container_name>
```
* Borrar un contenedor según su ID o su nombre:
```
  $ docker rm <container_id>
  $ docker rm <container_name>
```
* Borrar TODOS los contenedores APAGADOS:
```
  $ docker container prune
```

* Correr un S.O Ubuntu dentro de un container y ejecutarlo en modo interactivo (-i: interactivo, -t: abre la terminal):
```
  $ docker run ubuntu
  $ docker run -it ubuntu
```
#
# Ciclo de vida de los Contenedores

Cada vez que un contendor se ejecuta, en realidad lo que ejecuta es un proceso del sistema operativo. Este proceso se le conoce como **Main process**.

**Main process:**
Determina la vida del contenedor, un contendor corre siempre y cuando su proceso principal este corriendo.

**Sub process:**
Un contenedor puede tener o lanzar procesos alternos al main process, si estos fallan el contenedor va a seguir encedido a menos que falle el main.

Ejemplo:
```
  $ docker run --name alwaysup -d ubuntu tail -f /dev/null
```
Este comando genera un output que es el ID del contenedor.
Luego utilizamos el comando: 
 ```
  $ docker exec -it alwaysup bash
 ```
El comando exec nos permite que, en un contenedor que ya existe y que está corriendo, ejecutar un comando u proceso (**subproceso docker**).

Para detener el contenedor debo matar el proceso main (tail -f /dev/null/) el cual se encuentra en mi S.O.

Para eso ejecuto el siguiente comando:
```
  $ docker inspect --format '{{.State.Pid}}' alwaysup
```
El cuál me devuelve el process id del proceso que está ejecutando el proceso principal del contenedor (tail -f /dev/null).

Entonces si ejecutamos:
```
  $ kill <process_id>
```
Podremos matar el proceso principal del contenedor y así detenerlo.

#
# Cómo exponer un contenedor

```
  $ docker run -d --name proxy nginx
```
-d o --detach: no vincula standar input ni standar output al contenedor, lo hace correr en background.

Luego al ejecutar 
```
  $ docker ps
```
 veremos que nos muestra que puerto está utilizando. Pero el puerto que nos muestra no hace referencia a los puertos de mi máquina, sino a los puertos de docker.

Para correr un contenedor y especificar los puertos correctamente ejecutaremos el comando de la siguiente manera:
```
  $ docker run --name proxy -p 8080:80 -d nginx
```

Para detener un contenedor podemos utilizar el comando
```
  $ docker stop <container_name>
```

Para ver los logs de un contenedor debemos utilizar el comando:
```
  $ docker logs <container_name>
```
o también:
```
  $ docker logs --tail 10 -f <container_name>
```

#
# Manejo de datos en docker

Descargo y creo un contenedor con mongo:
```
  $ docker run -d --name db mongo
```
Abro una terminal dentro del contenedor:
```
  $ docker exec -it db bash
```

Una vez dentro ejecuto:
```
  $ mongosh
```
que es un binario que viene por defecto en este contenedor y me permite acceder al cliente de la base de datos.

Una vez dentro creo una nueva bd e inserto un nuevo usuario:
```
  $ use <db_name>
  $ db.users.insert({"nombre": "marcos"})
```

Si después de agregar el usuario elimino el contenedor, obviamente pierdo todos los datos que estaban en la base de datos.

Por lo cual, a continuación vamos a crear un  directorio en el cual vamos a tener una versión espejada del contenedor. Para eso vamos a usar una herramienta que se llama **Bind Mounts**
## Bind Mounts

Creo el directorio en el cual voy a espejar el contenedor:
```
  $ mkdir mongodata
```

Una vez creado el directorio ya puedo crear el nuevo contenedor:
```
  $ docker run -d --name db -v /home/marcos/platzi/docker/dockerdata/mongodata:/data/db mongo
```
El flag -v indica que voy a hacer un bind mount y seguido le indico la ruta completa del directorio que voy a utilizar (lo que está a la izquierda de : ).  A la derecha de los : le indico la ruta dentro del contenedor. En este caso /data/db/ porque sé que es ahí donde mongo guarda los datos.

Una vez creado el contenedor entro a él para utilizarlo y verificar que los datos se esten respaldando en la carpeta que yo especifiqué.

```
//Entro a mongo, inserto un usuario, salgo del contenedor y borro el contenedor.
  $ docker exec -it db bash
  $ mongosh
  $ use platzi
  $ db.users.insert({"nombre": "marcos"})
  $ db.users.find()
  $ exit
  $ exit
  $ docker rm -f db
```
Pero después de haber borrado el contenedor si ejecuto:
```
  $ ll mongodata
```
voy a ver que hay un montón de archivos que no estaban.

Ahora si volvemos a repetir el proceso de crear el contenedor nuevamente agregando este mismo directorio, vamos a ver que podemos "recuperar" la información.

```
  $ docker run -d --name db -v /home/marcos/platzi/docker/dockerdata/mongodata:/data/db mongo
  $ docker exec -it db bash
  $ mongosh
  $ use platzi
  $ db.users.find()
```

#
# Volúmenes

Los volúmenes o volumes son una evolución de los bind mounts que docker desarrolló para darle mas seguridad a las personas que ejecutan docker en entornos productivos. 

