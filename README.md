# docker-notes
Mis notas sobre Docker.

# Table of Contents
* **[Introducción](#introducción)**
* **[Instalación](#instalación)**
* **[Containers](#containers)**
  * **[Ports in Containers](#ports-in-containers)**
* **[Images](#images)**
  * **[Crear nuestras propias imágenes](#crear-nuestras-propias-images)**
* **[Limpieza](#limpieza)**
  * **[Containers](#containers-1)**
  * **[Images](#images-1)**
  * **[Volumes](#volumes)**
* **[Ciclo de vida de un contenedor (Create/Start/Stop/Kill/Remove)](#ciclo-de-vida-de-un-contenedor-createstartstopkillremove)**
* **[Dockerfiles](#dockerfiles)**
  * **[Cómo Empezar](#cómo-empezar)**
  * **[La Aplicación](#la-aplicación)**
  * **[Build del Dockerfile](#build-del-dockerfile)**
  * **[Run the App](#run-the-app)**
* **[Volumes](#volumes)**
* **[Networking](#networking)**
* **[Compose: Linkar Containers](#docker-compose-linkar-containers)**
* **[Docker CheatSheet](#docker-cheatsheet)**


# Introducción

* **Image:** An _image_ is a lightweight, stand-alone, executable package that includes everything needed to run a piece of software, including the code, a runtime, libraries, environment variables, and config files.

* **Container:**
  * A _container_ is a runtime instance of an image—what the image becomes in memory when actually executed.
  * It runs completely isolated from the host environment by default, only accessing host files and ports if configured to do so.
  * Containers **run apps natively** on the host machine’s kernel.
  * They have _better performance_ characteristics than virtual machines that only get virtual access to host resources through a hypervisor.
  * Containers can get native access, each one running in a discrete process, taking no more memory than any other executable.

# Instalación

Cuando hablamos de "instalar docker" nos estamos refiriendo a instalar "Docker Engine". Para ello, lo haremos siguiendo los pasos indicados en la [Documentación Oficial](https://docs.docker.com/engine/installation/).

# Containers
A continuación se muestran los comandos más habituales con "containers":

### `ps`

Muestra los contenedores en ejecución (running).

* Ver containers arrancados:                         
```shell
$ docker ps
```
* Ver "todo el historial" de containers arrancados:
```shell
$ docker ps -a
```

* Ver el "ID" (-q) del último (-l) container:
```shell
$ docker ps -l -q
```

### `exec`
Para acceder a la shell de un "container" que tuvieramos previamente arrancado:

Indicando su ID:
```shell
$ sudo docker exec -i -t 665b4a1e17b6 /bin/bash
```

o sino también a través de su nombre:
```shell
$ sudo docker exec -i -t loving_heisenberg /bin/bash
```

### `run`
Crea y arranca un "container".
Por defecto un contenedor se arranca, ejecuta el comando que le digamos y se para:
```shell
$ docker run busybox echo hello world
```

Por lo general `docker run` tiene la siguiente estructura:
```shell
$ docker run -p <puerto_host>:<puerto_container> username/repository:tag
```

Podemos pasarle varias opciones al comando `run`, como por ejemplo:

* `run` **interactivo**:
```shell
$ docker run -t -i ubuntu:16.04 /bin/bash
```
  * **-h:** Le configuramos un hostname al "container".
  * **-t:** Asigna una TTY.
  * **-i:** Nos comunicamos con el "container" de manera interactiva.

  **Nota:** Al salir del modo interactivo (-i) el "container" se detendrá.

* `run` **Detached Mode**:

  Como ya sabemos, trás correr un contenedor de manera interactiva, este finaliza. Si se quieren hacer contenedores que corran servicios (por ejemplo, un servidor web) el comando es el siguiente:
  ```shell
  $ docker run -d -p 1234:1234 python:2.7 python -m SimpleHTTPServer 1234
  ```
  * **Explicación:** Esto ejecuta un servidor **Python** (SimpleHTTPServer module), en el puerto **1234**.
    * **-p 1234:1234** le indica a Docker que tiene que hacer un _port forwarding_ del contenedor hacia el puerto 1234 de la máquina host. Ahora podemos abrir un browser en la dirección http://localhost:1234.

    * **-d:** hace que el "container" corra en segundo plano. Esto nos permite ejecutar comandos sobre el mismo en cualquier momento mientras esté en ejecución. Por ejemplo:
    ```shell
    $ docker exec -ti <container-id> /bin/bash
    ```

      * Aquí simplemente se abre una **tty** en modo **interativo**. Podrían hacerse otras cosas como cambiar el _working directory_, definir variables de entorno, etc.


### `inspect`
Muestra detalles acerca de un contenedor:

* Info sobre un container:
```shell
$ docker inspect
```

* Dirección IP de container:
```shell
$ docker  inspect --format '{{ .NetworkSettings.IPAddress }}' <nombre_container>
```

### `logs`
* Mostrar logs sobre un container:
```shell
$ docker logs
```

### `stats`

* Estadísticas del container (CPU,MEM,etc.):
```shell
$ docker stats
```

### Ports in Containers
* Puertos publicos del container:                                        
```shell
$ docker port
```
* Publicar puerto 80 del container en un puerto random del Host:
```shell
$ docker run -p 80 nginx
```
* Publicar puerto 80 del container en el puerto 8080 del Host:
```shell
$ docker run -p 8080:80 nginx
```
* Publicar todos los puerto expuestos del container en puertos random del Host:
```shell
$ docker run -P nginx
```
* Listar todos los mapeos de los puertos de un container:
```shell
$ docker port <container_name>
```


# Images

* Ver listado de imagenes:
```shell
$ docker images
```

* Ver "todo el historial" de imágenes arrancadas:
```shell
$ docker images -a
```

* Borrar una imágen:
```shell
$ docker rmi <images_name>
```

* Para conocer el historial y "layers" que tiene una imagen:
```shell
$ docker history <image_name>
```

  ### Crear nuestras propias Images:

  Podemos crear nuestras propias imágenes de diferentes maneras:

  A) `docker commit`: build an image from a container.

  Ejemplo:
  ```shell
   $ docker commit -m "Mensaje que queramos" -a "Nombre del que lo ha hecho" container-id NEW_NAME:TAG
   $ docker commit -m "MongoDB y Scrapy instalados" -a "Etxahun" 79869875807 etxahun/scrapy_mongodb:0.1
   ```
  B) `docker build`: create an image from a "Dockerfile" by executing the build steps given in the file.

  Dentro de un Dockerfile las "instructions" que podemos utilizar son las siguientes:

   * **FROM**: the base image for building the new docker image; provide "FROM scratch" if it is a base image itself.
   * **MAINTAINER**: the author of the Dockerfile and the email.
   * **RUN**: any OS command to build the image.
   * **CMD**: specify the command to be stated when the container is run; can be overriden by the explicit argument when providing docker run command.
   * **ADD**: copies files or directories from the host to the container in the given path.
   * **EXPOSE**: exposes the specified port to the host machine.


  Ejemplo:
  ```shell
  $ nano myimage/Dockerfile

  FROM ubuntu
  RUN echo "my first image" > /tmp/first.txt

  $ docker build -f myimage/Dockerfile   || o sino ||   docker build myimage
  Sending build context to Doker daemon 2.048 kB
  Step 1: FROM ubuntu
  ----> ac526a456ca4
  Step 2: RUN echo "my first image" > /temp/first.txt
  ----> Running in 18f62f47d2c8
  ----> 777f9424d24d
  Removing intermediate container 18f62f47d2c8
  Succesfully built 777f9424d24d

  $ docker images | grep 777f9424d24d

  <none>	<none>		777f9424d24d		4 minutes ago		125.2 MB

  $ docker run -it 777f9424d24d
  $ root@2dcd9d0caf6f:/#
  ```
  Podemos ponerle un nombre o "tagear" la imagen en el momento que estemos haciendo el "build":
  ```shell
  $ docker build <dirname> -t "<imagename>:<tagname>"
  ```
  Ejemplo:
  ```shell
  $ docker build myimage -t "myfirstimage:latest"
  ```
### Tag the image

La notación que se suele utilizar para asociar una imagen local con un "repository" dentro de un "registry" es la siguiente: `username/repository:tag`. La parte "tag" es opcional, pero recomendable, ya que es la manera en que versionaremos en Docker las imágenes.

Para realizar el "tag" haremos lo siguiente:

```shell
$ docker tag image username/repository:tag
```

Por ejemplo:

```shell
docker tag friendlyhello john/get-started:part2
```

Para comprobar la imagen que acabamos de "tagear":

```shell
$ docker images_name

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
john/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

### Publish the image

Para publicar la imagen haremos lo siguiente:
```shell
$ docker push username/repository:tagear
```

Una vez subido, podremos ver en la Web de [Docker Hub](https://hub.docker.com/ "Docker Hub").


# Limpieza
### Containers:
* Borrar container:
```shell
$ docker rm <container_ID>
```

* Borrar container y sus volumenes asociados:
```shell
$ docker rm -v <container_ID>
```

* Borrar TODOS los containers:
```shell
$ docker rm $(docker ps -a -q)
```

### Images:
* Borrar imágenes:
```shell
$ docker rmi <container_ID>
```

* Borrar TODAS las imáganes:
```shell
$ docker rmi $(docker images -q)
```
* Listar imágenes <none> "colgadas":
```shell
$ docker images -f "dangling=true"
```

* Borrar imágenes <none> "colgadas":
```shell
$ docker rmi $(docker images -f "dangling=true" -q)
```

### Volumes:
* Borrar TODOS los "volume" que no se estén utilizando:
```shell
$ docker volume rm $(docker volumels -q)
```


# Ciclo de vida de un contenedor (Create/Start/Stop/Kill/Remove)

Hasta ahora vimos cómo ejecutar un contenedor tanto en *foreground* como en *background* (detached). Ahora veremos cómo manejar el ciclo completo de vida de un contenedor. Docker provee de comandos como `create` , `start`, `stop`, `kill` , y `rm`. En todos ellos podría pasarse el argumento "-h" para ver las opciones disponibles.
Ejemplo:
```shell
$ docker create -h
```

Más arriba vimos cómo correr un contenedor en segundo plano (detached). Ahora veremos en el mismo ejemplo, pero con el comando `create`. La única diferencia es que ésta vez no especificaremos la opción "-d". Una vez preparado, necesitaremos lanzar el contenedor con `docker start`.

Ejemplo:
```shell
$ docker create -P --expose=8001 python:2.7 python -m SimpleHTTPServer 8001
  a842945e2414132011ae704b0c4a4184acc4016d199dfd4e7181c9b89092de13

$ docker ps -a
  CONTAINER ID IMAGE      COMMAND              CREATED       ... NAMES
  a842945e2414 python:2.7 "python -m SimpleHTT 8 seconds ago ... fervent_hodgkin

$ docker start a842945e2414
  a842945e2414

$ docker ps
  CONTAINER ID IMAGE      COMMAND              ... NAMES
  a842945e2414 python:2.7 "python -m SimpleHTT ... fervent_hodgkin
```

### Detener Containers:

Siguiendo el ejemplo, para **detener** el contenedor se puede ejecutar cualquiera de los siguientes comandos `kill` ó `stop`:
```shell
$ docker kill a842945e2414 (envía SIGKILL)
$ docker stop a842945e2414 (envía SIGTERM).
```

Asimismo, pueden reiniciarse (hacer un `docker stop a842945e2414` y luego un `docker start a842945e2414`):

```shell
$ docker restart a842945e2414
```

o destruirse:

```shell
$ docker rm a842945e2414
```

# Dockerfiles

**Problema:**

Ejecutar contenedores en modo interactivo (-ti), hacer algunos cambios y para luego hacer un "commit" de estos en una nueva imagen, funciona bien. Pero en la mayoría de los casos, tal vez quieras automatizar este proceso de creación de nuestra propia imagen y compartir estos pasos con otros.

**Solución:**

Para automatizar el proceso de creación de imágenes Docker, crearemos los ficheros **Dockerfile**. Este archivo de texto está compuesto por:
* Una serie de instrucciones que describen cuál es la **imagen base** en la que está basado el nuevo contenedor.
* Los **pasos/instrucciones** que necesitan llevarse a cabo para instalar las dependencias de la aplicación.
* Archivos que necesitan estar presentes en la imagen.
* Los **puertos** serán expuestos por el contenedor.
* El/los **comando(s) a ejecutar** cuando se ejecuta el contenedor.

### Cómo Empezar

En primer lugar crearemos un directorio vacio y entraremos a dicho directorio
```shell
 $ mkdir pruebadockerfile
 $ cd pruebadockerfile/
 ```
Una vez dentro crearemos un fichero llamado "dockerfile" y le copiaremos el siguiente código:

```shell
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
Dentro del "dockerfile" se hace referencia a un par de fichero que no hemos creado aun: **app.py** y **requirements.txt**.

### La aplicación

Crearemos ambos ficheros dentro del mismo directorio donde se encuentra el fichero "dockerfile":

`requirements.txt`
```shell
Flask
Redis
```

`app.py`
```shell
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

Como vemos, en el fichero "requirements.txt" se especifican los paquetes de python **Flask** y **Redis** que se van a instalar.

### Build del Dockerfile

Ya estamos listos para hacer el "build" de la aplicación. Nos aseguraremos de estar en el mismo directorio donde están los fichero "dockerfile", "app.py" y "requirements.txt":

```shell
$ ls
Dockerfile		app.py			requirements.txt
```
Y a continuación realizamos el "build". Esto nos creará un "Docker image" que "tagearemos" con "-t" para que tenga un "friendly name":

```shell
$ docker build -t friendlyhello .
```
Para ver que se ha creado correctamente la imagen haremos lo siguiente:
```shell
$ docker images (o sino docker image ls)

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```
### Run the app

Arrancaremos la aplicación mapeando el puerto **4000** de nuestro host al puerto **80** del container mediante el parámetro "-p":

```shell
$ docker run -p 4000:80 friendlyhello
```

Si todo ha ido bien deberíamos ver como se ha cargado un servidor Web Flask de Python sirviendo en: `http://0.0.0.0:80`. Dicho mensaje lo indica el servidor Web que está corriendo dentro del container, sin embargo como hemos mapeado el puerto **4000** con el puerto **80**, abriremos el navegador y accederemos mediante: `http://localhost:4000`.

Si queremos que el container funcione en *background* (detached mode) haremos lo siguiente:

```shell
$ docker run -d -p 4000:80 friendlyhello
```
Hemos utilizado la opción "-d" para arrancarlo en "detached mode".

# Volumes

# Networking

# Docker Compose: Linkar containers

Cuando estamos diseñando una "aplicación distribuida", a cada una de las piezas se le conoce como "service". Por ejemplo, si pensamos en una aplicación de "Video Sharing site", tendremos que tener por un lado un servicio que nos permita almacenar en una base de datos tod el contenido multimedia, por otra parte tendremos un servicio para realizar el "transcoding" en background cada vez que un usuario suba un vídeo, tambien tendremos un servicio para la parte front-end, etc.

Llamamos "services" a los "containers" que pongamos en producción. Un servicio se compone de una sola imagen, con todo lo necesario para que ésta proporcione la funciones para lo que ha sido creada. En Docker, la manera en que definiremos dichas "images" es con "Docker Compose", escribiendo lo que se conocen como ficheros **docker-compose.yml**.

Cuando queramos "linkar" dos o más contenedores tendremos que establecer su relación en un fichero YAML. A continuación se muestra un ejemplo de un fichero que "linka" un container "Web" (Wordpress) y uno de base de datos "MySQL":

* Contenido del fichero **ejemplo.yml**:

  ```YAML
  web:
     image: wordpress
     links:
       - mysql
     environment:
       - WORDPRESS_DB_PASSWORD=sample
     ports:
       - "127.0.0.3:8080:80"
  mysql:
  image: mysql:latest
  environment:
     - MYSQL_ROOT_PASSWORD=sample
     - MYSQL_DATABASE=wordpress

  ```
  Y para ejecutarlo, estando en el mismo directorio donde está el fichero **ejemplo.yml**, haremos lo siguiente:

  ```shell
  $ docker-compose up
  ```
  Y para comprobar que todo ha ido bien, abriremos la url http://127.0.0.3:8080 para aceder a la página de Wordpress.

# Docker CheatSheet

A continuación se muestra un listado con los *comandos básicos* de Docker:

```shell
docker build -t friendlyname .                   # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyname               # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyname            # Same thing, but in detached mode
docker container ls                              # List all running containers
docker container ls -a                           # List all containers, even those not running
docker container stop <hash>                     # Gracefully stop the specified container
docker container kill <hash>                     # Force shutdown of the specified container
docker container rm <hash>                       # Remove specified container from this machine
docker container rm $(docker container ls -a -q) # Remove all containers
docker image ls -a                               # List all images on this machine
docker image rm <image id>                       # Remove specified image from this machine
docker image rm $(docker image ls -a -q)         # Remove all images from this machine
docker login                                     # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag       # Tag <image> for upload to registry
docker push username/repository:tag              # Upload tagged image to registry
docker run username/repository:tag               # Run image from a registry
```
