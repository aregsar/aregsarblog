# Configure Redis Session Store For Laravel

April 7, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Configure Redis Session Store For Laravel](https://aregsar.com/blog/2020/configure-redis-session-store-for-laravel)

In a previous article we learned how to configure laravel to use Redis.

In this article I will show you how to convert the Laravel Session configuration to use Redis.
By default Laravel uses the file driver to store session data on the server. This will not be performent in high trafic sites. 

## Setting the Laravel session driver

We can change the default Laravel configuration to use the Redis key value store for its session by changing the 'driver' setting in the `config/session.php` configuration file 

here is the setting in config/session.php:

```php
    # set SESSION_DRIVER=redis in .env
    # uses the 'redis' driver defined in config/database.php
    'driver' => env('SESSION_DRIVER', 'file'),
```

We can change the 'driver' setting the SESSION_DRIVER env var value in .env file to 'redis'.

change:

```ini
SESSION_DRIVER=file
```

To:

```ini
SESSION_DRIVER=redis
```

Another way to change the setting is by removing the SESSION_DRIVER setting from .env file and changing the 'file' default value passed into the second parameter in `config/session.php` to 'redis'.

```php
    'driver' => env('SESSION_DRIVER', 'redis'),
```

Yet Another way would be to just hard code the value:

```php
    'driver' => 'redis',
```

To be consistent I like to set the `SESSION_DRIVER=redis` in the .env file and also set the `'driver' => env('SESSION_DRIVER', 'redis')` in the `config/session.php`

## The Redis connection used by Laravel Session

The config/session.php file also has a setting for which Redis connection the session Redis driver should use.

```php
    #uses the 'cache' configuration of the 'redis' driver in config/database.php
    # add SESSION_CONNECTION=default or SESSION_CONNECTION=cache to .env
    # don't know if null parameter defaults to 'default' connection if SESSION_CONNECTION is not specified
    'connection' => env('SESSION_CONNECTION', null),
```

The connection value comes from one of the Redis driver connnection names defined in `config/database.php`
connections.

By default if SESSION_CONNECTION setting is not defined in .env file and the default parameter value is null which should select the 'default' connection of the 'redis' driver in `config/database.php`.

I like to be more explicit so I will add the setting to the .env file

```ini
SESSION_CONNECTION=cache
```

Also I change the default parameter to:

```php
    'connection' => env('SESSION_CONNECTION', 'cache'),
```

> Note that I did not use the 'default' connection. This is because I like to separate my session Redis connection used by the Laravel Session from other Redis connections uses by the Laravel Redis facade to store application data in the cache or the connections used by the Larave Cache and Queue facades to cache data or queue jobs.

For reference I have included the redis driver configuration from `config/database.php` here:

```php
'redis' => [

        #driver uses phpredis client as default
        # no need to set REDIS_CLIENT=phpredis in .env to specify phpredis explicitly
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

        #cache connection
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),#uses different database then 'default' connection
        ],

    ],
```

For details on how to configure the Laravel redis driver see:

[Laravel7 Redis Configuration](https://aregsar.com/blog/2020/laravel7-redis-configuration)
