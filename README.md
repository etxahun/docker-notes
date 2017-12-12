# docker-notes
Mis notas sobre Docker.

# Table of Contents
* **[Introducción](#introducción)**
  * **[What is Docker?](#what-is-docker)**
  * **[Docker Engine](#docker-engine)**
  * **[Docker Architecture](#docker-architecture)**
  * **[Docker Objects](#docker-objects)**
  * **[The underlying technology](#the-underlying-technology)**
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
  * **[Network Containers](#network-containers)**
    * **[1. Crear nuestro propio "Bridge Network"](#1-crear-nuestro-propio-bridge-network)**
    * **[2. Add Containers to a Network](#2-add-containers-to-a-network)**
* **[Compose: Linkar Containers](#docker-compose-linkar-containers)**
* **[CheatSheets](#cheatsheets)**
  * **[Docker General Commands](#docker-general-cheatsheet)**
  * **[Dockerfile Commands](#dockerfile-commands)**

# Introducción

### What is Docker?

Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly.

### Docker Engine

Docker Engine is a *client-server* application with these major components:

* A server which is a type of long-running program called a daemon process (the dockerd command).

* A REST API which specifies interfaces that programs can use to talk to the daemon and instruct it what to do.

* A command line interface (CLI) client (the docker command).

<p align="center">
  <img src="images/engine-components-flow.png">
</p>


### Docker Architecture

As previously mentioned, Docker uses a **client-server** architecture.
* The **Docker client** talks to the **Docker daemon**, which does the heavy lifting of building, running, and distributing your Docker containers.
* The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon.
* The Docker client and daemon communicate using a **REST API**, over UNIX sockets or a network interface.

<p align="center">
  <img src="images/architecture.svg">
</p>

### Docker Objects

When you use Docker, you are creating and using **images**, **containers**, **networks**, **volumes**, **plugins**, and **other objects**. This section is a brief overview of some of those objects.

* **Image:**
  * An _image_ is a lightweight, stand-alone, executable package that includes everything needed to run a piece of software, including the code, a runtime, libraries, environment variables, and config files.

  * An image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some additional customization.

  * You might create your own images. To do that, you create a **Dockerfile** with a simple syntax for defining the steps needed to create the image and run it. Each instruction in a Dockerfile creates a layer in the image.

* **Container:**
  * A _container_ is a runtime instance of an image.
  * **Isolation:** It **runs completely isolated** from the host environment by default, only accessing host files and ports if configured to do so. By default, a container is relatively well isolated from other containers and its host machine. You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine.
  * Containers **run apps natively** on the host machine’s kernel.
  * They have **better performance** characteristics than virtual machines that only get virtual access to host resources through a hypervisor.
  * Containers can get native access, each one running in a discrete process, taking no more memory than any other executable.

### The underlying technology

Docker is written in **Go** and takes advantage of several features of the Linux kernel to deliver its functionality.

* **Namespaces**

Docker uses a technology called **namespaces** to provide the isolated workspace called the container. When you run a container, Docker creates a set of namespaces for that container.

These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace.

**Docker Engine** uses namespaces such as the following on Linux:

  * **The `pid` namespace:** Process isolation (PID: Process ID).
  * **The `net` namespace:** Managing network interfaces (NET: Networking).
  * **The `ipc` namespace:** Managing access to IPC resources (IPC: InterProcess Communication).
  * **The `mnt` namespace:** Managing filesystem mount points (MNT: Mount).
  * **The `uts` namespace:** Isolating kernel and version identifiers. (UTS: Unix Timesharing System).

* **Control Groups**

Docker Engine on Linux also relies on another technology called control groups (**cgroups**). A **cgroup** limits an application to a specific set of resources. Control groups allow Docker Engine to share available hardware resources to containers and optionally enforce limits and constraints. For example, you can limit the memory available to a specific container.

* **Union File Systems**

Union file systems, or **UnionFS**, are file systems that operate by **creating layers**, making them **very lightweight** and **fast**. Docker Engine uses UnionFS to provide the building blocks for containers. Docker Engine can use multiple UnionFS variants, including AUFS, btrfs, vfs, and DeviceMapper.

* **Container Format**

Docker Engine combines the namespaces, control groups, and UnionFS into a wrapper called a **container format**. The default container format is **libcontainer**. In the future, Docker may support other container formats by integrating with technologies such as BSD Jails or Solaris Zones.

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

* **Problema:**

  Ejecutar contenedores en modo interactivo (-ti), hacer algunos cambios y para luego hacer un "commit" de estos en una nueva imagen, funciona bien. Pero en la mayoría de los casos, tal vez quieras automatizar este proceso de creación de nuestra propia imagen y compartir estos pasos con otros.

* **Solución:**

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

When you install Docker, it creates **three networks** automatically. You can list these networks using the `docker network ls` command:

```shell
$ docker network ls

NETWORK ID          NAME                DRIVER
7fca4eb8c647        bridge              bridge
9f904ee27bf5        none                null
cf03ee007fb4        host                host
```

When you run a container, you can use the `--network` flag to specify which networks your container should connect to.

* **Bridge:** The bridge network represents the `docker0` network present in all Docker installations. Unless you specify otherwise with the `docker run --network=<NETWORK>` option, the Docker daemon connects containers to this network by default.

We can see this bridge as part of a host’s network stack by using the `ip addr show` command:

```shell
$ ip addr show

docker0   Link encap:Ethernet  HWaddr 02:42:47:bc:3a:eb
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:47ff:febc:3aeb/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:17 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1100 (1.1 KB)  TX bytes:648 (648.0 B)
```

* **None:** The `none` network adds a container to a container-specific network stack. That container lacks a network interface. Attaching to such a container and looking at its stack you see this:

```shell
$ docker attach nonenetcontainer

root@0cb243cd1293:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters

root@0cb243cd1293:/# ip -4 addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever

root@0cb243cd1293:/#
```
* **Note:** You can detach from the container and leave it running with `CTRL-p CTRL-q`.

* **Host:** The `host` network adds a container on the host’s network stack. As far as the network is concerned, **there is no isolation between the host machine and the container**. For instance, if you run a container that runs a web server on port 80 using host networking, the web server is available on port 80 of the host machine.

The `none` and `host` networks are not directly configurable in Docker. However, you can configure the default `bridge` network, as well as your own user-defined bridge networks.

### Network Containers

Docker permite realizar "containers networking" gracias al uso de sus **network drivers**. Por defecto, Docker proporciona dos **drivers**: `bridge` y `overlay`.

Toda instalacion de **Docker Engine** automáticamente incluye las siguientes tres redes:

```shell
$ docker network ls

NETWORK ID          NAME                DRIVER
18a2866682b8        none                null
c288470c46f6        host                host
7b369448dccb        bridge              bridge
```
* **Bridge:** Es una red especial. A menos que especifiquemos lo contrario, Docker siempre arrancará los containers en ésta red. Podemos probar a hacer lo siguiente:
```shell
$ docker run -itd --name=networktest ubuntu

74695c9cea6d9810718fddadc01a727a5dd3ce6a69d09752239736c030599741
```
<p align="center">
  <img src="images/bridge1.png">
</p>

Para comprobar la IP del container, haremos lo siguiente:

```shell
$ docker network inspect bridge

[
    {
        "Name": "bridge",
        "Id": "f7ab26d71dbd6f557852c7156ae0574bbf62c42f539b50c8ebde0f728a253b6f",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.1/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "3386a527aa08b37ea9232cbcace2d2458d49f44bb05a6b775fba7ddd40d8f92c": {
                "Name": "networktest",
                "EndpointID": "647c12443e91faf0fd508b6edfe59c30b642abb60dfab890b4bdccee38750bc1",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "9001"
        },
        "Labels": {}
    }
]
```

Para desconectar un container de una red tendremos que indicar la red en la que está conectada así como el nombre del container:

```shell
$ docker network disconnect bridge networktest
```

**Importante:** Networks are natural ways to isolate containers from other containers or other networks.

#### 1. Crear nuestro propio "Bridge Network"

Como ya hemos comentado, **Docker Engine** soporta dos tipos de redes: *bridge* y *overlay*:

* **Bridge:** se limita a un "single host" donde esté funcionando "Docker Engine".
* **Overlay:** puede incluir múltimples hosts con "Docker Engine" instalado.

A continuación crearemos un "bridge network":

```shell
$ docker network create -d bridge my_bridge
```

El flag "-d" indica a Docker que tiene que cargar el driver de red "bridge". Es opcional ya que Docker por defecto carga "bridge".

Si volvemos a listar los drivers de red, veremos el que acabamos de crear:

```shell
$ docker network ls

NETWORK ID          NAME                DRIVER
7b369448dccb        bridge              bridge
615d565d498c        my_bridge           bridge
18a2866682b8        none                null
c288470c46f6        host                host
```

Y si hacemos un "inspect" de la red veremos que no tiene ninguna información:

```shell
$ docker network inspect my_bridge

[
    {
        "Name": "my_bridge",
        "Id": "5a8afc6364bccb199540e133e63adb76a557906dd9ff82b94183fc48c40857ac",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1"
                }
            ]
        },
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

#### 2. Add Containers to a Network

Cuando construyamos aplicaciones Web que deban funcionar de manera conjunta, por seguridad, crearemos una red. Las redes, por definicion, proporcionan una aislamiento completo a los containers. Cuando vayamos a arrancar un container podremos agregarlo a una red.

En el siguiente ejemplo arrancaremos un container de base de datos PostgreSQL pasándole el flag "--net=my_bridge":

```shell
$ docker run -d --net=my_bridge --name db training/postgres
```

Si ahora realizamos un "inspect" de la red "my_bridge" veremos que tiene un container asociado. También podemos inspeccionar el container para ver a qué red está conectado:

```shell
$ docker inspect --format='{{json .NetworkSettings.Networks}}'  db

{"my_bridge":{"NetworkID":"7d86d31b1478e7cca9ebed7e73aa0fdeec46c5ca29497431d3007d2d9e15ed99",
"EndpointID":"508b170d56b2ac9e4ef86694b0a76a22dd3df1983404f7321da5649645bf7043","Gateway":"10.0.0.1","IPAddress":"10.0.0.254","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02"}}
```

Si ahora arrancamos la imagen Web veremos que la red está de la siguiente manera:

```shell
$ docker run -d --name web training/webapp python app.py
```

<p align="center">
  <img src="images/bridge2.png">
</p>


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

# CheatSheets
### Docker General Commands

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

### Dockerfile Commands

#### ADD

The ADD command gets two arguments: a source and a destination. It basically copies the files from the source on the host into the container's own filesystem at the set destination. If, however, the source is a URL (e.g. http://github.com/user/file/), then the contents of the URL are downloaded and placed at the destination.

Example:
```shell
# Usage: ADD [source directory or URL] [destination directory]
ADD /my_app_folder /my_app_folder
```

#### CMD

The command `CMD`, similarly to `RUN`, can be used for executing a specific command. However, unlike `RUN` it is **not executed during build**, but **when a container is instantiated** using the image being built. Therefore, it should be considered as an initial, default command that gets executed (i.e. run) with the creation of containers based on the image.

To clarify: an example for `CMD` would be running an application upon creation of a container which is already installed using RUN (e.g. RUN apt-get install …) inside the image. This default application execution command that is set with CMD becomes the default and replaces any command which is passed during the creation.

Example:
```shell
# Usage 1: CMD application "argument", "argument", ..
CMD "echo" "Hello docker!"
```
#### ENTRYPOINT

`ENTRYPOINT` argument sets the concrete default application that is used every time a container is created using the image. For example, if you have installed a specific application inside an image and you will use this image to only run that application, you can state it with ENTRYPOINT and whenever a container is created from that image, your application will be the target.

If you couple `ENTRYPOINT` with `CMD`, you can remove "application" from CMD and just leave "arguments" which will be passed to the ENTRYPOINT.

* **Example:**

  ```shell
  # Usage: ENTRYPOINT application "argument", "argument", ..
  # Remember: arguments are optional. They can be provided by CMD
  #           or during the creation of a container.
  ENTRYPOINT echo

  # Usage example with CMD:
  # Arguments set with CMD can be overridden during *run*
  CMD "Hello docker!"
  ENTRYPOINT echo
  ```

#### ENV

The `ENV` command is used to set the environment variables (one or more). These variables consist of “key = value” pairs which can be accessed within the container by scripts and applications alike. This functionality of docker offers an enormous amount of flexibility for running programs.

* **Example:**

  ```shell
  # Usage: ENV key value
  ENV SERVER_WORKS 4
  ```

#### EXPOSE

The `EXPOSE` command is used to associate a specified port to enable networking between the running process inside the container and the outside world (i.e. the host).

* **Example:**

  ```shell
  # Usage: EXPOSE [port]
  EXPOSE 8080
  ```

#### FROM

`FROM` directive is probably **the most crucial command** amongst all others for Dockerfiles. It **defines the base image to use to start the build process**. It can be any image, including the ones you have created previously. If a `FROM` image is not found on the host, docker will try to find it (and download) from the docker image index. It needs to be the first command declared inside a Dockerfile.

* **Example:**

  ```shell
  # Usage: FROM [image name]
  FROM ubuntu
  ```

#### MAINTAINER

One of the commands that can be set anywhere in the file - although it would be better if it was declared on top - is `MAINTAINER`. This non-executing command **declares the author**, hence setting the author field of the images. It should come nonetheless after `FROM`.

* **Example:**

  ```shell
  # Usage: MAINTAINER [name]
  MAINTAINER authors_name
  ```

#### RUN

The `RUN` command is the central executing directive for Dockerfiles. It takes a command as its argument and runs it to form the image. Unlike `CMD`, it actually is used to build the image (forming another layer on top of the previous one which is committed).

* **Example:**

  ```shell
  # Usage: RUN [command]
  RUN aptitude install -y riak
  ```

### USER

The `USER` directive is used to set the UID (or username) which is to run the container based on the image being built.

* **Example:**

  ```shell
  # Usage: USER [UID]
  USER 751
  ```

#### VOLUME

The `VOLUME` command is used to enable access from your container to a directory on the host machine (i.e. mounting it).

* **Example:**

  ```shell
  # Usage: VOLUME ["/dir_1", "/dir_2" ..]
  VOLUME ["/my_files"]
  ```

#### WORKDIR

The `WORKDIR` directive is used to set where the command defined with CMD is to be executed.

* **Example:**

  ```shell
  # Usage: WORKDIR /path
  WORKDIR ~/
  ```

#### EJEMPLO: Create an Image to Install MongoDB

I will create a Dockerfile document and populate it step-by-step with the end result of having a Dockerfile, which can be used to create a docker image to run MongoDB containers.

#### 1. Create the empty Dockerfile

Using the nano text editor, let's start editing our Dockerfile:
```shell
$ sudo nano Dockerfile
```

#### 2. Defining our file and its purpose

Although optional, it is always a good practice to let yourself and everybody figure out (when necessary) what this file is and what it is intended to do.

For this, we will begin our Dockerfile with fancy comments (i#) to describe it.
```shell
############################################################
# Dockerfile to build MongoDB container images
# Based on Ubuntu
############################################################
```

#### 3. Setting the base image to use

```shell
# Set the base image to Ubuntu
FROM ubuntu
```


#### 4. Defining the Maintainer (Author)

```shell
# File Author / Maintainer
MAINTAINER Example McAuthor
```

#### 5. Updating the application repository list

**Note:** This step is not necessary, given that we are not using the repository right afterwards. However, it can be considered good practice.

```shell
# Update the repository sources list
RUN apt-get update
```

#### 6. Setting arguments and commands for downloading MongoDB

```shell
################## BEGIN INSTALLATION ######################
# Install MongoDB Following the Instructions at MongoDB Docs
# Ref: http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/

# Add the package verification key
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

# Add MongoDB to the repository sources list
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/mongodb.list

# Update the repository sources list once more
RUN apt-get update

# Install MongoDB package (.deb)
RUN apt-get install -y mongodb-10gen

# Create the default data directory
RUN mkdir -p /data/db

##################### INSTALLATION END #####################
```

#### 7. Setting the default port MongoDB
```shell
# Expose the default port
EXPOSE 27017

# Default port to execute the entrypoint (MongoDB)
CMD ["--port 27017"]

# Set default container command
ENTRYPOINT usr/bin/mongod
```

#### 8. Saving the Dockerfiles

After you have appended everything to the file, it is time to save and exit. Press `CTRL+X` and then "Y" to confirm and save the Dockerfile.

```shell
############################################################
# Dockerfile to build MongoDB container images
# Based on Ubuntu
############################################################

# Set the base image to Ubuntu
FROM ubuntu

# File Author / Maintainer
MAINTAINER Example McAuthor

# Update the repository sources list
RUN apt-get update

################## BEGIN INSTALLATION ######################
# Install MongoDB Following the Instructions at MongoDB Docs
# Ref: http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/

# Add the package verification key
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

# Add MongoDB to the repository sources list
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/mongodb.list

# Update the repository sources list once more
RUN apt-get update

# Install MongoDB package (.deb)
RUN apt-get install -y mongodb-10gen

# Create the default data directory
RUN mkdir -p /data/db

##################### INSTALLATION END #####################

# Expose the default port
EXPOSE 27017

# Default port to execute the entrypoint (MongoDB)
CMD ["--port 27017"]

# Set default container command
ENTRYPOINT usr/bin/mongod
```

#### 9. Building our first image

Using the explanations from before, we are ready to create our first MongoDB image with docker!

```shell
$ sudo docker build -t my_mongodb .
```

* **Note:** The `-t [name]` flag here is used to **tag the image**. To learn more about what else you can do during build, run sudo `docker build --help`.


#### 10. Running a MongoDB instance

Using the image we have build, we can now proceed to the final step: creating a container running a MongoDB instance inside, using a name of our choice (if desired with `-name [name]`).

```shell
sudo docker run -name my_first_mdb_instance -i -t my_mongodb
```

* **Note:** If a name is not set, we will need to deal with complex, alphanumeric IDs which can be obtained by listing all the containers using sudo `docker ps -l`.

* **Note:** To detach yourself from the container, use the **escape sequence** `CTRL+P` followed by `CTRL+Q`.
