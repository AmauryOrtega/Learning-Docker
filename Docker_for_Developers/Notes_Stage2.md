# Stage 2

## 1.0 Docker registry for Linux Part 1

A **registry** is a service for storing and accessing Docker images.

Docker expects to connect to a HTTPS registry for this test, we will host our own docker registry and for that we'll need to change a config in our docker engine.

```
vim /etc/docker/docker

# add this line
DOCKER_OPTS="--insecure-registry 127.0.0.1:5000"
```

And then you must restart the docker daemon `systemctl restart docker`. Then we try to run the registry by using `docker run -d -p 5000:5000 --name registry registry:2`.

Then we can pull hello-world with `docker pull hello-world`, the real name of this image is `docker.io/hello-world:latest` where:
- `docker.io` is the hostname of the registry
- `hello-world` is the repository name
- `latest` is the image tag

Because our registry is running under `127.0.0.1:500`, this will be the hostname.

### 1.1 Pushing and Pulling from the Local Registry

As said before, the hostname is the key to select from which registry the docker daemon gets the image. If we already have a image from the docker hub, we can change the image tag with `docker tag` to add our hostname (aka registry) on it.

```
docker tag hello-world 127.0.0.1:5000/hello-world
docker push 127.0.0.1:5000/hello-world
```

For testing purposes, we will delete all of the images stored on the docker daemon and then we will pull the image from our running registry.

```
docker rmi 127.0.0.1:5000/hello-world
docker rmi hello-world
docker pull 127.0.0.1:5000/hello-world
```

So far it works like a charm but the registry is using a temporal file system, to store that data outside the container, we'll use a directory on the host's file system.

### 1.2 Running a Registry Container with External Storage

Let's delete all of our images in the host and any running containers.

```
docker kill registry
docker rm registry
```

Now we'll create a directory called `registry-data` which will be linked to `/var/lib/registry` when the container runs.

```
mkdir registry-data
docker run -d -p 5000:5000 --name registry -v $(pwd)/registry-data:/var/lib/registry registry
```

Now we'll get, tag and push the `hello-world` image to our running registry.

```
docker pull hello-world
docker tag hello-world 127.0.0.1:5000/hello-world
docker push 127.0.0.1:5000/hello-world
```

Now we can destroy and create a new container with the same mapping `$(pwd)/registry-data` to `/var/lib/registry` without loosing data. Next, I'll use SSL to make the registry secure.