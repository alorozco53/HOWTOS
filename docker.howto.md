# Docker HOWTO

- Tutorial: https://docs.docker.com/get-started/

## Ubuntu Install

- https://docs.docker.com/desktop/install/ubuntu/
- After you’ve successfully installed Docker Desktop, you can check the versions 
  of these binaries by running the following commands:
  ```bash
   docker compose version
   docker --version
   docker version
  ```
- To enable Docker Desktop to start on login
  ```bash
  systemctl --user enable docker-desktop
  ```
- To stop Docker Desktop
  ```bash
  systemctl --user stop docker-desktop
  ```
- [Post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/) 


## Upgrade Docker Desktop

```bash
sudo apt-get install ./docker-desktop-<version>-<arch>.deb
```

## Credentials management for Linux users

https://docs.docker.com/desktop/get-started/#credentials-management-for-linux-users

Docker Desktop relies on `pass to store credentials in gpg2-encrypted files.

You can intialize pass by using a gpg key. To generate a gpg key, run:

```bash
gpg --generate-key
```

To initialize pass, run:

```bash
pass init <pubkey>
```

### Docker ROOTLESS Security Issues

https://docs.docker.com/engine/security/#docker-daemon-attack-surface

## Docker Contexts

https://docs.docker.com/engine/context/working-with-contexts/

A context is a combination of several properties. These include:

- Name
- Endpoint configuration
- TLS info
- Orchestrator

```bash
docker context ls
docker context inspect default
```

- To create a new context:
  ```bash
  docker context create docker-test \
  --default-stack-orchestrator=swarm \
  --docker host=unix:///var/run/docker.sock
  ```
  
## Docker Compose

https://docs.docker.com/compose/

- [Compose File Specification](https://docs.docker.com/compose/compose-file/)

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a
YAML file to configure your application’s services. Then, with a single command, you create and start all
the services from your configuration. 

Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run docker compose up and the Docker compose command starts and runs your entire app. 
   You can alternatively run `docker-compose up` using Compose standalone (docker-compose binary).
   
```bash
docker compose up
docker compose stop
docker compose down --volumes
```

- `-d`: run containers in the background

- Modifying on the fly (listen for changes): make sure to add a volume(s)
  to `docker-compose.yml`

   
A docker-compose.yml looks like this:

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    depends_on:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

| Compose Application Model | Description |
| ----------------- | ------------- |
| [Services](https://docs.docker.com/compose/compose-file/#services-top-level-element) | abstract definition of a computing resource, defined by a Docker image and set of runtime arguments |
| [Networks](https://docs.docker.com/compose/compose-file/#networks-top-level-element) | platform capability abstraction to establish an IP route between containers |
| [Volumes](https://docs.docker.com/compose/compose-file/#volumes-top-level-element) | services store and share persistent data into volumes |
| [Configs](https://docs.docker.com/compose/compose-file/#configs-top-level-element) | configuration data that is dependent on the runtime or platform |


A **Project** is an individual deployment of an application specification on a platform. A project’s 
name is used to group resources together and isolate them from other applications or other installation of
the same Compose specified application with distinct parameters. A Compose implementation creating resources
on a platform MUST prefix resource names by project and set the label `com.docker.compose.project`.

Another example, of a 2-service (front vs backend) compose file:

```yaml
services:
  frontend:
    image: awesome/webapp
    ports:
      - "443:8043"
    networks:
      - front-tier
      - back-tier
    configs:
      - httpd-config
    secrets:
      - server-certificate

  backend:
    image: awesome/database
    volumes:
      - db-data:/etc/data
    networks:
      - back-tier

volumes:
  db-data:
    driver: flocker
    driver_opts:
      size: "10GiB"

configs:
  httpd-config:
    external: true

secrets:
  server-certificate:
    external: true

networks:
  # The presence of these objects is sufficient to define them
  front-tier: {}
  back-tier: {}
```

- [Environment variable usage](https://docs.docker.com/compose/environment-variables/)

## GPU Support

https://docs.docker.com/config/containers/resource_constraints/#gpu


- [**Compose GPU Support**](https://docs.docker.com/compose/gpu-support/)
- A typical error: https://askubuntu.com/questions/1400476/docker-error-response-from-daemon-could-not-select-device-driver-with-capab

- Add `--gpus all` to `docker run` command

## Running Stuff

- Building
  ```bash
  docker build -t <name:tag> .
  ```
- Start container
  ```bash
  docker run [-dp] <HOST_PORT>:<CONTAINER_PORT> <name>
  ```
  - `-d`: detached mode (background)
  - `-p`: mapping between host's port to container's port
- Stop all containers
  ```bash
  docker kill $(docker ps -q)
  ```
- Remove all containers
  ```bash
  docker rm $(docker ps -a -q)
  ```
- Remove all docker images
  ```bash
  docker rmi -f $(docker images -q)
  ```
- To stop a container: list all containers (`docker container ls`), stop `CONTAINER_ID` and rmove it (`docker rm`)

## Versioning

- Tag image
  ```bash
  docker tag <name>[:<tag>] <YOUR-USER-NAME>/<name>
  ```
- Push to repository
  ```bash
  docker push <YOUR-USER-NAME>/<name>
  ```

## Binding Mounts

https://docs.docker.com/storage/bind-mounts/#choose-the--v-or---mount-flag

When you use a bind mount, a file or directory on the host machine is mounted into a container.
The file or directory is referenced by its absolute path on the host machine.
By contrast, when you use a volume, a new directory is created within Docker’s storage directory
on the host machine, and Docker manages that directory’s contents.
