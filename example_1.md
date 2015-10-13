## First example
Let's put together all the concepts we've seen by building a docker project. To begin with, we are going to create a folder for our project and change into it:

```
$ mkdir first_docker_project
$ cd first_docker_project
```
### Context
This folder is going to be the **context** for our image. Inside it we are going to create a simple shell script and save it with the name of `counter.sh`:

```
$ vim counter.sh
```
These are going to be the contents of the script:

```bash
#!/usr/bin/env bash

while true
do
  X=$[$X+1]
  echo $X
  sleep 1
done
```
It's just a script that outputs a counter to the `stdout`.

### Dockerfile
Now, inside the same folder let's create a simple `Dockerfile`:

```
FROM ubuntu

MAINTAINER Bart Simpson <bart@springfield.com>

COPY counter.sh /usr/local/counter.sh

RUN chmod +x /usr/local/counter.sh

CMD ["/usr/local/counter.sh"]
```

Let's go over line by line:

* In the first line, we are using the **instruction** `FROM` and passing [ubuntu][1] as our **base image**.
* The second instruction specifies the `MAINTAINER` of the image, in case we wanted to upload it to [Docker hub][2].
* In the third line, the `COPY` instruction is creating a copy of our script inside the image filesystem.
* Next we are changing the permits of this file on the image soon to be container filesystem.
* Lastly, the `CMD` instruction executes the script once the container starts running.

### Building the image
From the `Dockerfile` and the **context** of this project, in this case a simple shell script, we are going to build a Docker image using the command:

```sh
$ docker build -t simple_bash_counter .
Sending build context to Docker daemon 3.072 kB
Step 0 : FROM ubuntu
---> cdd474520b8c
... # more steps here
Successfully built 9976ffd720c3
```
In this command we are passing the `build` option to the docker **client** passing also a **tag** for the image (`-t` option) that will be used to identify it. The tag name is `simple_bash_counter`. The **dot** at the end signifies the **context**, in this case the current directory.

When we press enter, we sent the build context to the **daemon** and the build process starts, executing the **instructions** of the `Dockerfile`, which are output as **steps**  to `stdout` upon completion. Hopefully at the end of the process we are rewarded with the line that indicates a successful build.

To check that our new image exists we can run the command `docker images`, and our new image should be there.

### Running a container
Ok, let's spin up a container out of our image. Let's start running `docker run` in its most basic form:

```sh
$ docker run simple_bash_counter
1
2
3
... # Counting in an infinite loop.
```
A disadvantage of running a container like this, it's that we've lost control over our terminal. The only way to regain control is:

* Open a new terminal tab and load the Docker machine's environment using:
```sh
$ eval "$(docker-machine env default)"
```
* List the running containers doing `docker ps` and stop the one using its id or name with `docker stop`. It takes some time before the container is stopped, this is a **grace period** of about ten seconds by default. You can change it using `--time=15` for example.

> We could have avoid that using the `--detach` option (`-d`), which runs the container in the background.

### Starting/stopping the container
As a result of running the `docker stop` command in our container, doing `docker ps` would not list it(because it's not running), but the container still exists, we just have to use the `-a` option to list stopped containers. There's no need to spin up new containers indefinitely, we can start/stop them as we need.

We saw before than the `docker run` command, starts a container in an interactive way. The `docker start` command starts a container in a non-interactive way, so we don't lose control of our terminal. So how do we know the container is running?

> To start a container in **interactive** mode use the `-i` option.

### The log file
We can use the `docker logs` command to see the output(if any) of our running container. In our example:

```sh
$ docker logs stoic_cori
1
2
3
...
1
2
...
```
The container's log file shows the output of all the times we have run it. Each time we run the container, the output is **appended** to the log file.

### Deleting a container
Once we are done playing around with our toy container we cand get rid of it. Before that, make sure that the **status** of the container is `Exited` (not running) using the `docker ps -a` command.

Or you can go ahead and use the `-f` option, to force its removal without even stopping it before:

```sh
$ docker rm -f stoic_cori
stoic_cori
```
This command returns the removed container. You can run `docker ps -a` to check that it's not in the list anymore.
