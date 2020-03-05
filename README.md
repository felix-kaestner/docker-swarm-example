
<p align="center">
  <a href="https://github.com/docker/swarm">
    <img alt="Docker Swarm" src="https://github.com/docker/swarm/raw/master/logo.png?raw=true" width="980">
  </a>
</p>

<h1 align="center">Docker Swarm Example</h1>

<h3 align="center">
  A small and simple example to demonstrate the usage of docker swarm for container orchestration 
</h3>

## Features
- #### Build with Docker 🐳
- #### Easy to use 🚀
- #### Fast deploys ⚡
- #### Batteries included 🔋

## Quick Start

### 1 - Start Docker Swarm Mode 

Initialise Docker Swarm with: `$ docker swarm init --advertise-addr <MANAGER_IP>` 
If you are using a Mac or Windows PC with Docker Desktop you can omit the --advertise-addr flag.

```bash
$ docker swarm init --advertise-addr 192.168.65.3
Swarm initialized: current node (sa6c5bgy7tuuvtee2prvt5141) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5cqlf1vko17n16xc8x5v633ltdjr6xtilp26nw7d9avyxhfin6-1xinuuayy7hd8f75sar3zmaz8 192.168.65.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions. 
```

### 2 - Choose a container registry 

Since Docker Swarm will run on multiple Docker Hosts it is mandatory that your container images are available in a container registry. 
You can e.g. push them to [quay.io](https://quay.io/) or [Docker Hub](https://hub.docker.com/).

If you just want to try out docker swarm, then feel free to create a "throwaway container registry" locally.
You can do so by executing:

```bash
$ docker service create --name registry --publish published=5000,target=5000 registry:2

ujf26qwx67juj1ni5dyma2o8l
overall progress: 1 out of 1 tasks
1/1: running
verify: Service converged

$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
ujf26qwx67ju        registry            replicated          1/1                 registry:2          *:5000->5000/tcp
```

### 3 - Test your application.

Before you deploy your swarm, make shure your application is working like expected.
To do so, simply start it with docker-compose.

```bash
$ docker-compose up -d

WARNING: The Docker Engine you\'re using is running in swarm mode.

Compose does not use swarm mode to deploy services to multiple nodes in a swarm. All containers will be scheduled on the current node.

To deploy your application across the swarm, use `docker stack deploy`.

Creating network "docker-swarm-example_default" with the default driver
Creating volume "docker-swarm-example_data" with default driver
Creating docker-swarm-example_web_1        ... done
Creating docker-swarm-example_visualizer_1 ... done
Creating docker-swarm-example_redis_1      ... done
```

Check if all services are running:

```bash
$ docker-compose ps

              Name                             Command               State           Ports
---------------------------------------------------------------------------------------------------
docker-swarm-example_redis_1        docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp
docker-swarm-example_visualizer_1   npm start                        Up      0.0.0.0:8080->8080/tcp
docker-swarm-example_web_1          python app.py                    Up      0.0.0.0:8000->8000/tcp
```

Then stop your application

```bash
$ docker-compose down --volumes

Stopping docker-swarm-example_redis_1      ... done
Stopping docker-swarm-example_visualizer_1 ... done
Stopping docker-swarm-example_web_1        ... done
Removing docker-swarm-example_redis_1      ... done
Removing docker-swarm-example_visualizer_1 ... done
Removing docker-swarm-example_web_1        ... done
Removing network docker-swarm-example_default
Removing volume docker-swarm-example_data
```

And push the images to the registry with


```bash
$ docker-compose push

Pushing web (127.0.0.1:5000/stackdemo:latest)...
The push refers to repository [127.0.0.1:5000/stackdemo]
2b26e33b602b: Pushed
df9e12995dc3: Pushed
d63fdf4f6b3d: Pushed
11210af4e885: Pushed
178572d98447: Pushed
61b675163d2a: Pushed
5216338b40a7: Pushed
latest: digest: sha256:95edea2d6eaaaa66b4eb705c64b76fd3e293cee88b1175928e83353f1ff92eb5 size: 1788
```

### 4 - Deploy your stack 🎉
Use the docker-compose.yml to deploy your application with docker swarm. All instructions that are not supported by docker swarm (like build) will automatically be ignored.

```bash
$ docker stack deploy --compose-file docker-compose.yml stackdemo

Ignoring unsupported options: build

Creating network stackdemo_default
Creating service stackdemo_web
Creating service stackdemo_redis
Creating service stackdemo_visualizer
```

You can view all of of the services running with:

```bash
$ docker stack services stackdemo

ID                  NAME                   MODE                REPLICAS            IMAGE                             PORTS
kebocx1hvtmk        stackdemo_redis        replicated          1/1                 redis:alpine                      *:6379->6379/tcp
vmtjfjv8ymj9        stackdemo_web          replicated          2/2                 127.0.0.1:5000/stackdemo:latest   *:8000->8000/tcp
wikio3xzeoyb        stackdemo_visualizer   replicated          1/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
```

If you wish to scale a service, simply do:

```bash
$ docker service scale <SERVICE>=<REPLICAS>
```

For Example:

```bash
$ docker service scale stackdemo_web=5

stackdemo_web scaled to 5
overall progress: 5 out of 5 tasks
1/5: running   [==================================================>]
2/5: running   [==================================================>]
3/5: running   [==================================================>]
4/5: running   [==================================================>]
5/5: running   [==================================================>]
verify: Service converged
```

If you know do another `$ docker stack services stackdemo`, you should see:

```bash
$ docker stack services stackdemo

ID                  NAME                   MODE                REPLICAS            IMAGE                             PORTS
kebocx1hvtmk        stackdemo_redis        replicated          1/1                 redis:alpine                      *:6379->6379/tcp
vmtjfjv8ymj9        stackdemo_web          replicated          5/5                 127.0.0.1:5000/stackdemo:latest   *:8000->8000/tcp
wikio3xzeoyb        stackdemo_visualizer   replicated          1/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
```

A visualization of the whole cluster is provided under [http://localhost:8080](http://localhost:8080).


### 5 - Join other nodes to the Swarm.
Just use the command provided when you initialized the cluster to join other nodes to the cluster.

```bash
$ docker swarm join --token SWMTKN-1-5cqlf1vko17n16xc8x5v633ltdjr6xtilp26nw7d9avyxhfin6-1xinuuayy7hd8f75sar3zmaz8 192.168.65.3:2377

This node joined s swarm as a worker 
```

To check if everything worked as expected:

```bash
$ docker node ls

ID                            HOSTNAME      STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
sdp8vy1dq1kxleu9g4u78tlag *   worker1       Ready               Active              Reachable           ...
sa6c5bgy7tuuvtee2prvt5141 *   manager1      Ready               Active              Leader              ...
```

If you wish to show the join command again, docker provides a useful built-in command. Simply execute:

```bash
# join command for a new manager node
$ docker swarm join-token manager

# join command for a new worker node
$ docker swarm join-token worker
```

If you want to test multi-node docker swarm on a single machine, you can use [docker machine](https://github.com/docker/machine).
Docker Machine allow you to create multiple virtual docker hosts on your machine. You can create one using:
    - Mac: `$ docker-machine create --driver virtualbox default`
    - Windows: `$ docker-machine create --driver hyperv default`
    - Linux: `$ docker-machine create --driver amazonec2 default` 
    (Linux actually provides a handful of different drivers incl. Amazon Web Services, Microsoft Azure, Digital Ocean, Google Compute Engine and OpenStack )

Then use `$ eval ”$(docker-machine env default)”` to switch you current shell to the created docker host.

### 6 - Use the Application 👏

```bash
$ curl http://localhost:8000
Hello World! I have been seen 1 times.

$ curl http://localhost:8000
Hello World! I have been seen 2 times.

$ curl http://localhost:8000
Hello World! I have been seen 3 times.
```

### 7 - Stop and remove your stack
Remove your stack with:

```bash
$ docker stack rm stackdemo

Removing service stackdemo_redis
Removing service stackdemo_visualizer
Removing service stackdemo_web
Removing network stackdemo_default
```

In case you create a throwaway local registry, you can remove it with:

```bash
$ docker service rm registry

registry
```

### 8 - Leave the swarm 😭
```bash
$ docker swarm leave --force

Node left the swarm.
```
