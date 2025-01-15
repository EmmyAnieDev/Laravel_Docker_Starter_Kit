# Laravel Demo App - Docker Setup

## What is Docker?

Docker is an open platform for developing, shipping, and running applications. It enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure the same way you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.

## Key Docker Concepts

### Dockerfile

A `Dockerfile` is a text document that contains all the commands you would normally execute manually to assemble an image. Using a `Dockerfile`, you can automate the steps of creating a Docker image, ensuring consistency across different environments.

**Example of a Dockerfile for Laravel:**

```Dockerfile
# Base image
FROM php:8.1-fpm

# Install dependencies
RUN apt-get update \
    && apt-get install -y \
       build-essential \
       libpng-dev \
       libjpeg-dev \
       libfreetype6-dev \
       libonig-dev \
       libxml2-dev \
       zip \
       unzip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Set working directory
WORKDIR /var/www

# Copy files
COPY . /var/www

# Install Composer
COPY --from=composer:2.2 /usr/bin/composer /usr/bin/composer
RUN composer install

# Expose port 9000 and start PHP-FPM
EXPOSE 9000
CMD ["php-fpm"]
```

### Docker Image

A Docker image is a lightweight, standalone, and executable software package that includes everything needed to run a piece of software, including the code, runtime, libraries, and system tools. Think of an image as a blueprint for creating containers.

**Building an image from a Dockerfile:**

```bash
docker build -t laravel-app .
```

### Docker Container

A Docker container is a runnable instance of a Docker image. It provides a lightweight and portable environment for running your application. You can create, start, stop, move, or delete a container using Docker commands.

**Running a container from an image:**

```bash
docker run -d -p 8000:80 laravel-app
```

### Docker Volumes

Volumes are used to persist data generated or used by Docker containers. By default, containers are stateless, and all data is lost when the container is stopped or removed. Volumes allow you to store data outside the container and reuse it across multiple containers.

**Example of using a volume:**

```bash
docker run -v $(pwd):/var/www -p 8000:80 laravel-app
```

### Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. Using a `docker-compose.yml` file, you can configure the services your application needs, such as a web server, database, and caching service, in a single file.

**Example of a docker-compose.yml for Laravel:**

```yaml
version: "3.8"

services:
    app:
        build:
            context: .
            dockerfile: Dockerfile
        volumes:
            - .:/var/www
        ports:
            - "8000:80"
        depends_on:
            - db

    db:
        image: mysql:8.0
        restart: always
        environment:
            MYSQL_DATABASE: laravel
            MYSQL_USER: root
            MYSQL_PASSWORD: secret
            MYSQL_ROOT_PASSWORD: secret
        volumes:
            - db-data:/var/lib/mysql

volumes:
    db-data:
```

**Starting services using Docker Compose:**

```bash
docker-compose up -d
```

## How to Run This Laravel App with Docker

1. Clone the repository.
2. Ensure Docker and Docker Compose are installed on your system.
3. Build the Docker image:
    ```bash
    docker-compose build
    ```
4. Start the application:
    ```bash
    docker-compose up -d
    ```
5. Open your browser and navigate to `http://localhost:8000`.

## Notes

-   Use `docker-compose down` to stop and remove containers, networks, and volumes.
-   For Laravel migrations, run:
    ```bash
    docker exec -it <container_name> php artisan migrate
    ```
