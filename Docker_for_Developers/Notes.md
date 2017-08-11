# Stage 1

Test env with 
```
docker container run hello-world
```

## 1.0 First container

Get image of alpine from **Docker registry**, by default the registry is Docker **Store**
```
docker image pull alpine
```
You can see what images are available in your system with `docker image ls`

### 1.1 Docker Container Run

Run `ls -l` in the alpine image on a container
```
docker container run alpine ls -l
```

As `ls -l` you can run other commands like `echo`
```
docker container run alpine echo "hello from alpine"
```

If you want to run a shell inside alpine you as well
```
docker container run alpine /bin/sh
```

Altough you need a tty to actually use the shell, this can be done with `docker container run -it alpine /bin/sh`
