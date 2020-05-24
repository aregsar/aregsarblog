# Production Oriented Laravel Project Setup Quickstart

May 24, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Production Oriented Laravel Project Setup Quickstart](https://aregsar.com/blog/2020/production-oriented-laravel-project-setup-quickstart)

# Seting up a new Laravel project

First create a new project as instructed here:

[Create Laravel Project With Tailwind UI and Auth](https://aregsar.com/blog/2020/create-laravel-project-with-tailwind-ui-and-auth)

## Create the data directory

Create a new data directory in the Laravel project root directory:

```bash
echo '/data' >> .gitignore
mkdir data
```

## Create the docker-compose file

Create a new docker-compose.yml file in the Laravel project root directory:

```bash
touch docker-compose.yml
```

Next add the following content to the docker-compose.yml file

```yaml
version: "3.1"
services:

    mysqltest:
      image: mysql:8.0
      container_name: app-mysql-test
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
      ports:
        - "8011:3306"

    mysql:
      image: mysql:8.0
      container_name: app-mysql
      volumes:
        - ./data/mysql:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
      ports:
        - "8001:3306"

    redis:
      image: redis:alpine
      container_name: app-redis
      command: redis-server --appendonly yes --requirepass "${REDIS_PASSWORD}"
      volumes:
      - ./data/redis:/data
      ports:
        - "8002:6379"

    mailhog:
      image: mailhog/mailhog:latest
      container_name: app-mailhog
      ports:
        - "8001:8025"

    elasticsearch:
      image: elasticsearch:6.5.4
      container_name: app-elasticsearch
```
