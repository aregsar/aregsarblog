# Redis Configure Laravel

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

In this post we will configure Laravel to use a Redis server running in a local docker container.

## Steps to configure Laravel to use Redis

To start, go to the root of your Laravel project then follow steps below:

### Step 1 - Installing the redis driver

Laravel requires the php redis extension, to be able to connect to Redis servers.

So the first thing we need to do is to make sure we have the redis php extension installed.

```bash
pecl install --force redis
```

You will have to install this extension once for each version of PHP that you use.

For full details on using PECL to install PHP extensions see: [Installing PHP Extensions on MacOS](https://aregsar.com/blog/2021/installing-php-extensions-on-macos)

### Step 2 - Setup a Redis sever docker service

Lets first setup a Redis server using a Docker compose service, for local development that our application can connect to:

Create a docker-compose.yml file containing the redis docker service.

```bash
echo docker-compose.yml << EOL
version: "3.1"
services:
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

If a directory named `data` does not exist in the root of the project, run the following:

```bash
# make sure git ignores the data directory
echo '/data' >> .gitignore
mkdir data
#mkdir -p data/mysql
```

### Step 3 - Set the connection settings for the docker Redis service

Open the project `.env` file and set the connection settings for the docker redis service.

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=123456
REDIS_PORT=8002
```

The docker-compose.yml file we setup before will use the settings from the .env file.

### Step 4 - Configure the Redis driver

The redis driver setting declared as the `redis` setting in the root settings array in the `config/database.php` file.

Below I show the before and after configuration settings for the redis driver.

I have annotated the code to describe the settings.

Before:

```php
'redis' => [

        //this specifies the redis client used by Laravel
        //By default REDIS_CLUSTER is not specified in the .env file so falls back to using the 'phpredis' value.
        //the phpredis fallback value specified the php redis extension is used as the client so requires installing the redis php extension
        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            //this setting is only effective when using a managed redis cluster. No impact if redis cluster is not used.
            //falls back to the 'redis' value since by default the REDIS_CLUSTER setting is not specified in the .env file
            //tells the framework to use the default clustering capability of a managed Redis server cluster.
            'cluster' => env('REDIS_CLUSTER', 'redis'),

            //This setting adds a application specific prefix to the cache keys to distinguish between data from multiple apps using the same redis server
            //if you have one redis server per app then remove this setting, no need to differentiate redis keys between apps
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

        # default connection - used by the Laravel frameworks default Redis connection
        'default' => [
            'url' => env('REDIS_URL'),//not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],

        # cache connection - used by the Laravel frameworks default Cache connection
        'cache' => [
            'url' => env('REDIS_URL'),//not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),//uses different database then 'default' connection
        ],
],
```

After:

```php
'redis' => [
        //always uses the php redis extension
        'client' => 'phpredis',

        'options' => [
            //keep setting to be able to use Redis cluster in production if needed
            'cluster' => 'redis',
            //Removed prefix setting since we are using one Redis server per app
        ],

        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //instead use prefix to distinguish between connections
            'database' => `0`,
            //instead add prefix setting to distinguish between connections
            //prefix not required if using one redis server/cluster host per connection
            'prefix' => 'r:'

        ],
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => `0`,
            //instead add prefix setting to distinguish between connections
            //prefix not required if using one redis server/cluster host per connection
            'prefix' => 'c:'
        ],

],
```

The `default` connection is used by default by the Laravel Redis API. and the `cache` connection is used by default (when configured as such) by the Laravel Caching API.

We have configured the `default` and `cache` connections to use unique prefixes instead of separate databases and removed the application specific prefix since we are going to use separate Redis servers per application.

Redis clusters used in production do not support multiple databases so when the connection is used we are automatically namespacing the redis keys with the redis key prefix instead.

> Note: We can add additional connection settings in the `redis` setting array of `config/database.php`, each with their own unique name. In fact for scalability reasons we can add and use redis connections separate from the one used by the Laravel Redis and Cache connections. These other connections can be used by Laravel queues and the Laravel session or used for other application specific purposes.

### Step 5 - Rename the Redis facade alias

The redis PHP extension that we added in step one uses a class named `Redis` that conflicts with the laravel facade alias name.

Therefore we need to change the alias name in the `application.php` file aliases array.

I am changing the name from `Redis` to `ZRedis` below:

From:

```php
aliases' => [
'Redis' => Illuminate\Support\Facades\Redis::class,
]
```

To:

```php
aliases' => [
'ZRedis' => Illuminate\Support\Facades\Redis::class,
]
```

## Test the connection

From within your application or a Tinker session try the following:

```bash
php artisan tinker
```

```php
//use the facade
//test the 'default' connection
Illuminate\Support\Facades\Redis::connection()->ping();
//test the 'default' connection passing it explicitly
Illuminate\Support\Facades\Redis::connection("default")->ping();
//test the 'cache' connection
Illuminate\Support\Facades\Redis::connection("cache")->ping();

//use the facade alias
//test the 'default' connection
\ZRedis::connection()->ping();
//test the 'default' connection passing it explicitly
\ZRedis::connection("default")->ping();
//test the 'cache' connection
\ZRedis::connection("cache")->ping();
```

## Adding a non default connection

The `default` connection is used by the Laravel `Redis` facade and its underlying `RedisManager` and `Redis` classes by default without requiring the user to pass in the connection explicitly.

These is a connection named `cache` that is not the default connection that we could explicitly use. However since that connection is used as the default connection of the Laravel Cache (when configured as such), we will create a new connection and use that instead:

In the config/database.php inside the `redis` driver section, add a new connection named `redis2`:

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

This connection usea the same redis host and port as the other connections, but we will give it its own prefix `r2`.

If we wanted to scale our connections we could have added and assigned a different REDIS_HOST_2 setting to connect to an entirely separate Redis server.

Now we can explicitly use the `redis2` connection:

```php
Illuminate\Support\Facades\Redis::connection('redis2')->ping();

\ZRedis::connection('redis2')->ping();
```

Note that we are explicitly passing in the connection name to the connection method to override the default connection.
