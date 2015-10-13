## Docker Architecture
Docker uses a client-server architecture.  The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers. The Docker client and daemon communicate via sockets or through a RESTful API.

![architecture][img1]

In the diagram above, both the Docker **daemon** and client are running on the same system, the same host. But as we are going to see a bit later, we can also connect a Docker client to a remote Docker daemon, or to a daemon that lives on a virtual machine.

The user does not interact directly with the **daemon**, but instead through the Docker **client**, which is just a binary, the command `docker`. The Docker client accepts commands from the user and communicates back and forth with a Docker daemon. Every time we use the command `docker`, we are talking directly to the **client**, which is just an interface to the Docker **daemon**, the one that actually does the real work.

## How things work on OS X
As we said before, Docker client and the daemon can run on the same system, or you can...

[img1]: images/docker_architecture.png
