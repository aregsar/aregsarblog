# Redis Configure Laravel Session

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Session cache store

```ini
SESSION_LIFETIME=120
SESSION_DRIVER=file
```

```php
    'driver' => env('SESSION_DRIVER', 'file'),

    'connection' => env('SESSION_CONNECTION', null),
```

## change the session driver and session connection to redis

```ini
SESSION_LIFETIME=120
#choose the 'redis' driver section in config/database.php
SESSION_DRIVER=redis
#choose the connection inside the redis driver section in config/database.php
SESSION_CONNECTION=session
```

```php
    'driver' => env('SESSION_DRIVER', null),

    'connection' => env('SESSION_CONNECTION', null),
```

Together the driver and connection specify all that is needed to use a redis server as cache store.
Setting the driver to `redis` specifies the `redis` driver specified in the config/database.php file.
Setting the connection `session` specifies the `session` connection within the `redis` driver in the config/database.php file.

# adding a redis connection for the session to database.php

TODO move this section before the section on setting the session config

## A mistake in comments for session store in config/session.php

There is a third setting `'store' => env('SESSION_STORE', null)` in the session.php file that does not apply to configuring redis session driver. The notes for this configuration specify it applying to redis but it is a mistake. The older versions of the session.php file did not specify this.

Since the session.php file already has explicit settings for the redis driver and redis connection to use, specifying a store setting would be redundant. This is because a (cache) store as specified in the cache.php file (see store description - put link here) just encapsulates a driver setting and a connection setting which in this case is already explicitly been specified in the session.php file.

The store setting would be selecting a session store configured in the 'stores' array in config/cache.php file.

```php

    /*
    |--------------------------------------------------------------------------
    | Session Cache Store
    |--------------------------------------------------------------------------
    |
    | While using one of the framework's cache driven session backends (e.g SESSION_DRIVER=memchached) you may
    | list a cache store that should be used for these sessions. This value
    | must match with one of the application's configured cache "stores".
    |
    | Affects: "apc", "dynamodb", "memcached", "redis"
    |
    */

    //selects a session store configured in the 'stores' array in config/cache.php file
    'store' => env('SESSION_STORE', null),
```
