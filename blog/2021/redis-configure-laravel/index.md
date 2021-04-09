# Redis Configure Laravel

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

In this post I am going to show haow we configure redis for use in Laravel applications.

## Setup a Redis sever docker service

Lets first setup a Redis server using a Docker compose service, for local development that our application can connect to:

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

If you already have a `docker-compose.yml` file in the project root then just add the redis service portion of the above yml code under the services section.

We also need to create a directory in the Laravel project root to persist the redis data.

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

## Install the redis driver

pecl install --force redis

## Configure the Redis driver

Before:

```php
'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

        # default connection
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],

        # cache connection
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),#uses different database then 'default' connection
        ],
],
```

After:

```php
'redis' => [
        'client' => 'phpredis',

        'options' => [
        //only impacts clustered redis
        //tells the framework to use the default clustering capability of a managed Redis server cluster.
            'cluster' => env('REDIS_CLUSTER', 'redis'),

            //if you have one redis server per app then comment out, no need to differentiate redis keys between apps
            //'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

        'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', '6379'),
        'database' => `0`,
        'prefix' => 'r:'
        ],
        'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', '6379'),
        'database' => `0`,
        'prefix' => 'c:'
        ],

],
```

## Rename the Redis facade alias

From
aliases' => [
'Redis' => Illuminate\Support\Facades\Redis::class,
]

To
aliases' => [
'ZRedis' => Illuminate\Support\Facades\Redis::class,
]

## Test the connection

Here also the default connection is used:

Uses the `default` redis driver connection from config/database.php is used:
Illuminate\Support\Facades\Redis::connection()->ping();

\ZRedis::connection()->ping();

## Adding a non default connection

These is a connection named `cache` that is not a default connection that we could explicitly used. But since that connection is used as the default connection by the Laravel Cache, lets create a new connection and use it instead:

in the config/database.php inside the `redis` driver section, add a new connection named `redis2`:

```php
'redis2' => [
'url' => env('REDIS_URL'),
'host' => env('REDIS_HOST', '127.0.0.1'),
'password' => env('REDIS_PASSWORD', null),
'port' => env('REDIS_PORT', '6379'),
'database' => `0`,
'prefix' => 'r2:'
],
```

It will use the same redis host and port as the other connections,but we will give it its own prefix `r2`:

Now we can explicitly use the `redis2` connection from config/database.php:

Illuminate\Support\Facades\Redis::connection('redis2')->ping();

\ZRedis::connection('redis2')->ping();

## Redis cluser setttings

In order to use a managed cluster you only need to add the line 'cluster' => env('REDIS_CLUSTER', 'redis') to the ‘options’ array in the 'redis' driver in config/database.php shown below:

from
'options' => [
'cluster' => env('REDIS_CLUSTER', 'redis'),
],

to
'options' => [
'cluster' => 'redis',
],
