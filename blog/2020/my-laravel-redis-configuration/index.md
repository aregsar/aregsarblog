# My Laravel Redis Configuration

April 28, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[My Laravel Redis Configuration](https://aregsar.com/blog/2020/my-laravel-redis-configuration)

## config/database.php configuration file

```bash
 'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'options' => [
            'cluster' => 'redis',
        ],
        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            `prefix` => `d:`
        ],
        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            `prefix` => `c:`
        ],
        'queue' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            `prefix` => env('REDIS_QUEUE_PREFIX', 'q0:'),
        ],
        'session' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            #since config/session.php does not have an app level prefix we can change
            #this prefx to `myapp:s:` if we share redis server between apps
            `prefix` => `s:`
        ],
    ],
```

> Note: REDIS_QUEUE_PREFIX env setting is not hardcoded and only specified in production to allow production deployment to switch queue for new deployments with serialized model changes
> Note: `prefix` setting is used instead of using different database number to be able to segment the keys when used with a managed redis cluster which does not support multiple databases. Also `prefix` prefixes keys for the app that uses this configuration. Cache and queue clients that use these redis connections may also specify an application level prefix to segment cache keys between different applications using the same redis server.

## Local environment variable settings for config/database.php configuration file

```ini
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=null
```

## Digitalocean Managed Redis Cluster environment variable settings

```ini
REDIS_HOST=tls://<your_redis_host>.db.ondigitalocean.com
REDIS_PORT=25061
REDIS_PASSWORD=<your_redis_password>
```

> Note: the `tls:` scheme and TLS protocol port number is used since managed Redis only supports TLS connections

## Redis enabled config/session.php configuration file

```php
    'driver' => 'redis',
    'connection' => 'session',
```

## Redis enabled config/cache.php configuration file

```php
   #select the `redis` cache connection in `connections` setting below
  'default' => `redis`,

  'stores' => [
    'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
        ],
  ],

  # redis cache key prefix (used to segment between multiple apps)
  # remove this key if we never share a redis server between apps
  'prefix' => Str::slug(env('APP_NAME', 'myapp'), '_').'_cache',
```

## Redis enabled config/queue.php configuration file

```php
  #select the `redis` queue connection in `connections` setting below
  'default' => 'redis',

  'connections' => [
        'redis' => [
            'driver' => 'redis',
            'connection' => 'queue',
            'retry_after' => 90,
            'block_for' => null,
            # redis queue key prefix (used to segment between multiple apps)
            # remove this key if we never share a redis server between apps
            # key must be wrapped in brackets for redis cluster(Note: TBD This may only apply when using client controlled clustering)
            'queue' => '{myapp}',
        ],
    ],
```

## Redis Cluster Queue key hash tags

> Note: TBD This may only apply when using client controlled clustering

From Laravel docs at `https://laravel.com/docs/7.x/queues`

If your Redis queue connection uses a Redis Cluster, your queue names must contain a key hash tag. This is required in order to ensure all of the Redis keys for a given queue are placed into the same hash slot:

'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
],
