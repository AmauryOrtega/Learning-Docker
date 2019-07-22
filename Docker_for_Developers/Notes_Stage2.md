# Docker registry

## Custom registry

A **registry** is a service for storing and accessing Docker images.

Docker expects to connect to a HTTPS registry for this test, we will host our own docker registry and for that we'll need to change a config in our docker engine by adding the following.

### /etc/docker/docker

```shell
DOCKER_OPTS="--insecure-registry 127.0.0.1:5000"
```

And then you must restart the docker daemon `systemctl restart docker`. Then we try to run the registry by using `docker run -d -p 5000:5000 --name registry registry:2`.

Then we can pull hello-world with `docker pull hello-world`, the real name of this image is `docker.io/hello-world:latest` where:

- `docker.io` is the hostname of the registry
- `hello-world` is the repository name
- `latest` is the image tag

Because our registry is running under `127.0.0.1:500`, this will be the hostname.

## Custom registry with SSL certificates

Now well create our own SSL certificate inside a `certs` directory.

```shell
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

And now we have to pass this certificate (`domain.crt`) to the docker daemon.

```shell
mkdir /etc/docker/certs.d
mkdir /etc/docker/certs.d/127.0.0.1:5000 
cp $(pwd)/certs/domain.crt /etc/docker/certs.d/127.0.0.1:5000/ca.crt
```

An then we restart the daemon with `systemctl restart docker`.

Now we can run the registry with the volume `registry-data` and the certificates.

```shell
docker run -d -p 5000:5000 --name registry \
  --restart unless-stopped \
  -v $(pwd)/registry-data:/var/lib/registry -v $(pwd)/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry
```

Where:

- `--restart unless-stopped` restarts the container when it exits so it's always available.
- ` -v $(pwd)/certs:/certs mounts` the volume for certificates.
- `-e REGISTRY_HTTP_TLS_CERTIFICATE` location for the SSL certificate file
- ` -e REGISTRY_HTTP_TLS_KEY` location of the SSL key file

Now we can publish and pull images to this secure registry.

```shell
docker pull hello-world
docker tag hello-world 127.0.0.1:5000/hello-world
docker push 127.0.0.1:5000/hello-world
docker pull 127.0.0.1:5000/hello-world
```

## Custom registry with authentication

We create a `auth` directory to store usernames and encrypted passwords using Apache's httpasswd.

```shell
mkdir auth
docker run --entrypoint htpasswd registry:latest -Bbn moby gordon > auth/htpasswd
```

This will create a user called `moby` with password `gordon`

This process is similar to adding SSL. This time we have to get access to the `htpasswd` in the container using environment variables.

```shell
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

Now if we try to pull `127.0.0.1:5000/hello-world` it won't be possible  because the client is not using any kind of authentication. To do so we must login with `docker login 127.0.0.1:5000`
