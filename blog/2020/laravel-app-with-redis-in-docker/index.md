# Laravel App With Redis In Docker

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Laravel App With Redis In Docker](https://aregsar.com/blog/2020/laravel-app-with-redis-in-docker)

## Creating files and directories

> Note if you have already a docker-compose.yml then these steps are not required

echo '/data' >> .gitignore
mkdir data
touch docker-compose.yml
echo `version: "3.1"' >> docker-compose.yml
echo `services:' >> docker-compose.yml

## Adding the mysql service to docker-compose

```yaml
  redis:
      image: redis:alpine
      container_name: myapp-redis
      command: redis-server --appendonly yes --requirepass myapp
      volumes:
      - ./data/redis:/data
      ports:
        - "8002:6379"
```

## Adding the environment variables for the redis service

```ini

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

## Changing redis configuration settings for the redis service
