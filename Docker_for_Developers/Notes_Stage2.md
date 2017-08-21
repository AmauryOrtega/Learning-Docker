# Stage 2

## 1.0 Part 1 Docker registry for Linux

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

## 2.0 Part 2 Running a Secured Registry Container in Linux

Now well create our own SSL certificate inside a `certs` directory.

```
mkdir -p certs 
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
------
Country Name (2 letter code) [AU]:CO
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:xXDrackleroXx
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:127.0.0.1
Email Address []:
```

And now we have to pass this certificate (domain.crt) to the docker daemon.

```
mkdir /etc/docker/certs.d
mkdir /etc/docker/certs.d/127.0.0.1:5000 
cp $(pwd)/certs/domain.crt /etc/docker/certs.d/127.0.0.1:5000/ca.crt
```

An then we restart the daemon with `systemctl restart docker`.

Now we can run the registry with the volume `registry-data` and the certificates.

```
docker run -d -p 5000:5000 --name registry \
  --restart unless-stopped \
  -v $(pwd)/registry-data:/var/lib/registry -v $(pwd)/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry
```

Where:
- **--restart unless-stopped** restarts the container when it exits so it's always available.
- **-v $(pwd)/certs:/certs** mounts the volume for certificates.
- **-e REGISTRY_HTTP_TLS_CERTIFICATE** location for the SSL certificate file
- **-e REGISTRY_HTTP_TLS_KEY** location of the SSL key file

## 2.1 Accessing the Secure Registry

Now we can publish and pull images to this secure registry.

```
docker pull hello-world
docker tag hello-world 127.0.0.1:5000/hello-world
docker push 127.0.0.1:5000/hello-world
docker pull 127.0.0.1:5000/hello-world
```

# 3.0 Part 3 Using Basic Authentication with a Secured Registry in Linux

We create a `auth` directory to store usernames and encrypted passwords using Apache's httpasswd.

```
mkdir auth
docker run --entrypoint htpasswd registry:latest -Bbn moby gordon > auth/htpasswd
```

This will create a user called *moby* with password gordon

# 3.1 Running an Authenticated Secure Registry

This process is similar to adding SSL. This time we have to get access to the `htpasswd` in the container using environment variables.

```
docker run -d -p 5000:5000 --name registry \
  --restart unless-stopped \
  -v $(pwd)/registry-data:/var/lib/registry \
  -v $(pwd)/certs:/certs \
  -v $(pwd)/auth:/auth \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -e REGISTRY_AUTH=htpasswd \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry
```

Where:
- **-v $(pwd)/auth:/auth** mounts the folder `auth` into the containers.
- **-e REGISTRY_AUTH=htpasswd** Use `htpasswd` for authentication method.
- **-e REGISTRY_AUTH_HTPASSWD_REALM='Registry Realm'** specify the authentication realm.
- **-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd** location of the `htpasswd` file.

## 3.3 Authenticating with the Registry

Now if we try to pull `127.0.0.1:5000/hello-world` it won't be possible  because the client is not using any kind of authentication. To do so we must login with `docker login 127.0.0.1:5000`

# Conclusion

Docker Registry is a free, open-source application for storing and accessing Docker images.