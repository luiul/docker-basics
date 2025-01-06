# How to Create a Great Local Python Development Environment with Docker

Docker is a tool for creating reproducible development environments. This guide shows you how to set up a local Python development environment using Docker.

## Commands

We use the following commands in this guide:

```shell
docker build -t my_app .
docker run -d --name my_app_container -p 80:80 my_app
docker run -d --name my_app_container -p 80:80 -v $(pwd):/code my_app
docker compose up --build -d
```

## Dockerize an App

Create a `Dockerfile`:

```Dockerfile
FROM python:3.10-slim
WORKDIR /code
COPY ./requirements.txt ./
RUN pip install --no-cache-dir --upgrade -r requirements.txt
COPY ./src ./src
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "80", "--reload"]
```

Build and run the Docker container:

```console
docker build -t fastapi-image .
docker run --name fastapi-container -p 80:80 fastapi-image
docker run -d --name fastapi-container -p 80:80 fastapi-image
```

Use the `Dockerfile` and the code in the `src` directory. For smaller image sizes, use slim or alpine base images.

## Immediate File Changes (Volumes)

Stop and remove the container:

```console
docker stop fastapi-container
docker ps -a
docker rm fastapi-container
```

Run the container with a volume to reflect immediate file changes:

```console
docker run -d --name fastapi-container -p 80:80 -v $(pwd):/code fastapi-image
```

## Docker Compose

Create a `docker-compose.yml` file:

```yml
services:
  app:
    build: .
    container_name: python-server
    command: uvicorn src.main:app --host 0.0.0.0 --port 80 --reload
    ports:
      - 80:80
      - 5678:5678
    volumes:
      - .:/code
```

Run the services:

```console
docker-compose up
docker-compose down
```

## Add More Services (e.g., Redis)

Extend `docker-compose.yml` to include additional services like Redis:

```yml
services:
  app:
    ...
    depends_on:
      - redis

  redis:
    image: redis:alpine
```

## Debug Python Code Inside a Container

Add the following code snippet to enable debugging:

```python
import debugpy

debugpy.listen(("0.0.0.0", 5678))

# Uncomment to wait for the debugger to attach
# debugpy.wait_for_client()
```

Update the `docker-compose.yml` file to expose the debug port:

```yml
services:
  app:
    ...
    ports:
      - 80:80
      - 5678:5678
```

Attach your debugger to the running container.

## Try a Python Version Easily with Docker

Use Docker to test different Python versions:

```console
docker pull python:3.11-slim
docker run -d -i --name python_dev python:3.11-slim
docker exec -it python_dev /bin/sh
```

## Further Resources

- [Debugging Dockerized ML Applications in Python](https://towardsdatascience.com/debugging-for-dockerized-ml-applications-in-python-2f7dec30573d)
- [Try Python Flask Redis Docker Compose](https://github.com/Wyntuition/try-python-flask-redis-docker-compose)
- [Docker Compose Getting Started Guide](https://docs.docker.com/compose/gettingstarted/)
- [Containerized Python Development](https://www.docker.com/blog/containerized-python-development-part-1/)

## Other Commands for Cleaning Up

Remove containers and images:

```console
docker rm container_name
docker image rm image_name
docker system prune
docker images prune
```

Check folder size:

```console
du -sh *
```
