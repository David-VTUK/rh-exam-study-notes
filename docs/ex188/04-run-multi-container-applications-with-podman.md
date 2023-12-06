# Run multi-container applications with Podman

## Create application stacks

Broadly speaking, application stacks involve the use of multiple containers that work together to form a complete application system. This can be accomplished in two different ways - Podman Pods and Podman-compose, as detailed below.

### Using Podman Pods

A podman `pod` is a group of one or more containers that share the same network namespace, amongst other resources.

To create a Podman pod:

```bash
podman pod create --name mypod -p 8080:80
```

Note: we don't have anything running this pod, it's a shell.  We'll add a `nginx` container inside this pod:

```bash
podman run -d --pod mypod nginx
```

Which we can then access by `curl localhost:8080`

To add a second container to this pod:

`podman run -d --pod mypod redis`

Which we can inspect with

```bash
podman pod ps

POD ID        NAME        STATUS      CREATED        INFRA ID      # OF CONTAINERS
6babca889ba2  mypod       Running     5 minutes ago  a45cc929c682  3
```

Note: it states 3 containers because of the `pause` container.  

### Using Podman-compose

`podman-compose` is a tool designed to help manage multi container applications with Podman. It works in a similar way to Docker Compose, but is tailored for Podman.

`podman-compose` is not a part of the main `podman` binary, therefore it is installed separately.

`podman-compose` uses the same `docker-compose.yml` file format as Docker compose

Within a `docker-compose.yml` file we specify services, networks and volumes in `yaml` format. Each service corresponds to a container with its own configuration, such as `image`, `ports`, `volumes`, etc.

Example full stack application (Frontend, Backend and Database):

```yaml
version: '3'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DB_URL=mongodb://db:27017/myapp
    depends_on:
      - db

  db:
    image: mongo:latest
    volumes:
      - db_data:/data/db

volumes:
  db_data: {}
```

Breaking this down:

**Latest version of docker/podman compose files**:

```yaml
version: '3'
```

**Where we define our services, or related containers**:

```yaml
services:
  frontend:
  backend:
  db:
```

We must either specify a `image` and/or `build` instruction.

* `image` - Pull an pre-built image from a registry
* `build` - Specify a location of a `dockerfile` to build an image from, and use that. Podman uses a default naming convention to tag the built image. This convention is based on the directory name where the docker-compose.yml file is located, along with the service name defined in the compose file, followed by a tag that is usually latest.
* if using both `image` and `build`, podman will build the image with the name and optional tag.

```yaml
version: '3'
services:
  frontend:
    build: ./frontend

  backend:
    build: ./backend

  db:
    image: mongo:latest
```

Specify external ports to access the applications on. No need to expose the DB service externally:

```yaml
version: '3'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"

  backend:
    build: ./backend
    ports:
      - "5000:5000"

  db:
    image: mongo:latest

volumes:
  db_data: {}
```

Specify any environment variables and dependencies with `depends_on`

```yaml
version: '3'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DB_URL=mongodb://db:27017/myapp
    depends_on:
      - db

  db:
    image: mongo:latest
```

Followed by any persistent volumes:

```yaml
version: '3'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DB_URL=mongodb://db:27017/myapp
    depends_on:
      - db

  db:
    image: mongo:latest
    volumes:
      - db_data:/data/db

volumes:
  db_data: {}
```

## Understand container dependencies

From the example above, `depends_on` is used to define dependencies between containers. A typical example includes a backend/frontend service and a database.

## Working with environment variables

From the example above, `environment` is used to define environment variables used by the containers which can influence application behavior.

## Working with secrets

`environment` variables can be used to parse secret information however, it is not secure as it's visible in plain text:

```yaml
version: '3'
services:
  web:
    image: my-web-app
    environment:
      DATABASE_PASSWORD: mysecretpassword
```

Another way to accomplish this is to create a file that contains environment variable definitions and reference it:

```yaml
version: '3'
services:
  web:
    image: my-web-app
    env_file:
      - web.env
```

In web.env, you'd have:

```bash
DATABASE_PASSWORD=mysecretpassword
```

## Working with volumes

### Named Volumes

Here, db_data is a named volume that is mounted into the postgres container at /var/lib/postgresql/data.

```yaml
version: '3'
services:
  db:
    image: postgres
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data: {}
```

Note: Podman stores named volumes in a specific directory on the host system. This directory is usually under `/var/lib/containers/storage/volumes` (or a similar path, depending on your Podman configuration and installation). Within this directory, each named volume has its own subdirectory.

Named volumes can be managed with `podman volume`.

### Bind Mounts

Bind mounts are used to mount specific paths on the host to the container. They are useful for development purposes where you want to reflect code changes immediately in the container.

```yaml
version: '3'
services:
  app:
    image: my-app
    volumes:
      - ./app:/usr/src/app
```

In this example, the `./app` directory on the host is mounted into the `my-app` container at `/usr/src/app`.

### Anonymous Volumes

Anonymous volumes are not named and are often used for temporary or throwaway data that doesn't need to be persisted after the container is removed.

```yaml
version: '3'
services:
  app:
    image: my-app
    volumes:
      - /usr/src/app
```

## Working with configuration

With all configuration sections complete.  We can start up with:

`podman-compose up -d`

Note: `-d` runs it in detached mode.

Stopping and removing:

`podman-compose down`

If in doubt - `podman-compose --help`
