# Configure Laravel Session To Use Redis

April 7, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Configure Laravel Session To Use Redis](https://aregsar.com/blog/2020/configure-laravel-session-to-use-redis)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed newer versions of Laravel 

In a previous article [Configure Laravel To Use Php Redis](https://aregsar.com/blog/2020/configure-laravel-to-use-php-redis) we learned how to configure laravel to use Redis.

In this article I will show you how to convert the Laravel Session configuration to use Redis.
By default Laravel uses the file driver to store session data on the server. This will not be performant for high traffic sites.

## Setting the Laravel session driver

We can change the default Laravel session store configuration to use the Redis key value store for its session by changing the 'driver' setting in the `config/session.php` configuration file.

Here is the default setting in config/session.php:

```php
    'driver' => env('SESSION_DRIVER', 'file'),
```

First you can change the default setting parameter from `file` to `redis`.

```php
    'driver' => env('SESSION_DRIVER', 'redis'),
```

This changes the default value to use the `'redis'` driver defined in `config/database.php`.

We also need need to change the 'driver' setting the `SESSION_DRIVER=file` in the .env file to `SESSION_DRIVER=redis`. This sets the SESSION_DRIVER environment variable setting to the `'redis'` driver defined in `config/database.php`.

Since we set the default parameter to `redis` we can even remove the `SESSION_DRIVER=redis` setting from .env file altogether and the configuration will still default to using Redis.

If we know we are always going to use Redis for the session store, we can just hard code the driver value to `redis` in the `config/session.php` file:

```php
    'driver' => 'redis',
```

## The Redis connection used by Laravel Session

The `config/session.php` file also has a `'connection'` setting that selects which Redis driver `connection` the session should use, when the session is set to use the `redis` driver.

```php
    #uses the 'cache' configuration of the 'redis' driver in config/database.php
    # add SESSION_CONNECTION=default or SESSION_CONNECTION=cache to .env
    # don't know if null parameter defaults to 'default' connection if SESSION_CONNECTION is not specified
    'connection' => env('SESSION_CONNECTION', null),
```

The connection value comes from one of the Redis driver connection names defined in `config/database.php`.

By default the `SESSION_CONNECTION` environment variable setting is not defined in `.env` file. Also the default parameter value for the `'connection'` setting in `config/session.php` is null. So by default `'connection'` is set to null causing the `'default'` connection of the `'redis'` driver in `config/database.php` to be selected as the session connection by the Laravel session provider\facade.

## Explicitly configuring the session redis connection

Instead of allowing the session connection to use the default redis connection, I like to be more explicit.

The following steps set the session connection to a completely separate redis connection.

First add a new connection to the available redis connections inside the `redis` driver section in `config/database.php`:

```bash
'redis' => [

    'session' => [
                'url' => env('REDIS_URL'),
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'password' => env('REDIS_PASSWORD', null),
                'port' => env('REDIS_PORT', '6379'),
                'database' => env('REDIS_CACHE_DB', '3'),
    ],
],
```

The new connection is named `session` and its `database` setting is set to `3` to use a different database then the default connection since they both use the same redis server settings.

Next remove default the `SESSION_CONNECTION=file` setting from the `.env` file because it will not be used.

Finally in `config/session.php` I change the default connection parameter from `null` to `'session'` :

```php
    'connection' => env('SESSION_CONNECTION', 'session' ),
```

This sets the default value to the new `session` connection added to `config/database.php`.

Alternatively in `config/session.php` you can hard code the session driver `connection` to `session`:

```php
    'connection' => 'session',
```

This sets the value to directly to the new `session` connection added to `config/database.php`.

> Note that I did not use the 'default' connection. This is because I like to separate my session Redis connection used by the Laravel Session from other Redis connections uses by the Laravel Redis facade to store application data in the cache or the connections used by the Larave Cache and Queue facades to cache data or queue jobs.
