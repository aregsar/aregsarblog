# Redis Configure Laravel

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

In this post I am going to show haow we configure redis for use in Laravel applications.

## Setup

#### Step 7 - Setup the Redis sever docker service

Create a docker-compose.yml file containing the redis docker service.

```bash
echo docker-compose.yml << EOL
version: "3.1"
  redis:
    image: redis:alpine
    container_name: redis
    command: redis-server --appendonly yes --requirepass "${REDIS_PASSWORD}"
    volumes:
      - ./data/redis:/data
    ports:
      - "${REDIS_PORT}:6379"
EOL
```

If you already have a docker-compose.yml file in the project root then just add the redis service portion of the above yml code under the services section.

Also create a directory in the Laravel project root to persist the redis data.

```bash
mkdir -p data/redis
```

#### Step 8 - Set the connection settings for the docker Redis service

Open the project `.env` file and set the connection settings for the docker redis service.

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=123456
REDIS_PORT=8002
```

The docker-compose.yml file we setup before will use the settings from the .env file.
