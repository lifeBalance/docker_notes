## Images vs Containers
A Docker **image** is a *read-only template*. For example, an image could contain an Ubuntu operating system with Apache and your web application installed. Images are used to create Docker **containers**. Docker images are the **build component** of Docker.

Docker **containers** are similar to a directory. A Docker container holds everything that is needed for an application to run. Each container is created from a Docker image. Docker containers can be run, started, stopped, moved, and deleted. Each container is an isolated and secure application platform. Docker containers are the **run component** of Docker.

## Images
There are two ways of using Docker images:

* We can build our own new images using a `Dockerfile`.
* Or we can download Docker images that other people have already created.

## Building our own images
Each image consists of a series of **layers**, that's one of the reasons Docker is so lightweight. Every image starts from a **base image**, for example a base Ubuntu image. Then, we can add layers on top of that base image using a simple, descriptive set of steps we call **instructions**. Each instruction creates a new layer in our image. These instructions are stored in a file called a `Dockerfile`.

> To create these **layers**, Docker uses a technology called [UnionFS][1]. Actually, Docker make use of several **union file system** variants including: AUFS, btrfs, vfs, and DeviceMapper.

A `Dockerfile` is a text document that contains all the instructions needed to build an **image**. Traditionally, the `Dockerfile` is called `Dockerfile` and is located at the root of the context. But we can use the `-f` flag with `docker build` to point to a `Dockerfile` anywhere in our file system.

Once we have finished our `Dockerfile`, we can build our image from it using the command:

```
$ docker build .
```
When we run this command, Docker reads the `Dockerfile`, executes the instructions, and returns a final image. This **image** is based on the `Dockerfile` and its **context**.

> A **context** is the files at a specified location on our local system(PATH), or a location at a git repository(URL).

A context is processed recursively, so a `PATH` includes any subdirectories, and a `URL` includes the repository and its submodules. To increase the build’s performance, we can exclude files and directories by adding a `.dockerignore` file to the context directory.

## Downloading pre-built Images
We have seen how we can create our own images using `Dockerfile` and a context, but we can also download pre-built images from the [Docker hub][2]. There we can find **images** that members of the docker community have made available for free.

It's also possible to **search** for images from the command line. For example, let's search for an Ubuntu image:

```
$ docker search ubuntu
...
```

Once we have decided which **image** we want, downloading it it's as easy as doing:
```
$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
48731f0a6276: Downloading [=====================>    ] 44.07 MB/65.69 MB
9302827ed0a5: Download complete
...
```

## Managing images
Once we have several images in our local registry, either created by ourselves or downloaded from Docker hub, we have available a series of commands for managing them:

* We have a command for **listing** images:
```
$ docker images
REPOSITORY      TAG         IMAGE ID         CREATED         VIRTUAL SIZE
ubuntu          latest      cdd474520b8c     2 days ago      188 MB
```

* Another one for **deleting** them:
```
$ docker rmi <image_name>
```

## Containers
A container consists of an operating system, user-added files, and meta-data. As we’ve seen, each container is built from an **image**. That image tells Docker what the container holds, what process to run when the container is launched, and a variety of other configuration data. The Docker image is **read-only**. When Docker runs a container from an image, it adds a **read-write** layer on top of the image in which your application can then run.

### How to run a container
Let's see how to launch a container from an image:

```
$ docker run -i -t ubuntu /bin/bash
root@779818aa34a0:/#
```
Let’s break down this command:
* First, we are using the `docker` command with the `run` option so the Docker **client** tells the Docker **daemon** to launch a new container.
* The `ubuntu` argument is the base image to build the container from, in this example a base Ubuntu image. This is the bare minimum we need to run a container.
The Docker daemon checks for the existence of the Ubuntu image in the local registry, and if it doesn't find it, it downloads it from Docker hub.
* `/bin/bash` is the command you want to run inside the container when it is launched. Here we are also using the options `-i` and `-t`, which stand for *interactive* and *tty* respectively, meaning we are going to start a Bash interactive session in a pseudo-tty that attaches `stdin` and `stdout`, inside the new ubuntu container. To exit this session type `exit` and you'll be return back to the host environment.

> When we use the `exit` command the **first time** a container is run, the container will dissapear. To avoid this, use the **escape sequence** `Ctrl-p` + `Ctrl-q` and the container will continue to exist in a running state.


Once we have a running container, we can start another interactive shell session using:

```
$ docker exec -it jovial_almeida /bin/bash
```

> We can also use the **id** of the container, instead of its name, but names are easier to  remember.

When we exit this session using `exit` or the **escape sequence** (`Ctrl-p` + `Ctrl-q`), we should have a running container, which we can check using `docker ps`.

Why would you want to start an interactive shell session in a container? It is the first step for modifying a container from inside and save it as a customized image. More about this later.

Check this https://docs.docker.com/articles/basics/


## Managing containers
* Listing **running** containers:
```
$ docker ps
```

* Listing **all** containers:
```
$ docker ps -a
```

* To **stop** a running container without deleting it:
```
$ docker stop <container_name>
```

* To **start** a stopped container:
```
$ docker start <container_name>
```
Also available is the `docker restart` command that runs a stop and then start on the container.

* Once a container is stopped, we can go ahead and **delete** it using:
```
$ docker rm <container_name>
```

>  Always remember that removing a container is final!

* The `docker inspect <container_name>` command returns a JSON document containing useful configuration and status information for the specified container.

> In all of these cases we can use the <container_id> instead of its name.

Sometimes we don't need all the information provided by the `docker inspect` command, and we can use the `--format`(`-f` for short) option to limit the intel to any field of the JSON document. For example:

```
$ docker inspect -f {{.NetworkSettings.IPAddress}} <container_name>
```

> The surrounding curly braces (`{{<.FieldName>}}`) are part of the [Go language][3] [templating system][4]. Remember that Docker is written in Go. The **fields** in the JSON object are accessed common **dot notation**, used in a lot of programming languages.


---
[1]: https://en.wikipedia.org/wiki/UnionFS
[2]: https://hub.docker.com/
[3]: https://golang.org/
[4]: https://golang.org/pkg/text/template/
