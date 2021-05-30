# Docker commands


## Run
```
$ docker run -it --rm -v $(pwd):/app:rw [image-name] [COMMAND] [ARG...]

-i : Keep STDIN open even if not attached
-t : Allocate a pseudo-tty
-v : Bind mount a volume.
    --volume=[host-src:]container-dest[:<options>]
--rm : automatically clean up the container and remove the file system when the container exits
```
* ref: [Docker run reference](https://docs.docker.com/engine/reference/run/)

## Start
start the stopped container
```
$ docker start [container-name]
```

## Save and Load image
```bash
# save as .tar
$ docker save [image-name] > [image-name.tar]

# load
$ docker load < [image-name.tar]
```

## Image (List / Del)
```bash
# list
$ docker images 

or

$ docker image ls
```
```bash
# delete
$ docker image rm [image-name]
```

## Container
### List
```bash
# list all running containers
$ docker container ls
$ docker ps

# list all containers including exited
$ docker container ls -a
$ docker ps -a

# list all running containers with Container ID
$ docker ps -q
```
### Stop
```bash
# stop a running container
$ docker stop [container-name]
$ docker stop [container-id]

# stop all running container
$ docker stop $(docker ps -q)
```
### Remove
```bash
# remove stopped container
$ docker container rm [container-name]

# remove all stopped containers
$ docker container rm $(docker ps -a -q)
```

## Commit
commit to image  
"src image" &rightarrow; "changed container" &rightarrow; "new image"
```bash
$ docker commit [container-id] [image-name]
```

* ref: [Difference between save and export in Docker](https://tuhrig.de/difference-between-save-and-export-in-docker/)

