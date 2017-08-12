# Stage 1

## 0.0 Hello World

`docker container run hello-world` to test docker installation

## 1.0 First Alpine Linux Container
`docker image pull alpine` Gets image of alpine from **Docker registry**, by default the registry is **Docker Store**

`docker image ls` See what images are available in your system 

### 1.1 Docker Container Run
`docker container run alpine ls -l` Runs `ls -l` in the alpine image on a container

`docker container run alpine echo "hello from alpine"` You can run any command

`docker container run alpine /bin/sh` This won't work by itself

`docker container run -it alpine /bin/sh`

Here, `-i` makes the container interactive and `-t` attaches a tty session to the container
- **Images** The file system and configuration of our application which are used to create containers.
- **Containers** Running instances of Docker images.
- **Docker daemon** The background service running on the host that manages building, running and distributing Docker containers.
- **Docker client** The command line tool that allows the user to interact with the Docker daemon.
- **Docker Store** Store is, among other things, a registry of Docker images.

### Questions
Where do images get pulled from by default? *Docker Store*

Which command lists your Docker images? *docker image ls*

## 2.0 Simple Web App
`docker container run -d seqvence/static-site` Runs a static website in detached mode `-d` Altough is not accesible like this

First get the id of the container to stop it, you get the id using `docker container ls` and then you use `docker container stop [ID]` and `docker container rm [ID]`

`docker container run --name static-site -e AUTHOR="Amaury Ortega" -d -P seqvence/static-site` Runs the app with these flags
- `-d` Detached mode
- `-P` Exposes container's ports to random ports on Docker host
- `-e` Passes environment variables to the container
  - `AUTHOR` environment variable
- `--name` Specify the container name

Then you can get the container's port with `docker container port static-site` so you can access the container with http://[Docker host's IP]:[Port]

If you want to use an specific port, you can use `docker container run --name static-site -e AUTHOR="AmauryOrtega" -d -p 8080:80 seqvence/static-site` This will map container's port 80 to Docker host's port 8080

### 2.1 Docker Images
`docker image ls` List of images on docker host

`docker search` Search images on registry, by default is the **Docker Store**

- **Base images** are images that have no parent images, usually images with an OS like ubuntu, alpine or debian.
- **Child images** are images that build on base images and add additional functionality.

- **Official images** Docker sanctioned images.
- **User images** Created and shared by users. `user/image-name`

### 2.2 Create your first image
Necesary Files
- [app.py](../master/Docker_for_Developers/Stage1_2/app.py)
- [requirements.txt](../master/Docker_for_Developers/Stage1_2/requirements.txt)
- [Dockerfile](../master/Docker_for_Developers/Stage1_2/Dockerfile)
- [templates](../master/Docker_for_Developers/Stage1_2/templates)

`docker image build -t xxdrackleroxx/simple-app .` To build image

`docker container run -p 8888:5000 --name app xxdrackleroxx/simple-app` To run

`docker container rm app` To remove exited container or 

```
docker container stop app
docker container rm app
```
or
`docker container rm -f app`

### 2.3 Dockerfile commands
- `FROM` starts the Dockerfile. Defines your base layer.
- `RUN` is used to build up the Image you’re creating. For each `RUN` command, Docker will run the command then create a new layer of the image.
- `COPY` copies local files into the container.
- `CMD` defines the commands that will run on the Image at start-up. Unlike a `RUN`, this does not create a new layer for the Image, but simply runs the command. There can only be one `CMD` per a Dockerfile/Image. If you need to run multiple commands, the best way to do that is to have the `CMD` run a script. `CMD` requires that you tell it where to run the command, unlike `RUN`. So example `CMD` commands would be:
```
CMD ["python", "./app.py"]
CMD ["/bin/bash", "echo", "Hello World"]
```
- `EXPOSE` opens ports in your image to allow communication to the outside world when it runs in a container.

### Questions
What command starts a container from the command line? *docker container run*

How do you get a list of running containers? *docker container ls*

Which of the following are Dockerfile commands? *RUN, CMD, COPY, FROM*

What is the first command in every Dockerfile? *FROM*

## 3.0 Swarm stack introduction
Using Node1 and Node2

#### Init your swarm
`docker swarm init --advertise-addr $(hostname -i)` On the master node (Node 1). 
`hostname -i` gives the IP Address

This will be the output on Node1
```
Swarm initialized: current node (b3nvddvuwjr7r9o1wb527m4zv) is now a manager.

To add a worker to this swarm, run the followingcommand:

    docker swarm join --token SWMTKN-1-3l60aqx95kyyzu1335c3wno4pw4dlpqqw18cn081ejd4ywmrqg-43yxqps0a1tpgj83m6xbmoao8 10.0.57.3:2377
```
Then you must the join command on Node2

#### Show members of swarm
On Node 1 `docker node ls`

#### Clone the voting-app
On Node 1
```
git clone https://github.com/docker/example-voting-app
cd example-voting-app
```

#### Deploy stack
A stack is a group of services that are deployed together. The docker-stack.yml in the current folder will be used to deploy the voting app as a stack.

On Node 1 `docker stack deploy --compose-file=docker-stack.yml voting_stack`

`docker stack ls` Shows stacks deployed

`docker stack services voting_stack` Shows the service in the stack

`docker service ps voting_stack_vote` Shows the tasks of the vote service

#### Questions
What is a stack? *a multi-service app running on a Swarm*

A stack can: *be deployed from the commandline, can use the compose file format to deploy, can be used to manage services over multiple nodes*