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

## 2.1 Docker Images
