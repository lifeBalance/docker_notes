# Docker on OS X

## Docker Machine
Once we have installed Docker using Docker Toolbox, we have available the following Docker tools:

* Docker Machine for running the docker-machine binary
* Docker Engine for running the docker binary
* Docker Compose for running the docker-compose binary
* Kitematic, the Docker GUI
* A shell preconfigured for a Docker command-line environment
* Oracle VM VirtualBox

By default, the standard **Docker Toolbox** installation installs binaries for the Docker tools in `/usr/local/bin`, making these binaries available to all users. Any existing VirtualBox installation is automatically updated, that's why it we have VirtualBox running, we have to stop it before running the **Docker Toolbox** installer.

**Docker Machine** lets you create Docker hosts on your computer, on cloud providers, and inside your own data center. It automatically creates a virtual linux host, installs Docker on it, then configures the docker client to talk to the daemon.

## A linux virtual machine
Since Docker runs natively on Linux, its use in other platform requires the use of **virtualization**. Let's take a look at the Docker system running on a Linux host:

![docker on linux][img1]

As you can see, on a typical Linux installation, the Docker **client**, the Docker **daemon**, and any containers run directly on your localhost. This means you can address ports on a Docker container using standard localhost addressing such as `localhost:8000` or `0.0.0.0:8376`.

Again, because the **Docker daemon** uses Linux-specific kernel features, you can’t run Docker natively in **OS X**. Instead, we must use a **Linux Virtual Machine** that hosts Docker when using Mac. Here, take a look:

![docker on mac][img2]

In an OS X installation, the docker daemon is running inside a Linux VM called **default**. The default virtual machine is a lightweight Linux VM made specifically to run the Docker daemon on Mac OS X. This VM runs completely from RAM, is a small ~24MB download, and boots in approximately 5s.

> Virtual machines are kept in `~/.docker/machine/machines`

In OS X, the **Docker host address** is the address of the Linux VM. When you start the VM with `docker-machine` it is assigned an IP address. When you start a container, the ports on a container map to ports on the VM.

## Running containers in OS X
So, if we are on a Mac, the workflow of running a Docker container is a bit different to a native Docker installation:

1. First, we have to create a new (or start an existing) Docker virtual machine.
2. Switch our environment to this virtual machine.
3. Only then we can start using the **Docker client** to create, load, and manage containers.

Once we create a machine, we can reuse it as often as we like. Like any VirtualBox VM, it maintains its configuration between uses.

### Creating or starting a Docker Machine
There are a couple of ways for creating a new Docker machine:

#### Using the Docker Quickstart Terminal
1. Open the “Applications” folder or the “Launchpad”.

2. Find the Docker Quickstart Terminal and double-click to launch it. It will open a new terminal window and create a virtual machine called `default` if it doesn’t exists, or start the `default` VM if it does.

One way to verify that everything is working is by running the *hello-world* **container**:

```
$ docker run hello-world

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```

As you can read in the output, to run this container, the Docker daemon pulled the "hello-world" **image** from the Docker Hub.

> You can check the "hello-world" image on [Docker hub][1].

Then, the Docker daemon created a new **container** from that image, and run the executable in the container that produced the "Hello from Docker" message.

### Using our shell
A more typical way to interact with the Docker tools is from your regular shell command line. To create a **Docker Machine** using our shell, we would use the conveniently named `docker-machine` command.

This command accepts a lot of options. Maybe the first step should be listing the docker machines we have installed:

```
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM
default   *        virtualbox   Running   tcp://192.168.99.100:2376   
```
The output above shows us one machine named **default** running. Let's go over it in more detail:

* The `NAME` label is self explanatory.
* `ACTIVE` means that if we have running several virtual machines, only one of them is **active** at any given point, meaning that we can interact with its environment.
* The `DRIVER` label in this case is `virtualbox`. The **driver** represents the virtual environment. For example, on a local Linux, Mac, or Windows system the driver is typically [Oracle Virtual Box][2]. For cloud providers, Docker Machine supports drivers such as AWS, Microsoft Azure, Digital Ocean and many more. A complete list of the **supported drivers** is available [here][3].

> We have to specify a **driver** when creating a Docker machine.

* The `STATE` is *Running*. We can check the status of a machine using the `status` option and passing in the name of the machine, for example:

```
$ docker-machine status default
Running
```
* The `URL` field is really important. As we saw before, the ports on a container map to ports on the virtual machine, so if we want to access a containerized webapp on our browser, we will have to use the **IP** of the VM.
We can also get this address using:
```
$ docker-machine ip default
192.168.99.100
```

### Start, stop, restart a machine
We can **stop** a running machine doing:
```
$ docker-machine stop default
```

And check its **state** running:
```
$ docker-machine status default
Stopped
```
To **start** a currently stopped machine:
```
$ docker-machine start default
Starting VM...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```
As the above output says, every time we start(or restart) a machine, its **IP address** may have changed, that's why it's convenient to check its environment running:
```
$ docker-machine env default
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/javi/.docker/machine/machines/default"
export DOCKER_MACHINE_NAME="default"
# Run this command to configure your shell:
# eval "$(docker-machine env default)"
```

### Deleting and creating machines
As easy as:
```
$ docker-machine rm default
Successfully removed default
```

Now if we try to list our machines, we got nothing:
```
$ docker-machine ls
NAME   ACTIVE   DRIVER   STATE   URL   SWARM
```
Let's create another **default machine**:
```
$ docker-machine create --driver virtualbox default
Creating VirtualBox VM...
Creating SSH key...
Starting VirtualBox VM...
Starting VM...
To see how to connect Docker to this machine, run: docker-machine env default
```

Once the machine is created, it's also automatically started but it's not **active**:
```sh
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM
default            virtualbox   Running   tcp://192.168.99.100:2376   
```

## Connect to a Docker machine
To make a runnning machine **active** all we have to do is:
```sh
$ eval "$(docker-machine env default)"
```
Now, if we list our machines again, we can see which one is active:
```
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM
default   *        virtualbox   Running   tcp://192.168.99.100:2376   
```

## Other Docker Machine commands
So far, we have seen Docker Machine commands for doing basic operations with machines:

* start, stop, and restart machines
* listing, create and remove them

But Docker Machine supplies a lot more subcommands for managing them. Check the whole list [here][4].

Let's see some more of them

* upgrade the Docker client and daemon
* configure a Docker client to talk to your host

** FINISH IT! **

[img1]: images/linux_docker_host.png
[img2]: images/mac_docker_host.png
[1]: https://hub.docker.com/_/hello-world/
[2]: https://www.virtualbox.org/
[3]: https://docs.docker.com/machine/drivers/
[4]: https://docs.docker.com/machine/reference/
