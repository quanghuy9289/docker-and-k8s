# Docker

---

## 1. Common commands

`docker run <image_name>` - Create and start a docker container from an image name, if image is not exist then download it from docker hub

`docker ps` - List running containers

`docker ps -a` - List all the docker containers, include exited containers.

`docker create <image_name> <command>` - Docker command to create an container from image name, command is optional - run this command after create container.

`docker start -a <container_id` - Start/Restarting container

`docker system prune` - Removing all stopped containers

`docker logs <container_id>` - Retrieve logs of a container

`docker stop <container_id>` - Stop a container

`docker exec -it <container_id> <command_to_run>` - Execute a command in a container

`docker stop <container_id>` - Stop a container after few seconds (10s)

`docker kill <container_id>` - Stop a container immediately

---

## 2. Building custom images through Docker Server

### Create Docker image

> ### Dockerfile => Docker Client => Docker Server => Usable Image

- Create Dockerfile

  1. Specify a base image (OS) - `FROM <base_image>`
  2. Run some commands to install additional programs - `RUN <commands_to_install_dependencies>`
  3. Specify a command to run on container startup - `CMD <commands_to_run_on_container>`

- Build image from Dockerfile

    > cd `<Dockerfile_folder_path>` && docker build .

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


