<!-- omit in toc -->
# Docker Basics

<!-- omit in toc -->
## Table of Contents

- [1. Intro](#1-intro)
- [2. Docker Run Basics](#2-docker-run-basics)
- [3. Tags](#3-tags)
- [4. Runtime and Debugging](#4-runtime-and-debugging)
- [5. Persistent Storage](#5-persistent-storage)
  - [5.1. Volumes](#51-volumes)
  - [5.2. Bind Mounts](#52-bind-mounts)
- [6. Custom Docker Images](#6-custom-docker-images)
- [7. Layers](#7-layers)
- [8. Appendix](#8-appendix)
  - [8.1. Read-Only Bind Mounts](#81-read-only-bind-mounts)
  - [8.2. Layers](#82-layers)
  - [8.3. Misc](#83-misc)

## 1. Intro

- **`docker run <image_name>`**  
  Runs a container based on the specified Docker image. If the image isn't available locally, Docker will attempt to pull it from Docker Hub or another registry. Note that each time you run this command, a new container instance is created based on the image you specify. This means that changes made to the container (e.g., file modifications, installed packages) are not persisted across runs.
  
- **`docker pull <image_name>`**  
  Pulls the latest version of the specified image from a Docker registry (like Docker Hub) to your local machine without running it.

- **`docker image ls`**  
  Lists all Docker images that are currently stored on your local machine. Useful for checking which images you have available.

- **`docker ps`**  
  Displays the currently running containers on your system, showing information like container ID, names, and resource usage.

- **`docker ps --all`**  
  Lists all containers on your system, including those that are stopped. This is useful for seeing containers that are not currently running.

Example:

```shell
docker run nginx
```

This command runs an Nginx container based on the official Nginx image. If the image is not available locally, Docker will pull it from Docker Hub. The container will start and run the Nginx web server.

## 2. Docker Run Basics

- **`docker run -p <host_port>:<container_port> <image_name>`**  
  Runs a container from the specified image and maps a host port to the container port, allowing the container to communicate through the specified port.

- **`docker run -p <host_port>:<container_port> -d <image_name>`**  
  Runs the container in detached mode (`-d`), meaning it will run in the background. The terminal won't be attached to the container.

- **`docker run -p <host_port>:<container_port> -d --name <container_name> <image_name>`**  
  In addition to running the container in detached mode and mapping ports, this command also assigns a custom name to the container. This makes it easier to manage the container using its name instead of the container ID.

- **`docker stop <container_name|container_id>`**  
  Stops a running container using its name. This cleanly shuts down the container's processes.

- **`docker container prune`**  
  Removes all stopped containers, freeing up system resources. This is a useful cleanup command to remove containers no longer in use.

- **`docker run -p <host_port>:<container_port> -d --rm <image_name>`**  
  Runs the container in detached mode with port mapping, but the `--rm` flag ensures the container is automatically removed when it stops. This is helpful for temporary containers you don't need to keep after they finish running.

Example:

```shell
docker run -p 8080:80 -d nginx
```

This command runs an Nginx container in detached mode, mapping port 80 of the container to port 8080 on your host machine. You can access the Nginx web server through `http://localhost:8080`.

## 3. Tags

- **Tags are mutable**  
  Docker image tags (e.g., `nginx:latest`) are mutable, meaning they can point to different versions of an image over time. This is why it's recommended to use version-specific tags when you need consistency.

- **`docker run <image_name>:<tag>`**  
  Runs a specific version of the image, as identified by the tag (e.g., `nginx:1.21.0`). This ensures you are running the exact version of the image you expect.

- **`docker image ls --digests`**  
  Lists images with their unique digest information. Digests provide an immutable reference to an image, ensuring you're always using the exact same image content.

- **`docker run <image_name>@<digest>`**  
  Runs a Docker image based on its digest instead of a tag. Using the digest guarantees that the exact image content is used, as digests are immutable.

Example:

```shell
docker run -p 80:80 -d nginx:1.21.0-bookworm
```

This command runs the official Nginx image in detached mode and maps port 80 of the container to port 80 on your host machine. It allows you to access the Nginx web server through `http://localhost:80`. Additionally, the `1.21.0` tag ensures you're using a specific version of the Nginx image. In production, it's recommended to use version-specific digests for even greater consistency.

## 4. Runtime and Debugging

- **`docker run -e <key>=<value> <image_name>:<tag> <command>`**  
  Runs a container based on the specified image and executes the specified command within the container. This is useful for running one-off commands or scripts inside a container.

- **`docker exec -it <container_name> <command>`**  
  Executes a command inside a running container. The `-it` flag allows you to interact with the command in an interactive terminal session (interactive and pseudo-TTY).

Example:

```shell
docker run -e ABC=123 -e DEF=456 python:3.13-slim-bookworm python -c '
import os; print(os.environ)
' 
# or
docker run -e ABC=123 -e DEF=456 ghcr.io/astral-sh/uv:0.5.14-python3.13-bookworm-slim python -c '
import os; print(os.environ)
' 
```

This command runs a Python container with environment variables `ABC` and `DEF` set to `123` and `456`, respectively. The Python command prints the environment variables inside the container.

```shell
docker run -e ABC=123 -e DEF=456 -it ghcr.io/astral-sh/uv:0.5.14-python3.13-bookworm-slim /bin/bash
```

This command runs a Python container with environment variables `ABC` and `DEF` set to `123` and `456`, respectively. It starts an interactive shell session (`/bin/bash`) inside the container, allowing you to explore the container environment.

- **`docker logs <container_name|container_id>`**
  Displays the logs of a running container. This is useful for debugging issues or monitoring the output of a containerized application.
- **`docker exec -it <container_name|container_id> /bin/bash`**
  Accesses a running container's shell (`/bin/bash`) for interactive use. This is helpful for debugging, inspecting the container environment, or running commands inside the container.

## 5. Persistent Storage

Let begin with an example:

```shell
docker run python:3.13-slim-bookworm python -c '
f = "/msg.txt"
open(f, "a").write("Executables in the container ran!\n")
print(open(f).read())
'
```

Note that the file `data.txt` is created and written to inside the container. However, when the container stops, the file is lost because the container's filesystem is ephemeral. To persist data across container runs, you can use **volumes** or **bind mounts**. Tmpfs mounts are also available for non-persistent storage (not covered here).

### 5.1. Volumes

```shell
docker run -v docker_managed_volume:/app_data python:3.13-slim-bookworm python -c '
f = "/app_data/msg.txt"
open(f, "a").write("Executables in the container ran!\n")
print(open(f).read())
'
```

In this example, a volume named `docker_managed_volume` is mounted at the `/app_data` directory inside the container. This allows you to persist data across container runs and share data between containers. Note each docker run call creates a new container instance with a new filesystem but the volume is shared between them. Note that the volume is managed by Docker and not easily accessible from the host system.

Volumes are a good choice for **production** environments where you need to persist data across container runs and share data between containers. They are also useful for storing data that needs to be accessed by multiple containers. Docker volumes are managed by Docker and are not directly accessible from the host system, providing an additional layer of isolation and security. They can use remote or cloud storage solutions (e.g., AWS EFS, NFS, SSHFS) for more advanced use cases.

### 5.2. Bind Mounts

In the previous example we used a volume mount. Now let's see how to use a bind mount:

```shell
mkdir host_managed_volume &&
docker run -v $(pwd)/host_managed_volume:/app_data python:3.13-slim-bookworm python -c '
file_path = "/app_data/msg.txt"
open(file_path, "a").write("Executables in the container ran!\n")
print(open(file_path).read())
'
```

Note that the syntax is similar to volume mounts, but the difference is that bind mounts are directly linked to a directory on the host system. This allows you to easily share files between the host and container. The data is stored on the host system and is accessible even when the container is not running.

Bind mounts are useful for **development** environments where you want to share code or data between the host and container. They are also convenient for sharing configuration files or other resources that need to be accessed by the container. However, bind mounts can be less secure than volumes because they give the container direct access to the host filesystem. For added security, consider using read-only bind mounts.

## 6. Custom Docker Images

- **`docker build -t <image_name> -f <Dockerfile_path> .`**  
  Builds a Docker image based on the specified Dockerfile. The `-t` flag assigns a tag to the image, making it easier to reference. The `.` at the end specifies the build context, i.e., the directory containing the Dockerfile and any other files needed for the build process.

Example:

```shell
docker build -t my-website -f ./Dockerfile .
# or to simplify the command
docker build -t my-website . &&
docker run -p 80:80 -d my-website &&
docker exec -it heuristic_kare /bin/bash &&
cat /usr/share/nginx/html/index.html
# Output: <content of the index.html file>
```

After doing any change to the Dockerfile, you need to rebuild the image and run the container again. This is because the image is a snapshot of the filesystem at the time of the build, and changes made to the filesystem after the build are not reflected in the image.

Useful Dockerfile instructions:

- **`FROM <base_image>:<tag>`**: Specifies the base image to build upon. This is the starting point for your custom image. You can use official images from Docker Hub or other registries, or you can use other custom images you've built. The `tag` is optional and specifies a specific version of the base image. If omitted, Docker uses the `latest` tag by default.
- **`COPY <src> <dest>`**: Copies files or directories from the build context to the image. This is useful for adding application code, configuration files, etc. to the image. The `src` path is relative to the build context. The `dest` path is the location in the image where the files will be copied.
- **`RUN <command>`**: Executes a command during the build process. This is useful for installing packages, setting up the environment, etc. Each `RUN` instruction creates a new layer in the image. To reduce the image size, consider chaining commands together (e.g., `RUN apt-get update && apt-get install -y package`).
- **`WORKDIR <path>`**: Sets the working directory for any subsequent `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, or `ADD` instructions. This is where commands will be executed inside

## 7. Layers

Similar to image diffs, each command in a Dockerfile creates a layer. Layers are immutable, and images are composed of these layers. When you change a command in a Dockerfile, Docker rebuilds the image starting from the instruction that changed. This behavior allows Docker to reuse previously built layers, improving the build process’s efficiency.

Caching Rules:

1. Changing a command invalidates the cache for the current command and all subsequent commands.
2. Most commands are cached by default, even if they aren’t deterministic. For example, RUN apt-get install -y package will be cached unless the package list changes, even though the command itself is not deterministic.

**Warning**:

A RUN command that deletes files creates a new layer, but the files remain in the previous layer. Therefore, avoid placing sensitive information in Docker images (even temporarily) and then deleting it. Since deleted data exists in prior layers, it can be recovered by a malicious user.

## 8. Appendix

This section covers additional topics or expandsm on concepts discussed in the main content.

### 8.1. Read-Only Bind Mounts

A **bind mount** allows you to map a directory or file from your host system into a container. This can be useful for sharing configuration files, code, or other data between the host and the container.

**Read-only bind mounts** are a more secure way to provide access to files or directories from the host to the container without allowing the container to modify them. This is particularly useful when you want the container to have access to data that should not be changed (e.g., configuration files, static resources, etc.).

The use cases for read-only bind mounts include:

- **Static Configuration Files**: You might want to provide a configuration file to the container but prevent the container from altering it.
- **Static Website Files**: For containers serving static websites, you can mount the website files read-only to prevent the web server (or any processes in the container) from modifying them.
- **Shared Libraries**: If the container requires access to a host directory containing shared libraries, a read-only mount can ensure that the container doesn’t accidentally alter the libraries.

Let's say you are running a web server container (e.g., Nginx) and you want to serve static files located on your host. You can mount the directory containing the static files into the container in **read-only mode** to ensure that the files cannot be altered by any processes running inside the container.

```bash
docker run -d -p 8080:80 -v $(pwd)/static:/usr/share/nginx/html:ro nginx
```

In this command:

- **`-d`**: Runs the container in detached mode.
- **`-p 8080:80`**: Maps port 8080 on your host to port 80 in the container (standard web server port).
- **`-v $(pwd)/static:/usr/share/nginx/html:ro`**:
  - `$(pwd)/static`: Mounts the `static` directory from your current working directory.
  - `/usr/share/nginx/html`: This is where Nginx looks for static files to serve.
  - `:ro`: Mounts the directory as **read-only**, preventing the container from modifying any files inside `/usr/share/nginx/html`.
- **`nginx`**: Runs the official Nginx image.

In this example:

- The **host directory** (`static`) contains static HTML files you want the Nginx server to serve.
- The container can access and serve the files, but it cannot make changes to them because of the read-only flag (`:ro`).

Read-only bind mounts are useful in various scenarios, including:

- **Security Concerns**: If you're concerned about potential changes to sensitive or critical files (such as configuration files), use a read-only mount to ensure the container has read access only.
- **Immutable Data**: If the data you’re providing to the container doesn’t need to be modified (e.g., assets for a web app or libraries), using read-only mounts helps protect data integrity.
- **Auditing and Compliance**: In scenarios where changes to data need to be controlled and audited, ensuring that containers can only read but not write can help you maintain compliance.

By using read-only bind mounts, you effectively minimize the risk of accidental or malicious changes to important files inside your containers, adding a layer of security and control.

### 8.2. Layers

Docker images are built using a layered filesystem. Each instruction in a Dockerfile creates a new layer in the image. When you build an image, Docker caches the layers to improve build performance. If you make a change to a Dockerfile instruction, Docker rebuilds the image starting from the instruction that changed. This allows Docker to reuse previously built layers, speeding up the build process.

### 8.3. Misc

Rules for optimizing Docker images:

- **Combine Commands**: Combine multiple `RUN` instructions into a single `RUN` instruction to reduce the number of layers created. For example, instead of running `apt-get update` and `apt-get install` in separate `RUN` instructions, combine them into one.
- **Use .dockerignore**: Create a `.dockerignore` file to exclude unnecessary files and directories from the build context. This reduces the size of the build context and speeds up the build process.
- **Order Instructions**: Place instructions that change frequently (e.g., copying application code) towards the end of the Dockerfile. This allows Docker to reuse cached layers for instructions that change less frequently.
- **Use Multi-Stage Builds**: Use multi-stage builds to separate build dependencies from the final image. This helps reduce the size of the final image by only including necessary files and dependencies.
- **Minimize Image Size**: Remove unnecessary files, dependencies, and build artifacts from the final image to reduce its size. This can be done by using smaller base images, cleaning up after installation, and avoiding unnecessary packages.
- **Use Specific Tags**: Use specific tags for base images and dependencies to ensure consistency and avoid unexpected changes. This helps maintain reproducibility and stability in your Docker images.
- **Optimize Layers**: Be mindful of how layers are created in your Dockerfile. Avoid creating unnecessary layers and consider the impact of each instruction on the final image size.
