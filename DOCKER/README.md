# The Basic Anatomy of a Docker Run Command

*   Introduction
*   The Docker Run Command
*   Docker Run Parameters
*   Detached Mode
*   Naming a Container
*   Exposing Container Ports
*   What if I wanted to use different ports than ports 80 and 443?
*   Adding Volumes to a Container
*   Can I bind a folder in a container to a folder on my host machine?
*   Adding Environmental Variables to a Container
*   When a Container Should Restart
*   Conclusion
*   Related posts:

##### Links

*   [installed docker](https://codeopolis.com/posts/quick-and-easy-steps-to-install-docker/)
*   [the basic commands](https://codeopolis.com/posts/25-basic-docker-commands-for-beginners/)
*   [documentation page](https://docs.docker.com/engine/reference/commandline/run/)
*   [docker-compose](https://codeopolis.com/posts/getting-started-with-docker-compose/)
*   [5 simple commands to clean up docker](https://codeopolis.com/posts/simple-commands-to-clean-up-docker/)
*   [The Basic Anatomy of a Docker Run Command](https://codeopolis.com/posts/anatomy-of-a-docker-run-command)
*   [Huge Guide to Portainer for Beginners](https://codeopolis.com/posts/beginners-guide-to-portainer/)
*   [How to Install Plex on a Synology NAS using Docker](https://codeopolis.com/posts/install-plex-on-a-synology-nas-using-docker/)
*   [19 Simple Raspberry Pi Terminal Commands for Beginners](https://codeopolis.com/posts/raspberry-pi-terminal-commands/)

##### Marks

[May 6, 2020](https://codeopolis.com/posts/2020/05/) [Patrick](https://codeopolis.com/author/patski/) [Tutorials](https://codeopolis.com/categories/tutorials/) [0](https://codeopolis.com/posts/anatomy-of-a-docker-run-command/#mh-comments)

Toggle

*   [Introduction](#Introduction "Introduction")
*   [The Docker Run Command](#The_Docker_Run_Command "The Docker Run Command")
*   [Docker Run Parameters](#Docker_Run_Parameters "Docker Run Parameters")
    *   [Detached Mode](#Detached_Mode "Detached Mode")
    *   [Naming a Container](#Naming_a_Container "Naming a Container")
    *   [Exposing Container Ports](#Exposing_Container_Ports "Exposing Container Ports")
        *   [What if I wanted to use different ports than ports 80 and 443?](#What_if_I_wanted_to_use_different_ports_than_ports_80_and_443 "What if I wanted to use different ports than ports 80 and 443?")
    *   [Adding Volumes to a Container](#Adding_Volumes_to_a_Container "Adding Volumes to a Container")
        *   [Can I bind a folder in a container to a folder on my host machine?](#Can_I_bind_a_folder_in_a_container_to_a_folder_on_my_host_machine "Can I bind a folder in a container to a folder on my host machine?")
    *   [Adding Environmental Variables to a Container](#Adding_Environmental_Variables_to_a_Container "Adding Environmental Variables to a Container")
    *   [When a Container Should Restart](#When_a_Container_Should_Restart "When a Container Should Restart")
*   [Conclusion](#Conclusion "Conclusion")

Introduction
------------

In order to get a good understanding of Docker, you must first master the basics. Once you’ve [installed docker](https://codeopolis.com/posts/quick-and-easy-steps-to-install-docker/), and gone over [the basic commands](https://codeopolis.com/posts/25-basic-docker-commands-for-beginners/), it is now time to dive deeper into those commands in order to understand what their function is and what you can do with them. This article will help you understand the basics of one of your most used docker commands. The `docker run` command. It is one of the first commands you learn as a beginner and also one of the most important.

The Docker Run Command
----------------------

![docker run command](https://codeopolis.com/wp-content/uploads/2020/05/docker-run-command.png)

The `docker run` command is a command that you input into terminal in that is specifies the parameters and configuration of a Docker Container that you want to run. The command, at the very least, must specify the image of the container that you want to run.

In addition to the container image you want to run, you can also use it to outline the name of the container, the ports of the container you want to expose, the volumes that you want to use to persist data, and the environmental variables that you want to pass to the container. You can also specify if you want the container to run in attached or detached mode and add additional system capabilities to your container.

Below is an example of a basic `docker run` command with no parameters.

docker run alpine

xxxxxxxxxx

1

1

docker run alpine

When you run this command you will be presented with the following output.

Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
29e5d40040c1: Already exists
Digest: sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54
Status: Downloaded newer image for alpine:latest

xxxxxxxxxx

5

1

Unable to find image 'alpine:latest' locally

2

latest: Pulling from library/alpine

3

29e5d40040c1: Already exists

4

Digest: sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54

5

Status: Downloaded newer image for alpine:latest

Docker searched the host system to determine if it had the alpine image available. It didn’t locate it, so it automatically searched the [docker hub](https://hub.docker.com/) and downloaded the image. After it downloaded the image, it ran the container but because we didn’t specify any commands, the container automatically edited.

Another example would be the ‘hello world’ example that docker provides in its tutorials. You can run the following docker run command:

docker run hello-world

xxxxxxxxxx

1

1

docker run hello-world

After running the above command, you should be presented with the following output:

docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
256ab8fe8778: Pull complete
Digest: sha256:8e3114318a995a1ee497790535e7b88365222a21771ae7e53687ad76563e8e76
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

xxxxxxxxxx

1 docker run hello-world

2 Unable to find image 'hello-world:latest' locally

3 latest: Pulling from library/hello-world

4 256ab8fe8778: Pull complete

5 Digest: sha256:8e3114318a995a1ee497790535e7b88365222a21771ae7e53687ad76563e8e76

6 Status: Downloaded newer image for hello-world:latest

7 Hello from Docker!

9 This message shows that your installation appears to be working correctly.

10

​

11
To generate this message, Docker took the following steps:

12

 1. The Docker client contacted the Docker daemon.

13

 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.

14

    (arm64v8)

15

 3. The Docker daemon created a new container from that image which runs the

16

    executable that produces the output you are currently reading.

17

 4. The Docker daemon streamed that output to the Docker client, which sent it

18

    to your terminal.

19

​

20

To try something more ambitious, you can run an Ubuntu container with:

21

 $ docker run -it ubuntu bash

22

​

23

Share images, automate workflows, and more with a free Docker ID:

24

 https://hub.docker.com/

25

​

26

For more examples and ideas, visit:

27

 https://docs.docker.com/get-started/

Like the output explains, once you ran the command, docker searched the host system for the `hello-world` container image. When it didn’t locate the image, Docker downloaded it from the docker hub. Once the image was downloaded, Docker then ran the container and executed a command that presented the output that you see below. Since it didn’t have any additional commands to run, Docker exited the container.

Now that we’ve got the basic `docker run` command down, let’s start adding some parameters to our command.

Docker Run Parameters
---------------------

Adding parameters to your `docker run` command will greatly expand upon the capabilities of your containers. We will start with naming your container.

### Detached Mode

I wanted to touch based on how to use the `docker run` command to run a container in `detached mode` first. This is because all of our examples will use detached mode in order for you to be able to prevent the container from automatically shutting down then it doesn’t have a command to run. Detached mode is useful for containers that you want to run for a long period of time.

By default, the `docker run` command will run a container in attached mode, if the command doesn’t specify otherwise. Attached mode will “attach” the docker container to your terminal and allow you to view the output of the container as it runs the commands. Basically speaking, it will run the container in the foreground of your terminal, rather than the background.

To run a container in detached mode, you must use the `-d` flag within your `docker run` command. An example of a docker run command with the `-d` command is below:

docker run -d nginx

xxxxxxxxxx

1 docker run -d nginx

The above command will run a Nginx container in detached mode.

### Naming a Container

Naming your container is the best way to keep your Docker host system organized. Using names allows you to easily view what containers are running when running the `docker ps` command or `docker container ls` command in order to list those containers. If you choose not to add names to your containers, docker will assign names to them meaning you would have to identify them by other means.

To add the name of a container, you would need to add the `--name` parameter to your `docker run` command as seen below.

docker run -d \--name=web\_server nginx

xxxxxxxxxx

1

1

docker run -d \--name=web\_server nginx

The above command will run a docker container from the `nginx` image in detached mode and name it `web_server`.

Now, when you run the `docker ps` command, you can see the new `web_server` container running in the background.

### Exposing Container Ports

Exposing ports on a container allows the container to accept connections from the outside world. For example, since we are using the Nginx server in our guide, it is obvious since Nginx is a webserter, that we would want to expose it to ports `80` for `HTTP` traffic and ports `443` for `HTTPS` traffic.

In order to expose these ports, we must use the `-p` flag in our `docker run` command. Following the `-p` flag is the host port and the container port. So, if we wanted to map port `80` on the host to port `80` in the container we would use `-p 80:80`. For example:

docker run -d \--name=web\_server -p 80:80 -p 443:443 nginx

xxxxxxxxxx

1

1

docker run -d \--name=web\_server -p 80:80 -p 443:443 nginx

This will run the `web_server` container from the `nginx` image. It will run the container in `detached` mode and expose ports `80` and ports `443` and bind those ports to port `80` and port `443` in the within the container. You will now be able to communicate with this server by visiting `http://YOURHOSTIP` or `https://YOURHOSTIP` in a web browser.

#### What if I wanted to use different ports than ports `80` and `443`?

Since the container uses port `80` and port `443`, we can easily use a docker run command to map different ports on the host machine to these ports. Lets map port `8080` on the host to port `80` within the container. In addition, lets map port `4443` on the host to port `443` within the container. In order to do that, we would use the same `-p` flag. For example:

docker run \-d \--name=web\_server \-p 8080:80 \-p 4443:443 nginx

xxxxxxxxxx

1

1

docker run \-d \--name=web\_server \-p 8080:80 \-p 4443:443 nginx

The above command ran the `web_server` container from the `nginx` image. It is running the container in detached mode and will expose port `8080` on the host and map it to port `80` within the container. In addition, it will expose port `4443` on the host to port `443` within the container. You will now be able to communicate with this server by visiting `http://YOURHOSTIP:8080` or `https://YOURHOSTIP:4443` in a web browser.

### Adding Volumes to a Container

Currently, if we would remove our Nginx web\_server docker container, we would lose all of our data because we didn’t create a volume for the data to persist in between containers. In order to persist data within our container, we need to map a volume to the container. We can do that with the `-v` flag.

In the below example, we are going to use the `-v` flag to mount a volume on our system in order to locally store the data in the `www` folder of our `web_server` container. The volume we will use is called `www`, the volume was created by using the `docker volume create` command. We will bind the www volume to the `/usr/share/nginx/html` folder within the web\_server container.

docker run -d --name=web\_server -p 8080:80 -p 4443:443 -v www:/usr/share/nginx/html nginx

xxxxxxxxxx

1

1

docker run -d --name=web\_server -p 8080:80 -p 4443:443 -v www:/usr/share/nginx/html nginx

> In order to organize our commands from this point forward in this guide, I am going to use the backslash at the end of each parameter to split the commands up in to new lines. A backslash stops the next character in a command from being interpreted by the shell. If the next character is a new line, then the new line will not be interpreted as the end of the command by the shell. It effectively allows a command to span multiple lines, making the command easier to organize. The above `docker run` command can also be written as follows:

docker run -d \\
\--name=web\_server \\
\-p 8080:80 \\
\-p 4443:443 \\
\-v www:/usr/share/nginx/html \\
nginx

xxxxxxxxxx

6

1

docker run -d \\

2

\--name\=web\_server \\

3

\-p 8080:80 \\

4

\-p 4443:443 \\

5

\-v www:/usr/share/nginx/html \\

6

nginx

The above command ran the `web_server` container from the `nginx` image. It is running the container in detached mode and will expose port `80` on the host and map it to port `80` within the container. It will also expose port `443` on the host to port `443` within the container. In addition it will bind a volume mapping from the `www` docker volume to the `/user/share/nginx/html` folder within the container.

#### Can I bind a folder in a container to a folder on my host machine?

Instead of using docker volumes, you can use the `docker run` code to bind a folder within a container to a folder on your host machine. So, lets say you want to use the `/home/ubuntu/html` folder on your host system to house the files in your Nginx html folder located within the container at `/user/share/nginx/html`. All you have to do is use `-v /home/ubuntu/html:/user/share/nginx/html` within your docker run command as follows:

docker run -d \\
\--name=web\_server \\
\-p 80:80 \\
\-p 443:443 \\
\-v /home/ubuntu/html:/usr/share/nginx/html \\
nginx

xxxxxxxxxx

6

1

docker run -d \\

2

\--name\=web\_server \\

3

\-p 80:80 \\

4

\-p 443:443 \\

5

\-v /home/ubuntu/html:/usr/share/nginx/html \\

6

nginx

The above command ran the `web_server` container from the `nginx` image. It is running the container in detached mode and will expose port `80` on the host and map it to port `80` within the container. It will also expose port `443` on the host to port `443` within the container. In addition it will bind a volume mapping from the `/home/ubuntu/html` folder on the host machine to the `/user/share/nginx/html` folder within the Docker container.

Using this method allows us to easily modify the files that are used by the container in the `/home/ubuntu/html` folder.

### Adding Environmental Variables to a Container

Some containers will allow you to use environmental variables to make configuration changes within the container. For example, in the Nginx container you are able to use the `HOST` environmental variable to specify your host. In order to add an environmental variable to our `docker run` code you will use the `-e` flag.

The list of environmental variables that you can use with a container image is usually found in the location in which you obtained the container, for example the [docker hub](https://hub.docker.com/) or [Github](https://github.com/). Some container images can have many environmental variables that are required in the `docker run` command in order for them to run properly.

In order to configure the container with the host, we will use `NGINX_HOST` as the variable (as specified in the instructions in location in which you obtain the container image) and `yourhost.com` as the variable.

docker run -d \\
\--name=web\_server \\
\-p 80:80 \\
\-p 443:443 \\
\-v /home/ubuntu/html:/usr/share/nginx/html \\
\-e NGINX\_HOST=yourhost.com \\
nginx

xxxxxxxxxx

7

1

docker run -d \\

2

\--name\=web\_server \\

3

\-p 80:80 \\

4

\-p 443:443 \\

5

\-v /home/ubuntu/html:/usr/share/nginx/html \\

6

\-e NGINX\_HOST=yourhost.com \\

7

nginx

The above command ran the `web_server` container from the `nginx` image. It is running the container in detached mode and will expose port `80` on the host and map it to port `80` within the container. It will also expose port `443` on the host to port `443` within the container. In addition it will bind a volume mapping from the `/home/ubuntu/html`folder on the host machine to the `/user/share/nginx/html` folder within the Docker container. Next, it added `yourhost.com` to the `nginx.template` file which contained `NGINX_HOST` as a variable.

### When a Container Should Restart

What happens to your containers when the host machine restarts? That is up to you. By default, the containers do not start up when the host machine comes online. If you would like to change the default settings to configure under what condition that you would like the container to restart, then you must use the `--restart` flag in your `docker run` command.

Your restart options are the default option, or `--restart no` which is to not have the container restart automatically if the container should exit or the host system should restart. The next option is to have your container restart only on a container failure using the `--restart on-failure` flag. You can also have your container restart under all conditions using the `--restart always` flag. Finally, you can set your container to restart always unless you manually stop the container. This is accomplished using the `--restart unless-stopped` flag.

Since we are using the Nginx web server as our example in this guide, we will set our web server to restart always unless we manually stop it. That ensures that our web server always tries to make itself available when the docker host restarts or the docker daemon restarts for whatever reason. This can be accomplished by using the following command:

docker run -d \\
\--name=web\_server \\
\-p 80:80 \\
\-p 443:443 \\
\-v /home/ubuntu/html:/usr/share/nginx/html \\
\-e NGINX\_HOST=yourhost.com \\
\--restart unless-stopped \\
nginx

xxxxxxxxxx

8

1

docker run -d \\

2

\--name\=web\_server \\

3

\-p 80:80 \\

4

\-p 443:443 \\

5

\-v /home/ubuntu/html:/usr/share/nginx/html \\

6

\-e NGINX\_HOST=yourhost.com \\

7

\--restart unless-stopped \\

8

nginx

The above command ran the `web_server` container from the `nginx` image. It is running the container in detached mode and will expose port `80` on the host and map it to port `80` within the container. It will also expose port `443` on the host to port `443` within the container. In addition it will bind a volume mapping from the `/home/ubuntu/html`folder on the host machine to the `/user/share/nginx/html` folder within the Docker container. Next, it added `yourhost.com` to the `nginx.template` file which contained `NGINX_HOST` as a variable. Finally, the container is set to automatically restart always, unless it is stopped

Conclusion
----------

Hopefully this guide helped you learn the basic anatomy of a `docker run` command. You can now use your newfound skills to run various Docker containers on your docker host. If you would like a more information on more advanced uses of the `docker run` command you can check out the docker [documentation page](https://docs.docker.com/engine/reference/commandline/run/) on the subject. After you are comfortable with the `docker run` command, you can begin to explore starting services on your docker host with the [docker-compose](https://codeopolis.com/posts/getting-started-with-docker-compose/) command.

If you followed along with this guide, your docker host may have some dangling images, containers, and volumes that are no longer needed. In order to remove all of these containers and images you can learn the [5 simple commands to clean up docker](https://codeopolis.com/posts/simple-commands-to-clean-up-docker/). This guide will help you learn the commands that you can use to quickly clean up your images, container, and volumes that were created over the course of this guide.

If you have any questions about this article or feel that something should be added, please feel free to leave a comment below.

_The post, [The Basic Anatomy of a Docker Run Command](https://codeopolis.com/posts/anatomy-of-a-docker-run-command), first appeared on [Codeopolis](https://codeopolis.com/)._

Related posts:
--------------

1.  [Huge Guide to Portainer for Beginners](https://codeopolis.com/posts/beginners-guide-to-portainer/ "Huge Guide to Portainer for Beginners")
2.  [How to Install Plex on a Synology NAS using Docker](https://codeopolis.com/posts/install-plex-on-a-synology-nas-using-docker/ "How to Install Plex on a Synology NAS using Docker")
3.  [25 Basic Docker Commands for Beginners](https://codeopolis.com/posts/25-basic-docker-commands-for-beginners/ "25 Basic Docker Commands for Beginners")
4.  [19 Simple Raspberry Pi Terminal Commands for Beginners](https://codeopolis.com/posts/raspberry-pi-terminal-commands/ "19 Simple Raspberry Pi Terminal Commands for Beginners")

*   [Docker](https://codeopolis.com/tags/docker/)
