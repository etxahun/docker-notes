# docker-notes
My very own Docker notes.

### Table of Contents
**[Introducción](#Introducción)**<br>
**[Instalación](#Instalación)**<br>


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
