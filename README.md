Docker workshop
===============

What is Docker?
---------------

Docker is an **open-source** project that **automates** the **deployment** of applications inside software **containers**. It has very developed eco-system to make, distribute and handle containers. Also it’s very well documented.

Docker uses the resource isolation features of the Linux kernel such as _cgroups_ and kernel _namespaces_, and a _union-capable file system_ such as OverlayFS and others to allow independent "containers" to run within a single Linux instance, avoiding the overhead of starting and maintaining virtual machines.

Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run: code, runtime, system tools, system libraries – anything that can be installed on a server. This guarantees that the software will always run the same, regardless of its environment.

Link: [What is Docker](https://www.docker.com/what-docker#/overview)

Why do we need it?
------------------

* Faster development process (using 3rd parties like PostgreSQL, Redis, Elasticsearch, whatever).
* Handy application encapsulation (you can deliver your application in one piece).
* Same behaviour on local machine / dev / stage / production servers.
* Easy and clear monitoring.
* Easy to scale (If you’ve done your application right it will be ready to scaling even not only in Docker).

What is the difference from virtualization?
-------------------------------------------

**Virtual machines** include the application, the necessary binaries and libraries, and an entire guest operating system - all of which can amount to tens of GBs.

**Docker containers** include the application and all of its dependencies - but share the kernel with other containers, running as isolated processes in user space on the host operating system. Docker containers are not tied to any specific infrastructure: they run on any computer, on any infrastructure, and in any cloud.

Link: [Docker containers and VMs](https://www.docker.com/what-container#/virtual_machines)

Supported platforms
-------------------

Because Docker is based on features of Linux kernel the only native platform is Linux. But there a lot native applications for other platforms such as MacOS and Windows. For this platforms Docker application is encapsulated into tiny virtual machine with Linux. The usability of this applications now is on very high level. Also there a lot of supplementary apps such as [Kitematic](https://kitematic.com/) or [Docker Machine](https://docs.docker.com/machine/overview/) which helps to install Docker on platforms different from Linux and operate with it.

Link: [Docker components](https://docs.docker.com/#components)

Installation
------------

Link: [Installation](https://docs.docker.com/engine/getstarted/step_one/)

Terminology
-----------

**Container** - running instance with required application. Containers are always created from images. Container could expose ports and  volumes to interact with other containers or outer world. Container could be easily killed / removed and re-created again in a very short time.

Link: [About containers](https://www.docker.com/what-container#/package_software)

**Image** - basic element for every container. Uses copy-on-write model. During building images every step is cached and could be reused. Building images could take some time - it depends on particular image. While containers can be started from images very quickly.

Link: [Dockerfile - best practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

**Port** - is a TCP/UDP port in it’s classical meaning. There are a lot of modes for networking in Docker containers, so to make things simpler let’s assume that ports could be exposed to outer world (accessible from host OS) or connected to other containers - accessible only from that containers and invisible from outer world.

**Volume** - the best explanation is shared folder. Volume keeps data outside of its container but accessible from this container or other connected containers. Data stored in volumes is persistent.

Link: [Managing data in Docker volumes](https://docs.docker.com/engine/tutorials/dockervolumes/)

**Registry** - server that keeps images. Could be compared with git - you can pull image from registry to deploy it locally and you can push locally built images to registry (create new image or update image version on registry). Docker registry application is Open Source app like the main application - so you could deploy your private Docker Registry on any server you want.

Link: [Registry](https://docs.docker.com/registry/)

**Docker hub** - is a registry with web-interface. It keeps a lot of Docker images with different software. There are “official” Docker images (made by Docker team or in cooperation with original software vendors - don’t assume that it’s always official software vendors). It’s free to register and has payed options. You can have one private image per account and infinite amount of public images for free.

Link: [Docker Hub](https://hub.docker.com/explore/)

Preparations
------------

If you're running Docker on Linux you need to run all following commands as `root` or you can add your user to `docker` group and re-login to make changes apply:

```
sudo usermod -aG docker `whoami`
```

If you're running Docker on MacOS you don't need this actions.

Example 1: hello world
----------------------

Let’s run our first container:

```
docker run ubuntu /bin/echo 'Hello world'
```

Console output:

```
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
d54efb8db41d: Pull complete
f8b845f45a87: Pull complete
e8db7bf7c39f: Pull complete
9654c40e9079: Pull complete
6d9ef359eaaa: Pull complete
Digest: sha256:dd7808d8792c9841d0b460122f1acf0a2dd1f56404f8d1e56298048885e45535
Status: Downloaded newer image for ubuntu:latest
Hello world
```

* **docker run** is command to run a container.
* **ubuntu** is the image you run, for example the Ubuntu operating system image. When you specify an image, Docker looks first for the image on your Docker host. If the image does not exist locally, then the image is pulled from the public image registry Docker Hub.
* **/bin/echo 'Hello world'** is the command to run inside the new container.

This container just prints `Hello world` and stops its execution. Not very big deal, yeah?. Let’s try to create an interactive shell inside Docker container:

```
docker run -i -t --rm ubuntu /bin/bash
```

* **-t** flag assigns a pseudo-tty or terminal inside the new container.
* **-i** flag allows you to make an interactive connection by grabbing the standard input (STDIN) of the container.
* **--rm** flag to automatically remove the container when it exits. By default containers are not deleted.

This container lives until we keep the shell session and dies when we exit from session (like SSH session to remote server). To keep container running we need to daemonize it:

```
docker run --name daemon -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

* **--name daemon** assign name `daemon` to new container. If you don’t specify a name explicitly Docker will generate it automatically and assign it to the container.
* **-d** flag runs the container in the background (daemonize it).

Let’s see what containers do we have now:

```
docker ps -a
```

Console output:

```
CONTAINER ID  IMAGE   COMMAND                 CREATED             STATUS                         PORTS  NAMES
1fc8cee64ec2  ubuntu  "/bin/sh -c 'while..."  32 seconds ago      Up 30 seconds                         daemon
c006f1a02edf  ubuntu  "/bin/echo 'Hello ..."  About a minute ago  Exited (0) About a minute ago         gifted_nobel
```

* **docker ps** is command to command to list containers.
* **-a** show all containers (default shows only running).

Command shows us that we have two containers:
* `gifted_nobel` (name for this container was generated automatically - it will be different on your machine) - it's the first container we've created which printed 'Hello world' once.
* `daemon` - it's the third container we've created which runs as a daemon.

Also there is no second container with interactive shell because we’ve set `--rm` option and it was automatically deleted after execution.

Ok. Let’s see what daemon container is doing right - let’s see the logs of it:

```
docker logs -f daemon
```

Console output:

```
...
hello world
hello world
hello world
```

* **docker logs** is command to fetch the logs of a container.
* **-f** flag to follow the log output (works actually like `tail -f`).

Now let’s stop `daemon` container:

```
docker stop daemon
```

* **docker stop** is command to stop Docker container.

```
docker ps -a
```

Console output:

```
CONTAINER ID  IMAGE   COMMAND                 CREATED        STATUS                      PORTS  NAMES
1fc8cee64ec2  ubuntu  "/bin/sh -c 'while..."  5 minutes ago  Exited (137) 5 seconds ago         daemon
c006f1a02edf  ubuntu  "/bin/echo 'Hello ..."  6 minutes ago  Exited (0) 6 minutes ago           gifted_nobel
```

Now container `daemon` is stopped. We can start it again:

```
docker start daemon
```

And ensure that it’s running:

```
docker ps -a
```

Console output:

```
CONTAINER ID  IMAGE   COMMAND                 CREATED        STATUS                    PORTS  NAMES
1fc8cee64ec2  ubuntu  "/bin/sh -c 'while..."  5 minutes ago  Up 3 seconds                     daemon
c006f1a02edf  ubuntu  "/bin/echo 'Hello ..."  6 minutes ago  Exited (0) 7 minutes ago         gifted_nobel
```

Now let’s stop it again and remove all the containers manually:

```
docker stop daemon
docker rm <your first container name>
docker rm daemon
```

To remove all containers we can use this command:

```
docker rm -f $(docker ps -aq)
```

* **docker rm** is command to remove container.
* **-f** flag (for `rm`) is to stop container if it's running (force deletion).
* **-q** flag (for `ps`) is to print only container IDs.

Let's run `ps` again to ensure that we don't have containers:

```
docker ps -a
```

Console output:

```
CONTAINER ID    IMAGE    COMMAND    CREATED    STATUS    PORTS    NAMES
```

Links: [docker run](https://docs.docker.com/engine/reference/commandline/run/) | [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) | [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) | [docker start](https://docs.docker.com/engine/reference/commandline/start/) | [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) | [docker rm](https://docs.docker.com/engine/reference/commandline/rm/)

Example 2: nginx
----------------

Now let’s try to create and run more meaningful container like `nginx`.

```
cd examples/nginx
docker run -d --name test-nginx -p 80:80 -v $(pwd):/usr/share/nginx/html:ro nginx:latest
```

Console output:

```
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
693502eb7dfb: Pull complete
6decb850d2bc: Pull complete
c3e19f087ed6: Pull complete
Digest: sha256:52a189e49c0c797cfc5cbfe578c68c225d160fb13a42954144b29af3fe4fe335
Status: Downloaded newer image for nginx:latest
436a602273b0ca687c61cc843ab28163c720a1810b09005a36ea06f005b0c971
```

* **-p** is a ports mapping `<HOST PORT>:<CONTAINER PORT>`.
* **-v** is a volume mounting `<HOST DIRECTORY>:<CONTAINER DIRECTORY>`.

**Important**: `run` command accepts only absolute paths. In our example we've used `$(pwd)` to set current directory absolute path.

Now you can check [this url](http://127.0.0.1/) url in your browser.

We can try to change `/example/nginx/index.html` (which is mounted as a volume to directory `/usr/share/nginx/html` inside container) and refresh the page.

Let’s investigate some information about `test-nginx` container:

```
docker inspect test-nginx
```

This command displays system wide information regarding the Docker installation. Information displayed includes the kernel version, number of containers and images, exposed ports, mounted volumes, etc.

There are a lot of information in `inspect` output. If we just need to see the port mappings we can use `port`:

```
docker port test-nginx
```

Console output:

```
80/tcp -> 0.0.0.0:80
```

Links: [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) | [docker port](https://docs.docker.com/engine/reference/commandline/port/) | [Managing data in Docker volumes](https://docs.docker.com/engine/tutorials/dockervolumes/)

Dockerfile
----------

To build a Docker image you need to write Dockerfile. It’s a plain text file with instructions and its arguments. Here is the description of instructions we’ll use in next example:

* **FROM** - set base image
* **RUN** - execute command in container
* **ENV** - set environment variable
* **WORKDIR** - set working directory
* **VOLUME** - create mount-point for volume
* **CMD** - set executable for container

Link: [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

Example 3: writing Dockerfile
-----------------------------

Let’s create an image that will get site contents with `curl` and store it to the text file. We’ll pass site url via environment variable `SITE_URL`. Resulting file will be placed in a directory mounted as volume. Here is our [Dockerfile](examples/curl/Dockerfile):

```
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install --no-install-recommends --no-install-suggests -y curl
ENV SITE_URL https://google.com/
WORKDIR /data
VOLUME /data
CMD sh -c "curl -L $SITE_URL > /data/results"
```

Step into `examples/curl` directory and execute the following command to build an image:

```
docker build . -t test-curl
```

Console output:

```
Sending build context to Docker daemon 3.584 kB
Step 1/7 : FROM ubuntu:latest
 ---> 0ef2e08ed3fa
Step 2/7 : RUN apt-get update
 ---> Running in 4aa839bb46ec
Get:1 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]
Get:2 http://archive.ubuntu.com/ubuntu xenial-updates InRelease [102 kB]
...
Fetched 24.9 MB in 4s (5208 kB/s)
Reading package lists...
 ---> 35ac5017c794
Removing intermediate container 4aa839bb46ec
Step 3/7 : RUN apt-get install --no-install-recommends --no-install-suggests -y curl
 ---> Running in 3ca9384ecf8d
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed...
 ---> f3c6d26b95e6
Removing intermediate container 3ca9384ecf8d
Step 4/7 : ENV SITE_URL https://google.com/
 ---> Running in 21b0022b260f
 ---> 9a733ee39a46
Removing intermediate container 21b0022b260f
Step 5/7 : WORKDIR /data
 ---> c024301ddfb8
Removing intermediate container 3bc973e5584c
Step 6/7 : VOLUME /data
 ---> Running in a9594a8958fe
 ---> 6802707a7114
Removing intermediate container a9594a8958fe
Step 7/7 : CMD sh -c "curl -L $SITE_URL > /data/results"
 ---> Running in 37503bc4e386
 ---> 5ebb2a65d771
Removing intermediate container 37503bc4e386
Successfully built 5ebb2a65d771
```

* **docker build** is a command to build a new image locally.
* **-t** set the name:tag to image.

Now we have our new image and we can see it in list of existing images:

```
docker images
```

Console output:

```
REPOSITORY  TAG     IMAGE ID      CREATED         SIZE
test-curl   latest  5ebb2a65d771  37 minutes ago  180 MB
nginx       latest  6b914bbcb89e  7 days ago      182 MB
ubuntu      latest  0ef2e08ed3fa  8 days ago      130 MB
```

So we can create and run container from image. Let’s try with default parameters:

```
docker run --rm -v $(pwd)/vol:/data/:rw test-curl
```

Console output:

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   263  100   263    0     0   1568      0 --:--:-- --:--:-- --:--:--  1574
100 11607    0 11607    0     0  27104      0 --:--:-- --:--:-- --:--:-- 27104
```

To see results saved to file run `cat ./vol/results`.

Let’s try with `facebook.com`:

```
docker run --rm -e SITE_URL=https://facebook.com/ -v $(pwd)/vol:/data/:rw test-curl
```

Console output:

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100  116k    0  116k    0     0   168k      0 --:--:-- --:--:-- --:--:--  820k
```

Again to see results run `cat ./vol/results`.

Link: [docker build](https://docs.docker.com/engine/reference/commandline/build/) | [docker images](https://docs.docker.com/engine/reference/commandline/images/)

Difference between images & containers
--------------------------------------

**Containers** are the running instances based on images.

**Images** are the sequence of instructions (layers) from Dockerfile and base image context.

| mode | Command | Layer |  |
|-------|---------------------|-------|---|
| **write** | **container goes here** | 2b064135b7f0 |  |
| read only | CMD sh -c "curl -L $SITE_URL > /data/results" | 5ebb2a65d771 | ⬆ |
| read only | VOLUME /data | 6802707a7114 | ⬆ |
| read only | WORKDIR /data | c024301ddfb8 | ⬆ |
| read only | ENV SITE_URL `https://google.com/` | 9a733ee39a46 | ⬆ |
| read only | RUN apt-get install --no-install-recommends --no-install-suggests -y curl | f3c6d26b95e6 | ⬆ |
| read only | RUN apt-get update | 35ac5017c794 | ⬆ |
| **read only** | **base image** | 0ef2e08ed3fa | ⬆ |

Best practices of creating images
---------------------------------

* Include only necessary context - use `.dockerignore` file (it works like `.gitignore` in git).
* Avoid installing unnecessary packages - it will consume extra disk space.
* Use cache. Add context which changes a lot (for example source code of your project) at the end of Dockerfile - it will utilize Docker cache effectively.
* Be careful with volumes. You should remember what data is in volumes. Because volumes are persistent and don’t die with the containers - next container will use data from volume which was created by previous container.
* Use environment variables (in `RUN`, `EXPOSE`, `VOLUME`). It will make your Dockerfile more flexible.

Link: [Dockerfile - best practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

Connection between containers
-----------------------------

**Docker compose** - is the only right way to connect containers with each other.

You can setup connection between containers in a simple declarative manner by writing [YAML config file](https://docs.docker.com/compose/compose-file/compose-file-v2/).

In case of Linux you need to install compose separately - you can do it with `pip`:

```
pip install docker-compose
```

Link: [Docker compose - overview](https://docs.docker.com/compose/overview/)

Example 4: docker-compose Python + Redis
----------------------------------------

Step into directory `examples/compose` and execute the following command:

```
docker-compose --project-name app-test -f docker-compose.yml up
```

Console output:

```
Creating network "apptest_default" with the default driver
Creating volume "apptest_redis_data" with default driver
Pulling redis (redis:3.2-alpine)...
3.2-alpine: Pulling from library/redis
627beaf3eaaf: Pull complete
a503a4771a4a: Pull complete
72c5d910c683: Pull complete
6aadd3a49c30: Pull complete
adf925aa1ad1: Pull complete
0565da0f872e: Pull complete
Digest: sha256:9cd405cd1ec1410eaab064a1383d0d8854d1eef74a54e1e4a92fb4ec7bdc3ee7
Status: Downloaded newer image for redis:3.2-alpine
Building app
Step 1/9 : FROM python:3.5.2-alpine
3.5.2-alpine: Pulling from library/python
b7f33cc0b48e: Pull complete
8eda8bb6fee4: Pull complete
4613e2ad30ef: Pull complete
f344c00ca799: Pull complete
Digest: sha256:8efcb12747ff958de32b32424813708f949c472ae48ca28691078475b3373e7c
Status: Downloaded newer image for python:3.5.2-alpine
 ---> e70a322afafb
Step 2/9 : ENV BIND_PORT 5000
 ---> Running in 8518936700b3
 ---> 0f652cdd2cee
Removing intermediate container 8518936700b3
Step 3/9 : ENV REDIS_HOST localhost
 ---> Running in 027286e90699
 ---> 6da3674f79fa
Removing intermediate container 027286e90699
Step 4/9 : ENV REDIS_PORT 6379
 ---> Running in 0ef17cb512ed
 ---> c4c514aa3008
Removing intermediate container 0ef17cb512ed
Step 5/9 : COPY ./requirements.txt /requirements.txt
 ---> fd523d64faae
Removing intermediate container 8c94c82e0aa8
Step 6/9 : COPY ./app.py /app.py
 ---> be61f59b3cd5
Removing intermediate container 93e38cd0b487
Step 7/9 : RUN pip install -r /requirements.txt
 ---> Running in 49aabce07bbd
Collecting flask==0.12 (from -r /requirements.txt (line 1))
  Downloading Flask-0.12-py2.py3-none-any.whl (82kB)
Collecting redis==2.10.5 (from -r /requirements.txt (line 2))
  Downloading redis-2.10.5-py2.py3-none-any.whl (60kB)
Collecting itsdangerous>=0.21 (from flask==0.12->-r /requirements.txt (line 1))
  Downloading itsdangerous-0.24.tar.gz (46kB)
Collecting Werkzeug>=0.7 (from flask==0.12->-r /requirements.txt (line 1))
  Downloading Werkzeug-0.11.15-py2.py3-none-any.whl (307kB)
Collecting Jinja2>=2.4 (from flask==0.12->-r /requirements.txt (line 1))
  Downloading Jinja2-2.9.5-py2.py3-none-any.whl (340kB)
Collecting click>=2.0 (from flask==0.12->-r /requirements.txt (line 1))
  Downloading click-6.7-py2.py3-none-any.whl (71kB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->flask==0.12->-r /requirements.txt (line 1))
  Downloading MarkupSafe-1.0.tar.gz
Installing collected packages: itsdangerous, Werkzeug, MarkupSafe, Jinja2, click, flask, redis
  Running setup.py install for itsdangerous: started
    Running setup.py install for itsdangerous: finished with status 'done'
  Running setup.py install for MarkupSafe: started
    Running setup.py install for MarkupSafe: finished with status 'done'
Successfully installed Jinja2-2.9.5 MarkupSafe-1.0 Werkzeug-0.11.15 click-6.7 flask-0.12 itsdangerous-0.24 redis-2.10.5
 ---> 18c5d1bc8804
Removing intermediate container 49aabce07bbd
Step 8/9 : EXPOSE $BIND_PORT
 ---> Running in f277fa7dfcd5
 ---> 9f9bec2abf2e
Removing intermediate container f277fa7dfcd5
Step 9/9 : CMD python /app.py
 ---> Running in a2babc256093
 ---> 2dcc3b299859
Removing intermediate container a2babc256093
Successfully built 2dcc3b299859
WARNING: Image for service app was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating apptest_redis_1
Creating apptest_app_1
Attaching to apptest_redis_1, apptest_app_1
redis_1  | 1:C 08 Mar 09:56:55.765 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1  |                 _._
redis_1  |            _.-``__ ''-._
redis_1  |       _.-``    `.  `_.  ''-._           Redis 3.2.8 (00000000/0) 64 bit
redis_1  |   .-`` .-```.  ```\/    _.,_ ''-._
redis_1  |  (    '      ,       .-`  | `,    )     Running in standalone mode
redis_1  |  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
redis_1  |  |    `-._   `._    /     _.-'    |     PID: 1
redis_1  |   `-._    `-._  `-./  _.-'    _.-'
redis_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|
redis_1  |  |    `-._`-._        _.-'_.-'    |           http://redis.io
redis_1  |   `-._    `-._`-.__.-'_.-'    _.-'
redis_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|
redis_1  |  |    `-._`-._        _.-'_.-'    |
redis_1  |   `-._    `-._`-.__.-'_.-'    _.-'
redis_1  |       `-._    `-.__.-'    _.-'
redis_1  |           `-._        _.-'
redis_1  |               `-.__.-'
redis_1  |
redis_1  | 1:M 08 Mar 09:56:55.767 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_1  | 1:M 08 Mar 09:56:55.767 # Server started, Redis version 3.2.8
redis_1  | 1:M 08 Mar 09:56:55.767 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis_1  | 1:M 08 Mar 09:56:55.767 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
redis_1  | 1:M 08 Mar 09:56:55.767 * The server is now ready to accept connections on port 6379
app_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
app_1    |  * Restarting with stat
app_1    |  * Debugger is active!
app_1    |  * Debugger pin code: 299-635-701
```

Current example will increment view counter in Redis. Open [this url](http://127.0.0.1:5000/) in your browser to check it.

Using `docker-compose` is a topic for separate workshop. To get started you can play with some images from Docker Hub or if you want to create your own images - follow best practices listed above. The only thing I can add in terms of using docker-compose: **always give explicit names to your volumes** in docker-compose.yml (if image has volumes). This simple rule will save you from issue in the future when you'll be inspecting your volumes.

```
version: '2'
services:
  ...
  redis:
    image: redis:3.2-alpine
    volumes:
      - redis_data:/data
volumes:
  redis_data:
```

In this case `redis_data` will be the name inside docker-compose.yml file, for the real volume name it will be prepended with project name prefix.

To see volumes run:

```
docker volume ls
```

Console output:

```
DRIVER              VOLUME NAME
local               apptest_redis_data
```

Without explicit volume name there will be UUID. And here is an example from my local machine:

```
DRIVER              VOLUME NAME
local               0ebafda095b98b1d637025fab5ac93320eda3f099caab60281fb7a24f17ba438
local               0ecb5d83864eea53d1330fbe56dc824a5c0a9bc564a471325000476e15a9c568
local               15b900dd9049e747bd7bf7215b3f7c35ca45de94b9f7a524c99b07c3438b3313
local               1a96735966220310b756793cf0c4d802e55eb7dceac46702daadb7aac1cf596c
local               1ba9533c790b3128e5cf155706369b0aabb944f908a75c6ab43045adb632fc54
local               1e2724b6d24cca6efbdec1fa1bfd1d1b17db578431bb0526e0aba50c8bd5937b
local               279903b1bc69a95c2df4db364cfee6208ee16ee06f4b347e48271e54e1225f16
local               3c6e66a263fee534ef98f2244f9f022e5df1b860ccdd622ca8068b18c94e7667
local               3dc5313f8e45e46e20075302d5bdb72437302f6ab471b83b68bca661b10dc065
local               55019ff44a9a5c1a1781a9023f901568f5e6a917fa88ac3773dda49656900517
local               5e9c5bbd17ca720b599d1878254fbf512b57adb59ac282520e2a3b3170ac4d52
local               67dd8b104343a835690d8e3e10f9f524a8ba9e3fbb97bb6ada0d6a9f3706c455
local               6af42a79396a523b67f31eeef944f070b78bf81de37b0b23b5f8c513729ffe34
local               6e112102b138194c4e0b608091de817e5ae1d1e6c418bf0bca3dd6cf94b08b3f
local               7c349ae180720e0aa41daf9e34fa9976d5fa83f6f7fff258de4f7a2bed201416
local               83169e361c0e962c8db6fcc6e0eff7369698cbfcacc6687bcf8264de95f19f0f
local               b473b4b9fc6c446a1e7af1f56f0a5958d3ae7ec303fad9412face79febd76263
local               c741ff815611a927b0dad73d0a80c51d0f0559c76c472a0edc9032bcfbe44512
local               dd34766f1be55871c890b8a808ab760134934ec0f69a0a7bbd45c1c63338090f
local               ec1a5ac0a2106963c2129151b27cb032ea5bb7c4bd6fe94d9dd22d3e72b2a41b
local               f3a664ce353ba24dd43d8f104871594de6024ed847054422bbdd362c5033fc4c
local               f81a397776458e62022610f38a1bfe50dd388628e2badc3d3a2553bb08a5467f
local               f84228acbf9c5c06da7be2197db37f2e3da34b7e8277942b10900f77f78c9e64
local               f9958475a011982b4dc8d8d8209899474ea4ec2c27f68d1a430c94bcc1eb0227
local               ff14e0e20d70aa57e62db0b813db08577703ff1405b2a90ec88f48eb4cdc7c19
local               polls_pg_data
local               polls_public_files
local               polls_redis_data
local               projectdev_pg_data
local               projectdev_redis_data
local               ws2_postgres_data
local               ws2_redis_data
```

Docker way
----------

Docker has some restrictions and requirements for the architecture of your system (applications that you pack into containers). You can ignore this requirements or find some workarounds to break this restrictions, but this actions won't let you fully get the profit from using Docker. My strong advice is to follow this statements:

* 1 application = 1 container
* Run process in foreground inside containers (don't use systemd, upstart or any other similar tools)
* Keep data out of container (use volumes)
* No SSH (if you need to step into container you can use [exec](https://docs.docker.com/engine/reference/commandline/exec/) command)
* No manual configurations (or actions) inside container
