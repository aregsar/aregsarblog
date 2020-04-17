# Create Laravel Project With Multiple Redis Stores

April 18, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Create Laravel Project With Multiple Redis Stores](https://aregsar.com/blog/2020/create-laravel-project-with-multiple-redis-stores)

## Configuring individual redis database connections in config/database.php

In this section I will configure a separate database for each of the Redis, Session, Cache and Queue providers used in a Laravel application.

The `config/database.php` file contains a section with Redis server connections
with two out of the box connections named `default` and `cache`:

```bash
 'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],
    ],
```

Note that the `default` connection has a `database` default of `0` and the `cache` connection has a `database` default of `1` to assign separate databases to those connections that are configured to use the same Redis server.

We will use the `default` connection for all in app access to Redis that directly use the Laravel `Redis` provider. We will use the `cache` connection for all in app access to Redis that uses the Laravel `Cache` provider.

We need to add two more connections named `queue` and `session` for the Laravel `Session` and `Queue` providers to use.

We will choose database numbers that are unique for each of the redis connections. 

> Note: A Redis server allows up to 16 unique databases

The resulting final set of connections is shown below:

```bash
 'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '2'),
        ],
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '3'),
        ],
    ],
```

## Setting the application name prefix key

The `prefix` option sets the redis database prefix key for all keys of the application:

```bash
 'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],
```

Change the `laravel` default to your app name.

```bash
 'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'MyApplication`), '_').'_database_'),
        ],
```

## The redis cluster setting

The `cluster` setting , indicates that our Redis server connection is connecting to a self managed Redis cluster.

> Note: A self managed cluster connection connects to a proxy cluster manager that that manages a cluster of redis instances, without requiring the redis client to manages cluster nodes

If we always use a managed cluster wr can change the ` 'cluster' => env('REDIS_CLUSTER', 'redis')` cluster setting to the hard coded `'cluster' => 'redis'` setting


## The Redis Driver

The default out of the box Laravel configuration is configured to use the Php Redis extension.

`'client' => env('REDIS_CLIENT', 'phpredis')`

For that to work we need to make sure the Php Redis extension is installed.

The instructions for doing so are here: `put link here`

## Setting up the environment variables for our configuration file to use






