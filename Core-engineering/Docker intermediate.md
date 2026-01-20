
---

### Docker volumes:

Docker volumes are used to persist data explicit to the life cycle of container. Which means even if the container is removed still we can use the data.

Types of volumes:

### 1. **Named Volumes**

- **Created and managed by Docker**.
- Stored in Docker's internal storage location (usually `/var/lib/docker/volumes` on Linux).
- Automatically created if referenced in a Compose file or `docker run`.
- **Reusable** and can be shared between multiple containers.

eg in compose:

```yaml
volumes:
  db-data:
  
services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data  # this is mounting 

```

At the top level of the `docker-compose.yml` file, this declares a **named volume** called `db-data`.

This does **not create it yet**, but tells Docker Compose:  
🗂️ “I want to use a persistent, named volume called `db-data`.” Docker will manage this volume and we might see something like 

on volume look up

```bash
docker volume ls
```

```wasm
local               yourprojectname_db-data
```

```yaml
services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data
```

- **`db-data` (host)**: A Docker-managed volume that lives **on your host machine**, not inside the container.
- **`/var/lib/postgresql/data` (container)**: This is where **PostgreSQL stores its database files** inside the container.

### 2. **Anonymous Volumes**

- Also created by Docker, but **without a name**.
- You can spot them by their randomly generated names.
- Used when you define volumes in `docker run` or Compose **without naming them**.

For example if u write something like this inside services then:

```yaml
volumes:
  - /app/data
```

This mounts a volume at `/app/data` inside docker container, and Docker assigns a random name. 

### 3. **Host Volumes (Bind Mounts)**

- Map a specific **host directory** or file into a container.
- Gives the container **direct access to host file system**, which is useful for development or accessing config/log files.

For example if you write something like this inside service.
So first part defines the path where you want to mount on host and second defines the path in docker container.

```yaml
volumes:
  - ./config:/app/config
```

Observe that how docker can figure out which was host path and which was name of docker volume.

| Volume Type              | Created By   | Location        | Use Case                        | Persist After Container? | Shareable |
| ------------------------ | ------------ | --------------- | ------------------------------- | ------------------------ | --------- |
| Named Volume             | Docker       | Docker managed  | Persistent app data             | ✅ Yes                    | ✅ Yes     |
| Anonymous Volume         | Docker       | Docker managed  | Temporary data, not reused      | ✅ (but hard to reuse)    | ❌ No      |
| Host Volume (Bind Mount) | User-defined | Host filesystem | Development, custom config/logs | ✅ Yes                    | ✅ Yes     |

### Docker network:

A **Docker network** is a virtual network that connects Docker containers. Think of it as a **bridge** that allows containers to talk to each other or the outside world (like your browser or another system).

Types:

### 1. **Bridge Network** (default)

The **bridge** network is Docker’s **default network driver** for containers running on a single host. When you start a container without explicitly specifying a network, Docker connects it to the default `bridge` network. This network mode **creates a private virtual network** on your host, and each container attached to it gets its **own virtual Ethernet interface** and private IP address within that network. The containers can communicate with each other via these internal IPs or via container names using Docker's built-in **DNS service**.

Bridge networks are **isolated from the host machine’s network**, which means that services running inside containers aren't accessible from outside unless you explicitly publish their ports using the `-p` or `--publish` flag (or `ports:` in Compose). For example, mapping `8080:80` exposes the container’s port 80 to the host’s port 8080. This provides a layer of security and flexibility, especially for local development or small applications that need basic isolation.

### 2. Host Network

The **host** network mode removes the **network isolation** between the Docker container and the host. Instead of creating a virtual network interface and assigning a separate IP address to the container, Docker simply allows the container to use the **host’s networking stack directly**. This means that the container shares the **same IP address and ports** as the host machine, and any ports opened in the container are also opened on the host.

This mode is useful when you need **maximum network performance**, or when a container needs to access low-level network features that don’t work well with virtual networks, such as certain **monitoring tools**, **VPN clients**, or **media servers**. There’s **no need for port mapping** with host mode, since the container uses the host’s ports directly.

```bash
docker run --network host nginx
```

### 3. Docker compose

In Docker Compose, **networks** are used to define how containers communicate with each other. By default, when you run `docker-compose up`, Docker creates a **default network** for all services within the same Compose project. All containers in the project are connected to this network and can communicate using **service names** as hostnames.

However, Docker Compose lets you define **custom networks** at the top level of your `docker-compose.yml` file to give you more control over container communication, isolation, and networking configurations.

```yaml
version: "3.9"
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
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

There is a concept of external networks in docker compose

You can also attach services to existing Docker networks not created by Compose by marking the network as `external`.

```yaml
networks:
  external_net:
    external: true

services:
  app:
    image: my-app
    networks:
      - external_net
```


### 4.Docker logs

Every Docker container has two primary output streams:

- **STDOUT** (standard output)
- **STDERR** (standard error)

When a container runs, it may print logs, messages, or errors to these streams — `docker logs` lets you **view that output history**. It’s like looking at the console output of a running process.

```bash
docker logs <container_id_or_name>
```

However we also have some more options:

|Option|Description|
|---|---|
|`-f`|Follow logs (like `tail -f`), live streaming as new logs appear|
|`--tail N`|Show only the last N lines of logs|
|`--since`|Show logs since a specific time or timestamp|
|`--timestamps`|Show timestamps with log lines|
|`-n`|Same as `--tail` (show last N lines)|
-f is very important as it allows us to tap into the live stream of a running container.

```bash
docker logs -f --tail 50 my_container
```

#### Logs in docker compose:

- `docker compose logs` shows logs from **all containers** started by your Compose project.
- You can filter logs by **service name** to see logs for just one service.
- By default, logs are combined in a single stream but labeled by container/service.

eg:

```bash
docker compose logs
docker compose logs -f
docker compose logs web

docker compose logs -f db  ## following logs of a db

docker compose logs web > web.log  # redirecting logs to files
docker compose logs db > db.log
docker compose logs -f web > web.log
```

#### Viewing Logs via `docker logs` vs `docker compose logs`

- `docker logs <container_id>` only works **container by container**.
- `docker compose logs` aggregates logs from all containers defined in your Compose project.
- Compose uses container names like `projectname_service_1` so logs are prefixed accordingly.

