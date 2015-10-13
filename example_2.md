## Second example
Let's see another basic Docker project, in this occasion we are going to run a web server from a docker container; [nginx][1] sounds like a good option. Do you remember how to search for images?:

```sh
$ docker search nginx
NAME         DESCRIPTION                   STARS     OFFICIAL
nginx        Official build of Nginx.      1456      [OK]
...
```
We could have also point our browser to Docker hub, and search for the [nginx image][2].

Either way, let's **pull** the image:

```sh
$ docker pull nginx
```

> It is possible **pull** and **run** the image all in one command: `docker run`. If the image doesn't exist in our local registry, it will be automatically pulled from Docker hub.

And **run** a container out of it:
```sh
$ docker run -d --name ngin_tonic nginx
```

Now, let's find out the `IP` of the container:
```sh
$ docker inspect -f {{.NetworkSettings.IPAddress}} ngin_tonic
172.17.0.23
```

And point our browser to that IP address or use the `curl` command:
```sh
$ curl http://172.17.0.23
```

But `curl` sits there idle, if we waited enough we would get an `ERR_CONNECTION_TIMED_OUT`.

Something's wrong, let's inspect the **ports** in the container:
```sh
$ docker inspect -f {{.NetworkSettings.Ports}} ngin_tonic
map[443/tcp:[] 80/tcp:[]]
```

Nothing special. Let's try again `curl`:
```sh
$ curl http://172.17.0.23:80
```

Still not working. If we were in **Linux**, this would be working, because remember, Docker runs natively in Linux. But since we are running Docker on a Linux virtual machine on **OS X**, we have to use the IP address of the virtual machine.

### Mapping ports
For this to work we also have to map the ports on the virtual machine to the ports in the container. We have to do that when we spin up the container, using the `-P` option, so let's stop and delete the `nginx_tonic` container and run a new one with the same name:
```sh
$ docker run -dP ngin_tonic nginx
```

The `-P` option tells Docker to map those ports to ports on the Docker host that are randomly selected from the range between 49153 and 65535. The port mappings are dynamic and are set each time the container is started or restarted.

We can check the current port mapping running:
```sh
$ docker inspect -f {{.NetworkSettings.Ports}} ngin_tonic
map[443/tcp:[{0.0.0.0 32768}] 80/tcp:[{0.0.0.0 32769}]]
```

As we can see, the ports that the nginx container exposes, (443 and 80) are mapped to the ports in the host. A diagram can be useful in this case:

![port mappings][img1]

Now, if we it the proper address with curl (or with the browser) everything should work properly:
```sh
$ curl http://$(docker-machine ip default):32771
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

And indeed it works. Note how we are using the output of `docker-machine ip` as part of the argument to the `curl` command.

** FINISH IT! **

## Another example
Building up on the two last examples, this time we are gonna tackle on a slightly more difficult Docker project. It's going to involve several **containers** interacting with each other:

* A **container** for running our shell script, with a small modification: we are going to redirect its output to another container.
* A **data container**, where we are gonna write the output of our script in an HTML file.
* Finally, a **container** running the [nginx container][1], which is going to serve data from the data container.

### The counter script
We are going to use the same `Dockerfile` we used in the last example. This time we are just going to modify the script itself; these are the contents:

** FINISH IT! **


[1]: http://nginx.org/
[2]: https://hub.docker.com/_/nginx/
[img1]: images/port_mappings.png
