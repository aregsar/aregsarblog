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

We will use the `default` connection for all in app access to Redis that directly use the Laravel `Redis` provider. 

> Note: Unlike in the session,cache and queue configuration files, there is no special `default` setting in the config/database.php file that is set to the default redis connection. Instead by
default when we use the Laravel `Redis` facade, helper or provider without specifying the connection to use, the connection that is labeled `default` will be used.

We will use the `cache` connection for all in app access to Redis that uses the Laravel `Cache` provider.

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
            'database' => env('REDIS_QUEUE_DB', '2'),
        ],
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_SESSION_DB', '3'),
        ],
    ],
```

> Note: the `database` setting for the `queue` and `session` settings is also updated to use `REDIS_QUEUE_DB` and `REDIS_SESSION_DB` environment settings accordingly

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

The `cluster` setting, indicates that our Redis server connection is connecting to a self managed Redis cluster.

> Note: A self managed cluster connection connects to a proxy cluster manager that that manages a cluster of redis instances, without requiring the redis client to manages cluster nodes

If we always use a managed cluster we could change the ` 'cluster' => env('REDIS_CLUSTER', 'redis')` cluster setting to the hard coded `'cluster' => 'redis'` setting

## The Redis Client

The default out of the box Laravel configuration is configured to use the Php Redis extension.

`'client' => env('REDIS_CLIENT', 'phpredis')`

For that to work we need to make sure the Php Redis extension is installed.

The instructions for doing so are here: `put link here`

Also unless we rename the facade alias we need to add a 
`use Illuminate\Support\Facades\Redis;` statement in the file we use the `Redis` facade in.

## Configuring the Session Configuration file to use redis

Change the `driver` configuration in `config/session.php` to:

```php
 'driver' => env('SESSION_DRIVER', 'redis'),
```

Change the `connection` configuration in `config/session.php` to:

```php
'connection' => env('SESSION_CONNECTION', 'session'),
```

## Configuring the Cache Configuration file to use redis

Change the `default` store configuration in `config/cache.php` to:

```php
'default' => env('CACHE_DRIVER', 'redis'),
```

Change the `prefix` configuration in `config/cache.php` to:

```php
  'prefix' => env('CACHE_PREFIX', Str::slug(env('APP_NAME', 'MyApplication'), '_').'_cache'),
```

We can leave the redis store connection value in `config/cache.php` as is, since it is set to `cache` out of the box which references the out of the box `cache` redis connection in `config/database.php`

```php
    'stores' => [
        'redis' => [
            'driver' => 'redis',
            //uses the 'cache' store
            'connection' => 'cache',
        ],
    ],
```

## Configuring the Queue Configuration file to use redis

Change the `default` configuration in `config/queue.php` to:

```php
  'default' => env('QUEUE_CONNECTION', 'redis'),
```

Change the `connections` configuration in `config/queue.php` to:

```php
      'connections' => [
        'redis' => [
            'driver' => 'redis',
            //uses queue connection of redis driver in config/database.php
            'connection' => 'queue',
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],
    ],
```

## Setting up the environment variables for our configuration file to use

The following environment variable need to be set to `redis`

```ini
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
```

Additionally we need to set the host and password settings. The values set below are for a local Redis instance running:

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

And these are Digitalocean Redis cluster settings:

```ini
REDIS_HOST=tls://<your_redis_host>.db.ondigitalocean.com
REDIS_PASSWORD=<your_redis_password>
REDIS_PORT=25061
```

## The final redis configuration

The final redis configuration settings for all redis related configuration files is shown below:

`config/database.php`:

```php
  'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'myapp'), '_').'_database_'),
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
            'database' => env('REDIS_QUEUE_DB', '2'),
        ],
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_SESSION_DB', '3'),
        ],
    ],
```

`config/cache.php`:

```php
  'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
        ],

    'default' => env('CACHE_DRIVER', 'redis'),

    'prefix' => env('CACHE_PREFIX', Str::slug(env('APP_NAME', 'myapp'), '_').'_cache'),
```

`config/session.php`:

```php
    'driver' => env('SESSION_DRIVER', 'redis'),

    'connection' => env('SESSION_CONNECTION', 'session'),
```

`config/queue.php`:

```php
  'default' => env('QUEUE_CONNECTION', 'redis'),

  'connections' => [
        'redis' => [
            'driver' => 'redis',
            'connection' => 'queue',//updated value
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],
    ],
```

## Testing the redis connections

The following code can be run using Laravel tinker:

An now we can test the connections add the following route to a new controller that you can add named `RedisController`:

```php
Route::get('/redis', 'RedisController@redisTest');
```

Then add the following `redisTest` method to the controller:

```php
use Illuminate\Support\Facades\Redis;

public function redisTest()
{
    $redis = Redis::connection();

    try{
        var_dump($redis->ping());

        Redis::incr('visits');
        Redis::set('name', 'Taylor');

    } catch (Exception $e){
        $e->getMessage();
    }
}
```
