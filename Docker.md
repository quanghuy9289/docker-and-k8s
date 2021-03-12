# Docker

---

## 1. Common commands

> `docker run <image_name>` - Create and start a docker container from an image name, if image is not exist then download it from docker hub

> `docker ps` - List running containers

> `docker ps -a` - List all the docker containers, include exited containers.

> `docker create <image_name> <command>` - Docker command to create an container from image name, command is optional - run this command after create container.

> `docker start -a <container_id>` - Start/Restarting container

> `docker system prune` - Removing all stopped containers

> `docker logs <container_id>` - Retrieve logs of a container

> `docker exec -it <container_id> <command_to_run>` - Execute a command in a container

> `docker stop <container_id>` - Stop a container after few seconds (10s)

> `docker kill <container_id>` - Stop a container immediately

> `docker build .` - Build image from Dockerfile

> `docker build -f <file_name> .` - Build image from `file_name` (ex: `Dockerfile.dev`, `Dockerfile.staging`...)

---

## 2. Building custom images through Docker Server

### a. Create Docker image

> ### Dockerfile => Docker Client => Docker Server => Usable Image

- Create Dockerfile

  1. Specify a base image (OS) - `FROM <base_image>`
  2. Run some commands to install additional programs - `RUN <commands_to_install_dependencies>`
  3. Specify a command to run on container startup - `CMD <commands_to_run_on_container>`

- Build image from Dockerfile

    > `docker build .`

    Dockerfile is built step by step. If you rebuid it then it will use image cache if steps not be changed. Otherwise, Dockerfile step will be rebuilt as the first time from changed step.

- Tagging an image
    > docker build -t `dockerID/imageName:version` .

    `dockerId` - your Docker hub Id

    `version` - semantic version of your image

    The `.` is build context folder => it means current working directory

- Copy file from current machine to container machine

  The files/folders in current machine can not be known by container, so we need to copy the nesseccessary files/folders from local machine to container (ex: when build NodeJS web server, we run `npm install` need `package.json` file, so copy `package.json` file from your source code in local machine to container to build)

  Use following command in Dockerfile:
  
  > `COPY ./ ./` - Copy all files/folders at current working directory in local machine to  current working directory in container

  The first `./`- current working directory relative to `build context folder`

  The second `./` - current working directory in container

- Container port mapping

  We cannot access to service that open a port in container from local machine. Ex: NodeJS server in container is listening on `8080` port, and we want to use that server for our work. If we open an url (ex: localhost:8080) to connect to Node server on container => cannot access because service port `8080 on local machine` not connect to node server with port `8080 on container`.

  To solve this problem, we use a technique call `port mapping`

  > `docker run -p <local_service_port>:<container_service_port> <image_name>` - Run image that mapping port from local service to container service

- Set working directory for container

  Use command:

  > `WORKDIR <folder_path>`

  After this command, every commands will run with context folder is working directory above

### b. Docker Volumes

Problem (In development mode):

- You copy source code from local machine to container to build and run project.
- If source code is changed after that, you need to rebuild image and re-run container to effect with new code
  => Take time - Not effective

Solve:

- In container machine, you `reference (share)` source code to local machine by mapping `volume` (source code) from current working directory to working directory in container
  > `-v $(pwd):/app` -  Map the pwd (present working directory) into the /app folder
- For source code that doesn't be copied to container machine (ex: `node_modules` folder to reduce build time), we need to put a bookmark to that folder on container
  > `-v /app/node_modules` - Put a bookmark on the node_modules folder (use `node_modules` folder in container, don't override by other
  )

Finally run docker image with below command:

> `docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image_name/id>`

If we change code after that, the changed code will effect immediately on container machine

## 3. Docker compose

Docker CLI is just enable to run docker image one by one => difficult to run multi containers
So docker-compose helps to run multiple containers in one command

### a. Create docker-compose.yml file

Ex: create docker-compose for 2 containers: redis and node

```docker
version: "3"
services:
  redis-server: # service name, can use as host for container
    image: "redis"
  node-app:
    restart: "always" # values maybe: "no", "always", "on-failure", "unless-stopped"
    build: . # build image using Dockerfile
    ports: # port mapping config, might multiple configs
      - "8080:8080"
```

Ex: docker-compose for create-react-app with dev mode

```docker
version: "3"
services:
  web:
    build:
      context: . # building context directory
      dockerfile: Dockerfile.dev # alternative file for Dockerfile
    ports:
      - "3000:3000" #port mapping for service from local machine : to container machine
    volumes:
      - /app/node_modules # put a bookmark on container node_modules folder
      - .:/app # map current directory into /app folder in container
```

### b. Networking between containers

Container A can connect to container B throught `service_name` as ip/host of container

Ex: NodeJS container connect with redis-server container like this:

```javascript
const redisClient = redis.createClient({
  host: "redis-server", // service name of redis specified in docker-compose.yml
  port: 6379, // port of redis server running in container
});
```

### c. docker-compose commands

> `docker-compose up` - Run docker-compose.yml file, start container and mapping port as config,...

> `docker-compose up -d` - Run docker-compose.yml file in background

> `docker-compose up --build` -  Run docker-compose.yml file, re-build container before starting containers

> `docker-compose down` - Stop and remove running container that start by `docker-compose up`

> `docker-compose ps` - List all containers start by docker-compose and their status

Note that docker-compose.yml need exist in current directory
