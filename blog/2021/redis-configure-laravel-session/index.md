# Redis Configure Laravel Session

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

#### Step 1 - adding a redis connection for the session to database.php

There are two existing connections out of the box in the database.php file.

One is named `default` and is used by default by the Laravel Redis methods.

The other is named `cache` which is used by default by the Laravel Cache methods (when the default cache store is configured to use the `redis` store that uses this `cache` connection).

While we can configure the session to use one of these two existing redis connections in the database.php file, it is better to create a separate connection just for the session.

This way if we need to scale out our application we can configure this connection to use it own separate Redis server.

Below I am showing the database.php file snippet showing the connections within the redis driver connections array before and after adding the redis connection for use by the session.

Before:

```php
'redis' => [

        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],

        'cache' => [
            'url' => env('REDIS_URL'),//This setting is not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],

    ],
```

After:

```php
'redis' => [
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'd:',
        ],

        'cache' => [
            'url' => env('REDIS_URL'),//This setting is not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c:',
        ],
         'session' => [
            'url' => env('REDIS_URL'),//This setting is not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 's:',
        ],

    ],
```

Note that I have added a third connection named 'session' and that I have changed all three connections to use different `prefix` setting instead of different `database` settings.

This is because I want to make these settings compatible with working with Redis cluster servers in production. Redis clusters only support a single database so prefix is used instead to separate the redis, cache and session keys. The prefix setting value will automatically get prepended to the redis keys used by the Laravel application.

#### Step 2 - Add a SESSION_CONNECTION setting to the .env file

Add a SESSION_CONNECTION to select the 'session' connection we added in the previous section

```ini
#choose the 'session' connection inside the 'redis' driver array in config/database.php
SESSION_CONNECTION=session
```

#### Step 3 - Update the SESSION_DRIVER setting in the .env file

Change the setting in the .env file to select the redis driver

Before:

```ini
SESSION_DRIVER=file
```

After:

```ini
#choose the 'redis' driver section in config/database.php
SESSION_DRIVER=redis
```

#### Step 4 - Update the default settings of the session driver and connection

Change the default parameter of the env() helper to the redis driver and session connection.
Before:

```php
    'driver' => env('SESSION_DRIVER', 'file'),

    'connection' => env('SESSION_CONNECTION', null),
```

After:

```php
    'driver' => env('SESSION_DRIVER', 'redis'),

    'connection' => env('SESSION_CONNECTION', 'session'),
```

Now if the SESSION_DRIVER and SESSION_CONNECTION are missing from the .env file the redis session connection will be selected.

#### Step 5 - Setting up a separate session store

As currently configured the Laravel session infrastructure will by default use a redis store named `redis` from the `stores` array of the cache.php file.

It will then override the `connection` setting within that store with the `connection` setting in the session.php file. This behavior is hard coded in the framework SessionHandler and is opaque and confusing to a user.

Also this redis store is typically configured to be used as a redis cache store so using it for the session seems a little hacky.

> Note: I am not sure why the framework needs to override the session store connection since the store itself specifies a connection. My guess is it is because the framework by default is using the `redis` cache store which might be getting shared for caching functionality as well that might need to use a different connection.

We can change the default behavior of the framework by explicitly selecting the redis store to use, using the `store` setting in the session.php file.

We can choose to explitly select the `redis` cache store that is already selected by default by the framework.

Or we can choose a separate cache store that we can add to the cache.php file.

I like to create a separate redis store named `session` in the `stores` array of the cache.php file. This way even if someone removes the original `redis` store because it is unused, then the redis session will continue will work.

Open the cache.php file and add the `session` store:

Before:

```php

    'stores' => [
        //out of the box redis store
        'redis' => [
                'driver' => 'redis',
                'connection' => 'cache',
                'lock_connection' => 'default',
            ],
    ]
```

After:

```php
    'stores' => [
        //out of the box redis store
        'redis' => [
                'driver' => 'redis',
                'connection' => 'cache',
                'lock_connection' => 'default',
            ],
        //session redis store used by the framework SessionManager. The connection of this store will be overridden by the SessionManager with the connection specified in the config/session.php file
        'session' => [
            'driver' => 'redis', //the cache driver that refers to the redis driver in config/database.php
            'connection' => 'session',//the  redis connection setting as specified in the config/database.php file
            'lock_connection' => 'default',
        ],
    ]
```

I will use the `session` cache store that we added above in the following step.

#### Step 6 - Configuring the session to explicitly use the session cache store

Open the .env file and add the `SESSION_STORE` setting

```ini
SESSION_STORE=session
```

Also open the session.php and update the default parameter of the env() helper

Before:

```php
        //selects a session store configured in the 'stores' array in config/cache.php file
    'store' => env('SESSION_STORE', null),
```

After:

```php
        //selects a session store configured in the 'stores' array in config/cache.php file
    'store' => env('SESSION_STORE', 'session'),
```

Now we are telling the framework to ovrride the `session` cache store with the session connection setting in the session.php file.

## The session lifetime

The session lifetime is configured by the SESSION_LIFETIME setting in the .env file.

The out of the box configuration is set to 120 minutes after which the session expires

```ini
SESSION_LIFETIME=120
```

## The session cookie settings

Even though we have not configures the application to use cookie sessions for storing our session data, The session itself uses cookies to persist the session across requests.

The following .env file settings are related to the session cookie.

```ini



```

## Takeaway

Together the driver, connection and cache store specify all that is needed to use a redis server as session store.
Setting the driver to `redis` specifies the `redis` driver specified in the config/database.php file.
Setting the connection `session` specifies the `session` connection within the `redis` driver in the config/database.php file.
