# Useful Docker Commands

## Intro

- **`docker run <image_name>`**  
  Runs a container based on the specified Docker image. If the image isn't available locally, Docker will attempt to pull it from Docker Hub or another registry.
  
- **`docker pull <image_name>`**  
  Pulls the latest version of the specified image from a Docker registry (like Docker Hub) to your local machine without running it.

- **`docker image ls`**  
  Lists all Docker images that are currently stored on your local machine. Useful for checking which images you have available.

- **`docker ps`**  
  Displays the currently running containers on your system, showing information like container ID, names, and resource usage.

- **`docker ps --all`**  
  Lists all containers on your system, including those that are stopped. This is useful for seeing containers that are not currently running.

## Docker Run Basics

- **`docker run -p <host_port>:<container_port> <image_name>`**  
  Runs a container from the specified image and maps a host port to the container port, allowing the container to communicate through the specified port.

- **`docker run -p <host_port>:<container_port> -d <image_name>`**  
  Runs the container in detached mode (`-d`), meaning it will run in the background. The terminal won't be attached to the container.

- **`docker run -p <host_port>:<container_port> -d --name <container_name> <image_name>`**  
  In addition to running the container in detached mode and mapping ports, this command also assigns a custom name to the container. This makes it easier to manage the container using its name instead of the container ID.

- **`docker stop <container_name>`**  
  Stops a running container using its name. This cleanly shuts down the container's processes.

- **`docker stop <container_id>`**  
  Similar to the previous command, but this stops a container using its ID instead of its name.

- **`docker container prune`**  
  Removes all stopped containers, freeing up system resources. This is a useful cleanup command to remove containers no longer in use.

- **`docker run -p <host_port>:<container_port> -d --rm <image_name>`**  
  Runs the container in detached mode with port mapping, but the `--rm` flag ensures the container is automatically removed when it stops. This is helpful for temporary containers you don't need to keep after they finish running.

## Tags

- **Tags are mutable**  
  Docker image tags (e.g., `nginx:latest`) are mutable, meaning they can point to different versions of an image over time. This is why it's recommended to use version-specific tags when you need consistency.

- **`docker run <image_name>:<tag>`**  
  Runs a specific version of the image, as identified by the tag (e.g., `nginx:1.21.0`). This ensures you are running the exact version of the image you expect.

- **`docker image ls --digests`**  
  Lists images with their unique digest information. Digests provide an immutable reference to an image, ensuring you're always using the exact same image content.

- **`docker run <image_name>@<digest>`**  
  Runs a Docker image based on its digest instead of a tag. Using the digest guarantees that the exact image content is used, as digests are immutable.

## Example

- **`docker run -p 80:80 -d nginx`**  
  This command runs the official Nginx image in detached mode and maps port 80 of the container to port 80 on your host machine. It allows you to access the Nginx web server through `http://localhost:80`.

## Continuation

## Runtime

- **`docker run -e <key>=<value> <image_name>:<tag> <command>`**  
  Runs a container based on the specified image and executes the specified command within the container. This is useful for running one-off commands or scripts inside a container.

- **`docker exec -it <container_name> <command>`**  
  Executes a command inside a running container. The `-it` flag allows you to interact with the command in an interactive terminal session (interactive and pseudo-TTY).

## Persistent Storage

Three types of mounts:

- Volume mounts
- Bind mounts
- Tmpfs mounts (non-persistent)

### Volume Mounts

- Often better in production
- Not dependent on host filesystem
- Easy to share across containers
- Can use remote or cloud storage (e.g., AWS EFS, NFS, SSHFS)
- Container does not need access to the host
- Not convenient for sharing with the host system

- **`docker run -v mydata:/data <image_name>:<tag> <command>`**  
  Runs a container with a volume named `mydata` mounted at the `/data` directory inside the container. This allows you to persist data across container runs and share data between containers.

### Bind Mounts

- Often convenient in development
- Quickly share data with the host system
  - Gives the container access to the host system
  - Consider using a read-only mount (e.g., `-v ./mydata:/path/in/container:ro`)

- **`mkdir mydata && docker run -v $(pwd)/mydata:/data <image_name>:<tag> <command>`**  
  Creates a local directory named `mydata` and mounts it to the `/data` directory inside the container. This is an example of a bind mount, which allows you to share files between the host and container.

## Custom Docker Images

continue here...

## Examples

- **`docker run -e ABC=123 -e DEF=456 python:3.12 python -c 'import os; print(os.environ)'`**  
  Runs a Python container with environment variables (`ABC=123`, `DEF=456`) and prints them inside the container.

- **`docker exec -it <container_name> /bin/bash`**  
  Accesses a running container's shell (`/bin/bash`) for interactive use.

- **`docker run python:3.12 python -c 'f="/data.txt";open(f, "a").write(f"Ran!\n");print(open(f).read())'`**  
  Runs a Python container and writes data to a file inside the container.

- **`docker run -v mydata:/data python:3.12 python -c 'f="/data/data.txt";open(f, "a").write(f"Ran!\n");print(open(f).read())'`**  
  Runs a Python container with a volume named `mydata` mounted at `/data`. The command writes to a file in the volume and reads its contents, demonstrating persistent storage.

## Appendix

### Read-Only Bind Mounts

A **bind mount** allows you to map a directory or file from your host system into a container. This can be useful for sharing configuration files, code, or other data between the host and the container.

**Read-only bind mounts** are a more secure way to provide access to files or directories from the host to the container without allowing the container to modify them. This is particularly useful when you want the container to have access to data that should not be changed (e.g., configuration files, static resources, etc.).

#### Use Cases for Read-Only Bind Mounts

- **Static Configuration Files**: You might want to provide a configuration file to the container but prevent the container from altering it.
- **Static Website Files**: For containers serving static websites, you can mount the website files read-only to prevent the web server (or any processes in the container) from modifying them.
- **Shared Libraries**: If the container requires access to a host directory containing shared libraries, a read-only mount can ensure that the container doesn’t accidentally alter the libraries.

#### Example of Read-Only Bind Mount

Let's say you are running a web server container (e.g., Nginx) and you want to serve static files located on your host. You can mount the directory containing the static files into the container in **read-only mode** to ensure that the files cannot be altered by any processes running inside the container.

### Example

```bash
docker run -d -p 8080:80 -v $(pwd)/static:/usr/share/nginx/html:ro nginx
```

#### Explanation

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

#### When to Use Read-Only Bind Mounts

- **Security Concerns**: If you're concerned about potential changes to sensitive or critical files (such as configuration files), use a read-only mount to ensure the container has read access only.
- **Immutable Data**: If the data you’re providing to the container doesn’t need to be modified (e.g., assets for a web app or libraries), using read-only mounts helps protect data integrity.
- **Auditing and Compliance**: In scenarios where changes to data need to be controlled and audited, ensuring that containers can only read but not write can help you maintain compliance.

By using read-only bind mounts, you effectively minimize the risk of accidental or malicious changes to important files inside your containers, adding a layer of security and control.
