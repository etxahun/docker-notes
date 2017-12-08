# docker-notes
My very own Docker notes.

### Table of Contents
**[Introducción](#introducción)**<br>
**[Instalación](#instalación)**<br>
**[Containers](#containers)**<br>
**[Ports in Containers](#ports-in-containers)**<br>
**[Images](#images)**<br>
**[Limpieza](#limpieza)**<br>
**[Ciclo de vida de un contenedor (Create/Start/Stop/Kill/RM)](#ciclo-de-vida-de-un-contenedor-(create/start/stop/kill/rm))**<br>
**[Dockerfiles](#dockerfiles)**<br>
**[Volumenes](#volumenes)**<br>
**[Networking](#networking)**<br>


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

      * Aquí simplemente se abre una **tty** en modo **interativo**. Podrían hacerse otras cosas como cambiar el _working directory_, definir variables de entorno, etc. La lista completa puede verse acá

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

## Ports in Containers
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

  ## Crear nuestras propias Images:

  Después de crear varios contenedores, tal vez queremos crear nuestras propias imágenes.

  Cuando arrancamos un contenedor, este se inicia desde una imagen base. Una vez con el contenedor en ejecución nosotros podríamos querer hacerle cambios, como por ejemplo, instalarle ciertas librerías o dependencias (ejemplo, correr `apt install htop vim git` dentro de un contenedor que tiene de imagen base, Ubuntu). Una vez ejecutado este comando, el contenedor ha modificado su *filesystem*. Si nuestra idea es querer ejecutar en un futuro contenedores iguales al anterior, tendremos que crear una *imagen* de este. Docker nos provee del comando *commit* para, a partir de un contenedor, crear una imagen. Docker mantiene las diferencias entre la imagen base y la que se quiere crear, creando una nueva layer usando **UnionFS**. Similar a **GIT**.

  1. Crearemos un contenedor de Ubuntu, y al mismo le actualizaremos la lista de repositorios.
```shell
$ docker run -t -i --name=contenedorPrueba ubuntu:14.04 /bin/bash
root@69079aaaaab1:/# apt update
```

    Cuando salgamos de este contenedor, se detendrá, pero seguirá estando disponibles a menos que lo eliminemos explícitamente con `docker rm`.

  2. A continuación haremos un `docker commit`, para definir la nueva imagen para mantener una imagen mas actualizada.
```shell
$ docker commit contenedorPrueba ubuntu:update
13132d42da3cc40e8d8b4601a7e2f4dbf198e9d72e37e19ee1986c280ffcb97c
$ docker images
REPOSITORY    TAG     IMAGE ID      CREATED          VIRTUAL SIZE
ubuntu        update  13132d42da3c  5 days ago  ...  213 MB
```
  **NOTA:** Esto `ubuntu:update` especifica ``<nombre_imagen>:<tag_del_commit>`.

  Luego ya podremos lanzar contenedores basados en la nueva imagen `ubuntu:update`.

  ### Adicional

  Podemos chequear las diferencias con `docker diff`:
  ```shell
  $ docker diff contenedorPrueba
  C /root
  A /root/.bash_history
  C /tmp
  C /var
  C /var/cache
  C /var/cache/apt
  D /var/cache/apt/pkgcache.bin
  D /var/cache/apt/srcpkgcache.bin
  C /var/lib
  C /var/lib/apt
  C /var/lib/apt/lists
  ```

# Limpieza
* Borrar container:
```shell
$ docker rm <container_ID>
```

* Borrar TODOS los containers:
```shell
$ docker rm $(docker ps -a -q)
```

* Borrar imágenes:
```shell
$ docker rmi <container_ID>
```

* Borrar TODAS las imáganes:
```shell
$ docker rmi $(docker images -q)
```

# Ciclo de vida de un contenedor (Create/Start/Stop/Kill/RM)

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

## Detener Containers:

Siguiendo el ejemplo, para **detener** el contenedor se puede ejecutar cualquiera de los siguientes comandos `kill` ó `stop`:
```shell
$ docker kill a842945e2414 (envía SIGKILL)
$ docker stop a842945e2414 (envía SIGTERM).
```

Asimismo, pueden reiniciarse (hace un `docker stop a842945e2414` y luego un `docker start a842945e2414`):

```shell
$ docker restart a842945e2414
```

o destruirse:

```shell
$ docker rm a842945e2414
```

# Dockerfiles

# Volumenes

# Networking
