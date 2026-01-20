
---

A **Docker image** is a lightweight, standalone, and executable package that includes everything needed to run a piece of software

- **Code**
- **Runtime**
- **Libraries**
- **Environment variables**
- **Configuration files**

Think of it as a snapshot of a file system and application settings.

Docker images are **read-only** templates. When you run an image, it creates a **Docker container**, which is a running instance of that image.

Docker images can be created using the Dockerfile and then using build command.

#### Common Dockerfile Instructions:

- `FROM`: **Specifies the base image** (e.g., `FROM ubuntu`, `FROM node:18`)
- `RUN`: Executes a command during the build (e.g., `RUN apt-get install -y curl`)
- `COPY` / `ADD`: Copies files from the host into the image
- `CMD`: Default command to run when the container starts
- `ENTRYPOINT`: Sets the main command for the container
- `EXPOSE`: Declares the port the container listens on
- `WORKDIR`: Sets the working directory inside the image

#### Dockerfile:

A Dockerfile is a **plain text file** containing a series of **instructions** and **arguments** that Docker uses to build an image. Each instruction is written in **uppercase** (by convention, though lowercase works too) followed by one or more arguments.

### Basic Syntax Rules:

1. **One instruction per line**  
    Each line generally contains one Dockerfile instruction (like `FROM`, `RUN`, `COPY`, etc.).
2. **Instructions are case-insensitive**  
    By convention, they are uppercase (e.g., `FROM`, `RUN`), but lowercase also works.
3. **Arguments follow instructions**  
    After the instruction, you provide the necessary arguments separated by spaces.
4. **Comments start with `#`**  
    Anything following `#` on a line is ignored (used for comments).
5. **Line continuation with backslash `\`**  
    For long commands (especially with `RUN`), use `\` at the end of a line to continue on the next line.

### Base Images

A **base image** is the starting point for building your Docker image. Common base images include:

- `alpine` – Small and secure Linux distro
- `ubuntu` – Full-featured Linux distro
- `node`, `python`, `golang`, etc. – Language-specific images with tools preinstalled

There are two types of base images:

1. **Parent images**: Built on top of another image (e.g., `FROM ubuntu`)
2. **Scratch image**: The most minimal image – actually an empty image.

**you can create a Docker image without any base image** by using the special keyword:

```Dockerfile
FROM scratch
```

This is called a **scratch image**, which is literally an empty image – no OS, no libraries, nothing.

Note that this scratch image is not usually very useful however it provides fine tuned controlled over images. 

### Docker image layers:

Docker images are built in **layers**, where each layer represents an instruction in the Dockerfile. When you write a Dockerfile, every command like `FROM`, `RUN`, `COPY`, or `ADD` creates a new layer on top of the previous ones. These layers are stacked to form the final image. This layered architecture is powerful because Docker **caches** these layers. If nothing changes in a particular layer during a rebuild, Docker reuses the cached layer instead of rebuilding it from scratch, which significantly speeds up the build process. Layers also enable Docker images to be space-efficient since multiple images can share common layers without duplicating data.

The `RUN` instruction in a Dockerfile is used to execute commands during the image build process. This is most commonly used to install packages or perform setup tasks inside the image. Every time you use `RUN`, Docker executes the command in a new intermediate container and commits the results as a new image layer. Because each `RUN` instruction creates its own layer, it’s best practice to combine related commands into a single `RUN` statement (using `&&`) to reduce the total number of layers and keep the image size smaller.

```Dockerfile
FROM ubuntu:20.04          # Layer 1: base operating system

# Layer 2: update package lists and install curl and git in a single RUN to optimize layers
RUN apt-get update && apt-get install -y curl git && rm -rf /var/lib/apt/lists/*

# Layer 3: copy local application files into the image
COPY ./myapp /app

# Layer 4: set working directory inside the image
WORKDIR /app

EXPOSE 80

# Layer 5: default command to run when container starts
CMD ["./start.sh"]
```

##### Docker layers internal working:

At its core, a **Docker layer** is a **read-only filesystem snapshot** created from a command in your Dockerfile. When Docker builds an image, each instruction (like `RUN`, `COPY`, `ADD`, etc.) generates a new layer on top of the existing image layers. These layers stack up to form the complete image.

Think of layers like transparent sheets stacked one on top of another. When combined, they create the full picture (the complete filesystem and environment for your container).

#### How Layers Work Internally

- Each layer contains **only the differences (deltas)** compared to the previous layer.
- For example, if you install `curl` in a `RUN` command, the new layer only stores the changes made (new files added, modified config, etc.).
- Docker uses a **union filesystem** (like OverlayFS on Linux) to **merge these layers into one unified view** when running containers.
- Because layers are **immutable**, once a layer is created, it never changes. Instead, new layers are added on top to reflect changes.

##### What Is Build Context?

When you run `docker build`, Docker needs to know **what files** it has access to while building the image. These files are provided in the **build context**, which is typically the current directory (`.`) you specify at the end of the `docker build` command.

```bash
docker build -t my-app .
```

- Sending a large build context slows down builds and uses more resources.
- Sensitive files (like `.env`, SSH keys, or config files) may accidentally be included if not excluded.

A `.dockerignore` file works just like `.gitignore`. It tells Docker what **not to send** in the build context.

#### Efficient practices:

Docker caches each instruction (`RUN`, `COPY`, etc.) as an image layer. If a layer hasn't changed, Docker can reuse the cached result to **speed up builds**.

- Place frequently-changing lines (like `COPY . .`) **lower in the Dockerfile**.
- Group related `RUN` commands to reduce layers:

#### EXPOSE:

The `EXPOSE` instruction **declares** which **network ports** the container will listen on at runtime. It tells Docker and anyone using the image that the application inside the container expects traffic on those ports.

- It **does not actually publish or open** the port to the host or external network by itself.
- It’s mainly **documentation** inside the image, signaling which ports are intended to be used.
- When you run the container, you still need to **map ports explicitly** using `-p` or `--publish` (e.g., `docker run -p 8080:80`) to make the port accessible outside the container.

### Docker entry point vs CMD:

Both **`ENTRYPOINT`** and **`CMD`** define **what gets executed when a Docker container starts**, but they serve slightly different purposes and behave differently.

- **`CMD`** specifies the **default command** and arguments to run **when the container starts**.
- It **can be overridden** by passing a different command when running the container.
- It’s ideal for **providing default behavior** that the user might want to change.

```Dockerfile
FROM ubuntu
CMD ["echo", "Hello from CMD"]
```

```bash
docker build -t cmd-example .
docker run cmd-example           # Output: Hello from CMD
docker run cmd-example ls -l     # CMD is overridden; Output: result of `ls -l`
```

- **`ENTRYPOINT`** is used to **define the main executable** that will always run.
- It **cannot be overridden** by normal command-line arguments (unless `--entrypoint` is used).
- It’s best when you want your container to always run a specific application.

```Dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
```

```bash
docker build -t entrypoint-example .
docker run entrypoint-example Hello World   # Output: Hello World
```

It is important to understand that both can be combined as well.

- `ENTRYPOINT` defines the **main executable**.
- `CMD` provides **default arguments** to it (which can be overridden at runtime).

```Dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello from CMD"]
```

```bash
docker build -t combo-example .
docker run combo-example               # Output: Hello from CMD
docker run combo-example "Hi there!"   # Output: Hi there!
```

CMD itself comes in two flavours:

```Dockerfile
CMD ["executable", "param1", "param2"]        # exec form (preferred)
CMD command param1 param2                      # shell form
```

#### What is `ENV` in Dockerfiles?

The `ENV` instruction in a Dockerfile is used to **set environment variables** inside the Docker image. These variables are available to all subsequent instructions during the build and also when the container runs.

For example while building we can set up the path of some server.

```Dockerfile
FROM ubuntu:20.04

ENV APP_HOME=/app
ENV APP_PORT=8080

# Alternate

ENV <key>=<value> [<key>=<value> ...]

WORKDIR $APP_HOME

EXPOSE $APP_PORT

CMD ["./start-server.sh"]

```

##### Using `ENV` vs `ARG`

- `ARG` variables are **only available during build time**.
- `ENV` variables are **available at build time and runtime** inside the container.

One can override the env when running build

```bash
docker run -e APP_PORT=9090 my-image
```

#### Work directory:

The `WORKDIR` instruction sets the **working directory** for any subsequent commands in the Dockerfile, such as `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD`.

- When you use `WORKDIR /app`, Docker changes the current directory inside the container to `/app`.
- If the directory doesn’t exist, Docker **creates it automatically**.
- All following commands will be executed relative to this directory.