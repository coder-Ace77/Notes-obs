
---

As your application grows, it often requires more than one container. Docker Compose allows you to **define and manage multi-container Docker applications** using a simple, human-readable YAML file called `docker-compose.yml`. Instead of running multiple commands, you define all your services in one file and launch everything with a single `docker-compose up`.

With Docker Compose you use a YAML configuration file, known as the Compose file, to configure your application’s services, and then you create and start all the services from your configuration with the Compose CLI.

The Compose file, or `compose.yaml` file, follows the rules provided by the Compose Specification in how to define multi-container applications. This is the Docker Compose implementation of the formal Compose Specification.

#### The Compose file:

The default path of compose file is `compose.yaml` that is placed in the working directory. `docker-compose.yaml` is actually the format from earlier versions and supported by the docker due to  backwards compatibility. If both files exist, Compose prefers the canonical `compose.yaml`.

The Docker CLI lets you interact with your Docker Compose applications through the `docker compose` command and its subcommands. If you're using Docker Desktop, the Docker Compose CLI is included by default.

A `docker-compose.yml` file is a YAML file that defines how Docker containers should behave in a multi-container setup. It specifies:

- What containers (services) to run
- What images or build instructions to use
- How to connect them
- Environment variables, volumes, ports, and much more

Basic syntax::

```yaml
version: '3.9'  # Compose file format version

services:       # Group of services (containers)
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: postgres
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass

volumes:        # Named volumes
  db_data:

networks:       # Custom networks (optional)
  app_net:
```

#### Anatomy of docker-compose file

We know that yaml file follows hierarical structure. On top level

1. version: specifies the version of docker compose file. Optional and not recommended for newer versions.
2. service: Core section where you define each container service.

	Each **service**:
		Runs one container (but can be scaled to more)
		Has its own configuration: image, ports, volumes, dependencies, etc.


> [!NOTE] Options in service
> 1. Image: specifies docker image to use.
> 2. Build: Tells to build an dockerfile from image.
> 3. ports: maps port from host ot container.
> 4. Volumes: mounts volume on containers.
> 5. environment: sets environment varibles.
> 6. depends_on: defines start up order
> 7. command: to override the default entry point.
> 8. network: to assign the service to one or more networks
> 9. restart: restart policy Options: `no`, `always`, `on-failure`, `unless-stopped`

```yaml

version:  '3.9'

services:
  web:
    image:nodejs
    ports:
	  - "<host>:<container>" #list here
	volumes:
	  - ./code:/app  #list here
	environments:
	  - NODE_ENV=production
	  - API_KEY=some key
	depends_on:
	  - db
	command: ["npm", "start"]  	
	networks:
	  - frontend
	  - backend
```

#### Other top level sections:

In Docker Compose, **top-level `volumes` and `networks`** are used to define reusable and configurable **storage** and **networking** components that can be shared across multiple services in your application. These are specified at the root (top level) of the `docker-compose.yml` file, separate from individual service definitions.

### Volumes (Top-level)

Top-level **`volumes`** define persistent storage that can be mounted into containers. When defined globally, these volumes can be shared by multiple services, allowing data to persist even if containers are stopped, recreated, or removed.

eg::

```yaml

services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
    driver: local
```

Docker containers have a life cycle and volumes do persist after a docker compose life cycle.

Even if the containers using them are stopped, removed, or recreated (`docker-compose down` or `docker-compose up --force-recreate`), the volumes **still exist** unless you explicitly remove them.

Also the volume will be created if not existed before.
### Networks (Top-level)

Top-level **`networks`** define custom Docker networks that control how containers communicate with each other. By default, Docker Compose creates a default network for all services, but defining custom networks gives more control over isolation, DNS resolution, and communication rules. Services can be attached to one or more of these custom networks.

```yaml

services:
  web:
    image: nginx
    networks:
      - frontend
  app:
    image: my-app
    networks:
      - frontend
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

```

- When you run `docker-compose up`, Docker automatically creates any defined networks if they don't already exist.
- Now networks won't persist by default but gets removed however the networks with external to be true will remain persistant. 


### Key terms in docker compose:

1. Service:  A **service** in Docker Compose is a definition of a container — essentially, it describes _how_ a container should run.

```yaml
services:
  web:
    image: nginx
    ports:
      - "80:80"
```

### Docker compose CLI basic commands:

```bash
docker compose up

docker compose up -d %% detached mode %%

```

what it does:

- Builds images if necessary
- Creates and starts containers
- Connects them to the default network
- Streams logs to your terminal (in undetached mode)

```shell
docker compose down
```

This command stops all the containers started by `up`, and removes containers, default networks, and optionally volumes.

```shell
docker compose down --volumes
```

Removes volumes that are orphan also.

docker compose build -> builds the images only but not runs them.
These are the images listed in `compose.yaml`

```shell
docker compose build

docker compose build web %% building only web service%%
```

```bash
%% to list all the containers %%
docker compose ps
```

`docker compose` is the **modern replacement** for the older `docker-compose` command. Instead of being a separate Python-based tool (`docker-compose`), it is now **natively integrated into the Docker CLI** as a **Go-based implementation**.

```bash
docker compose logs
docker compose logs -f web  # Follow logs for a specific service
```

Running a command inside a running service

```shell
docker compose exec web bash
```

Restarting services

```bash
docker compose restart web
```

#### Environment variables:

We can inject the values in compose files from environment variables as well. This allow us to not hard code the values.

```yaml
environment:
  - DB_USER=${POSTGRES_USER}
```

and its fetched from .env file

```env
POSTGRES_USER=admin
```

we can scale the services using docker compose scale flag but we also need some reverse proxy to handle the coming requests and load balance them.

