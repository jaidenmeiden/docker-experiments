## Docker

[Documentation](https://docs.docker.com/)

### What is a container?

Now that you’ve run a container, what is a container? Simply put, a container is a sandboxed process on your machine that is isolated from all other processes on the host machine. That isolation leverages kernel namespaces and cgroups, features that have been in Linux for a long time. Docker has worked to make these capabilities approachable and easy to use. To summarize, a container:

1. is a runnable instance of an image. You can create, start, stop, move, or delete a container using the DockerAPI or CLI.
2. can be run on local machines, virtual machines or deployed to the cloud.
3. is portable (can be run on any OS)
4. Containers are isolated from each other and run their own software, binaries, and configurations.

* [Docker run](https://docs.docker.com/engine/reference/commandline/run/)
* [Docker ps](https://docs.docker.com/engine/reference/commandline/ps/)
* [Docker exec](https://docs.docker.com/engine/reference/commandline/exec/)
* [Docker stop](https://docs.docker.com/engine/reference/commandline/stop/)
* [Docker rm](https://docs.docker.com/engine/reference/commandline/rm/)
* [Docker logs](https://docs.docker.com/engine/reference/commandline/logs/)

```bash
# Show the Docker version information
$ docker --version

# Display system-wide information
$ docker info
```

```bash
# Run a command in a new container
$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
$ docker run ['Image']

# The -it instructs Docker to allocate a pseudo-TTY connected to the container’s stdin; 
# creating an interactive bash shell in the container.
$ docker run -it ['Image']
# (--detach , -d) Run container in background and print container ID
$ docker run -it -d ['Image'] tail -f /dev/null

# (--publish , -p) 
$ docker run --name ['Any name'] -p ['Port host machine']:['Container port']

```

```bash
# List containers
$ docker ps [OPTIONS]
$ docker ps -a

# Fetch the logs of a container
$ docker logs [OPTIONS] CONTAINER
$ docker logs ['Container ID/Name']
# (--follow , -f) Follow log output
$ docker logs --tail 10 -f ['Container ID/Name']

# Get Pid from main process from container
$ docker inspect --format '{{.State.Pid}}' ['Container ID/Name']
$ kill ['Pid'] # You could kill the main process from container with Pid

# Inside the container
> ps
> ps -aux
```

```bash
# Run a command in a running container
$ docker exec -it ['Container ID/Name'] bash

# Return low-level information on Docker objects
$ docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

```bash
# Stop one or more running containers
$ docker stop ['Container ID/Name']

# Remove one or more containers
$ docker rm ['Container ID/Name']
# Stop and remove one or more containers
$ docker rm -f ['Container ID/Name']

# Remove all stopped containers
$ docker container prune [OPTIONS]
```

## Bind mounts

[Use bind mounts](https://docs.docker.com/get-started/06_bind_mounts/)

```bash
# Create directory to save data
$ mkdir mongo_data_bind_mount

$ docker run -d --name db -v /home/oem/Development/Learning/Docker/docker-experiments/mongo_data:/data/db mongo
$ docker exec -it db bash 
> mongo
~ show dbs
~ use data_base
~ db.users.insert({"nombre": "Jaiden"})
WriteResult({ "nInserted" : 1 })
~ db.users.find()
{ "_id" : ObjectId("61c81fe0bb44bd606e57136f"), "nombre" : "Jaiden" }
~ exit
> exit
```

## Volumes

[Use volumes](https://docs.docker.com/storage/volumes/)

```bash
$ docker volume ls
$ docker volume create data_volume
$ docker run -d --name db --mount src=data_volume,dst=/data/db mongo
$ docker inspec db 
$ docker exec -it db bash 
> mongo
~ show dbs
~ use data_base
~ db.users.insert({"nombre": "Jaiden"})
WriteResult({ "nInserted" : 1 })
~ db.users.find()
{ "_id" : ObjectId("61c81fe0bb44bd606e57136f"), "nombre" : "Jaiden" }
~ exit
> exit
$ docker volume rm ['Volume name']
```

## Insert and extract files from a container (Docker cp)

[Use docker cp](https://docs.docker.com/engine/reference/commandline/cp/)

```bash
# Insert files
$ touch test.txt
$ docker run -d --name copytest ubuntu tail -f /dev/null
$ docker exec -it copytest bash # Enter into container
> mkdir testing 
> exit
$ docker cp tes.txt copytest:/testing/test_copied.txt
$ docker exec -it copytest bash # Enter into container
> cd testing/
> ls -la
< -rw-r--r-- 1 1000 1000    0 Feb  2 15:58 test_copied.txt
> exit
# Extract files
$ docker cp copytest:/testing directory_extracted # Directories
$ docker cp copytest:/testing/test_copied.txt file_extracted.txt # Files
```

![Transfer information](images/transfert.png)

## Images

* [Images](https://docs.docker.com/engine/reference/commandline/images/)
* [Dockerfile](https://docs.docker.com/engine/reference/builder/)
 
```bash
$ docker image ls
$ docker pull ubuntu:20.04
$ mkdir ubuntu
$ touch ubuntu/Dockerfile
```

```dockerfile
FROM ubuntu:latest

RUN touch /usr/src/test/dockerfile.txt
```

```bash
$ cd ubuntu
$ docker build -t ubuntu:test_ubuntu . # Use by default Dockerfile into directory
# Create container with previus image
$ docker run -it ubuntu:test_ubuntu
> ll
> cd /usr/src/test
> ls -la
> exit
$ docker images ls
# Generate a  new tag fro the same image
$ docker tag ubuntu:test_ubuntu jaidenmeiden/ubuntu:test_ubuntu
$ docker login
$ docker push jaidenmeiden/ubuntu:test_ubuntu

```

## Layers system

[wagoodman/dive](https://github.com/wagoodman/dive)

```bash
$ docker history <image_name>
$ dive <image_tag>

```

## Command line reference (Docker CLI)

* [Docker run](https://docs.docker.com/engine/reference/run/)
* [Docker build](https://docs.docker.com/engine/reference/commandline/build/)
* [Docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
* [Docker push](https://docs.docker.com/engine/reference/commandline/push/)
* [Docker commit](https://docs.docker.com/engine/reference/commandline/commit/)

## Using docker to develop applications

[Docker node example](https://github.com/platzi/docker.git)

Original Dockerfile to manage application based on `node`

```dockerfile
FROM node:12

COPY [".", "/usr/src/"]

WORKDIR /usr/src

RUN npm install

EXPOSE 3000

CMD ["node", "index.js"]

```

```bash
$ cd node_test
$ git clone https://github.com/platzi/docker.git
# Review inner files
$ ls -la
> ...
> docker-compose.yml
> Dockerfile
> ...

# Build images
$ docker build -t node_test .

# Review images
$ docker image ls
> ...
REPOSITORY            TAG           IMAGE ID       CREATED              SIZE
node_test             latest        7e40a5bf7cd3   About a minute ago   931MB
> ...

# Run container
$ docker run --rm -p 3000:3000 node_test
...
Server listening on port 3000!
...
```
### Run container with bind mount

Modify original Dockerfile to manage application based on `node`

```dockerfile
FROM node:12

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src

RUN npm install

COPY [".", "/usr/src/"]

EXPOSE 3000

CMD ["npx", "nodemon", "index.js"]

```

```bash
# Run container with bind mount
$ docker run --rm -p 3000:3000 -v ~/Docker/docker-experiments/node_files/index.js:/usr/src/index.js node_test
...
[nodemon] 1.18.6
[nodemon] to restart at any time, enter `rs`
[nodemon] watching: *.*
[nodemon] starting `node index.js`
Server listening on port 3000!
...
```

Review browser: http://localhost:3000/

### Docker networking: collaboration between containers

Adjust original Dockerfile to manage application based on `node` without `nodemon`

```dockerfile
FROM node:12

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src

RUN npm install

COPY [".", "/usr/src/"]

EXPOSE 3000

CMD ["node", "index.js"]
```

[Docker network](https://docs.docker.com/engine/reference/commandline/network/)

```bash
$ docker network ls 
...
NETWORK ID     NAME      DRIVER    SCOPE
e54e43840649   bridge    bridge    local
b62a386578b2   host      host      local
cb2699850d72   none      null      local
...

# Create our network
$ docker network create --attachable my_network
$ docker network ls 

...
NETWORK ID     NAME         DRIVER    SCOPE
e54e43840649   bridge       bridge    local
b62a386578b2   host         host      local
b7e0625aef1a   my_network   bridge    local
cb2699850d72   none         null      local
...

$ docker network inspect my_network
...
[
    {
        "Name": "my_network",
        "Id": "b7e0625aef1a2aebd752de880b643072134b60a98f7dfca3b7ef5fe1c031e43e",
        "Created": "2022-02-04T19:05:00.461154705+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
...

$ docker run -d --name db mongo # Create new container
$ docker network connect my_network db
$ docker network inspect my_network
...
[
    {
        "Name": "my_network",
        "Id": "b7e0625aef1a2aebd752de880b643072134b60a98f7dfca3b7ef5fe1c031e43e",
        "Created": "2022-02-04T19:05:00.461154705+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "e05c7914226a9a6c5052bd5a0f2717ed74d51d9e5d335f2472b3b921a949bd1b": {
                "Name": "db",
                "EndpointID": "734941d8caa4149ee3ec3a328fa095ba72255e7e0349e181f150162211784214",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
...
```
Code section from index.js (Node application) where the application catch a `env` variable

```javascript
// Connection URL
const mongoUrl = process.env.MONGO_URL || 'mongodb://localhost:27017/test';
```

Init container with behind variations

```bash
$ docker network connect my_network app
$ docker network inspect my_network
...
[
    {
        "Name": "my_network",
        "Id": "b7e0625aef1a2aebd752de880b643072134b60a98f7dfca3b7ef5fe1c031e43e",
        "Created": "2022-02-04T19:05:00.461154705+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "40d491ab88c98d1df00d452cdd0954bd3c5591b6ef3eb6fdbc1f03a96a1786ce": {
                "Name": "app",
                "EndpointID": "985b75faec75cf4c0eaac4aa0f5d69cd36825931a964acef49ca89652c137860",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "e05c7914226a9a6c5052bd5a0f2717ed74d51d9e5d335f2472b3b921a949bd1b": {
                "Name": "db",
                "EndpointID": "734941d8caa4149ee3ec3a328fa095ba72255e7e0349e181f150162211784214",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
...

$ docker run -d --name app -p 3000:3000 --env MONGO_URL=mongodb://db:27017/test node_test 
$ docker ps
...
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                                       NAMES
40d491ab88c9   node_test   "docker-entrypoint.s…"   2 minutes ago    Up 2 minutes    0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   app
e05c7914226a   mongo       "docker-entrypoint.s…"   26 minutes ago   Up 26 minutes   27017/tcp                                   db
...
```

### Docker compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see the list of features:

1. Multiple isolated environments on a single host
2. Preserve volume data when containers are created
3. Only recreate containers that have changed
4. Variables and moving a composition between environments

* [Features](https://docs.docker.com/compose/#features)
* [Docker compose](https://docs.docker.com/engine/reference/commandline/compose/)

```yaml
version: "3.8"

services:
  app:
    image: node_test
    environment:
      MONGO_URL: "mongodb://db:27017/test"
    depends_on:
      - db
    ports:
      - "3000:3000"

  db:
    image: mongo
```

```bash
$ docker rm -f app db
$ docker ps
...
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
...

$ docker-compose up

```

Possible errors:

```bash
```
Solution:

```bash
# Verify docker-compose version
$ docker-compose --version
...
docker-compose version 1.25.0, build unknown
...

# Uninstall Docker compose 1
$ where docker-compose 
$ sudo rm /usr/bin/docker-compose
$ sudo apt autoremove

# Install docker version 2
$ mkdir -p ~/.docker/cli-plugins/
$ curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
$ chmod +x ~/.docker/cli-plugins/docker-compose
$ docker compose version
...
Docker Compose version v2.2.3
...

# Uninstall Docker Compose 2
$ sudo rm ~/.docker/cli-plugins/docker-compose

```