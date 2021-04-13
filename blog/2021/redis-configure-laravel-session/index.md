# Redis Configure Laravel Session

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

In this post we will configure Laravel to use a Redis server running in a local docker container to store and retrieve session data.

Out of the box the Laravel session is configured to use files.

> Note that the session id itself will still be stored in a session cookie. However you could configure the session to use cookies for session data as well by changing the default session from file to cookie.

## Steps to configure Laravel to use Redis for Session storage

To start, go to the root of your Laravel project then follow steps below:

### Step 1 - Install the Laravel Redis driver

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 2 - Setup the Redis sever docker service

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 3 - Set the connection settings for the docker Redis service

Refer to step 2 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 4 - Configure the Redis driver settings

Refer to step 4 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 5 - adding a redis connection for the session

Based on the last step we know that there are two existing connections out of the box in the `config/database.php` file.

One is the `default` connection used by the Laravel Redis API by default and the other is the `cache` connection used by the Laravel Cache API by default (since the `default` cache store is configured to use the `redis` store that uses this `cache` connection).

While we can configure the session to use one of these two existing redis driver connections from the `config/database.php` file, it is better to create a separate connection just for the session.

This way if we need to scale out our application we can configure this connection to use its own separate Redis server.

Below I am showing the `config/database.php` file snippet showing the connections within the redis driver connections array before and after adding the connection for use by the session.

Note that the before settings contain the configuration changes that were made in step 4.

Before:

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
        //we added this connection specifically for use as the redis session connection
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

Note that I have added a third connection named `session` that we will use for the session driver connection.

### Step 6 - Update the SESSION_DRIVER setting in the .env file

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

### Step 7 - Add a SESSION_CONNECTION setting to the .env file

Add a `SESSION_CONNECTION` setting to select the `session` connection we added in the previous section:

```ini
#choose the 'session' connection inside the 'redis' driver array in config/database.php
SESSION_CONNECTION=session
```

### Step 8 - Update the default settings of the session driver and connection

Change the default parameter of the `driver` and `connection` env() helper functions to `redis` driver and `session` connection respectively.

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

With these defaults, even if the `SESSION_DRIVER` and `SESSION_CONNECTION` are missing from the .env file the redis session connection will be selected.

### Step 9 - Setting up a separate session store

As currently configured the Laravel session infrastructure will by default uses a redis store named `redis` (hard coded by the Laravel framework) from the `stores` array of the `config/cache.php` file. This `redis` store is typically configured to be used as a redis cache store, hence it is typically configured to use the `cache` connection from `config/database.php`.

It will then override the `connection` setting within that store with the `connection` setting in the `config/session.php` file.

The effect of all this is that the session and the cache end up sharing the same `redis` store but with the session overriding the connection of this store.

This behavior is hard coded in the framework SessionHandler and is opaque and confusing to a user.

However we can change the default behavior of the framework by explicitly selecting a different redis store to use by using the `store` configuration setting in the `config/session.php` file. This separate store can be configured to use the spearate session connection that we added to `config/database.php` for use by the Laravel session.

So let us create a separate redis store named `session` in the `stores` array of the `config/cache.php` file.

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
        //added session redis store used by the framework SessionManager. The connection of this store will be overridden by the SessionManager with the connection specified in the config/session.php file
        'session' => [
            'driver' => 'redis', //the cache driver that refers to the redis driver in config/database.php
            'connection' => 'session',//the  redis connection setting as specified in the config/database.php file
            'lock_connection' => 'default',
        ],
    ]
```

Note that we set the connection of the `session` cache store to the `session` connection that we added to `database.php`. However this is strictly unnecessary since it will be overridden by the `connection` setting in `session.php` file which also has the value of `session`.

### Step 10 - Configuring the session to explicitly use the session cache store

In this steop we will use the `session` cache store that we added in the previous step.

Open the .env file and add the `SESSION_STORE` setting

```ini
SESSION_STORE=session
```

Also open the `config/session.php` file and update the default parameter of the `store` settings env() helper function.

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

Now we are configuring the framework to use the `session` cache store instead of the (hard coded in the framework) `redis` store.

The framework will now override the `session` cache store connection setting with the `session` connection that we configured in the `config/session.php` file in steps 7 and 8.

### Step 11 use the session

Start a Laravel project use the session methods during a web request.

```php
Session::put('key', 'value');
$value = Session::get('key');
$haskey = Session::has('key')
Session::forget('key');

var_dump(session());

//regenerate session id
session()->regenerate();
```

## The session lifetime

The session lifetime is configured by the SESSION_LIFETIME setting in the .env file.

The out of the box configuration is set to 120 minutes after which the session expires

```ini
SESSION_LIFETIME=120
```

## The session cookie settings

Even though we have not configures the application to use cookie sessions for storing our session data, The session itself uses cookies to persist the session id across requests.

The following `config/session.php` file settings are related to the session cookie.

```php

         //the name of the session cookie
        'cookie' => env('SESSION_COOKIE',Str::slug(env('APP_NAME', 'laravel'), '_').'_session',
        //session does not expire when browser tab is closed
        'expire_on_close' => false,
        //cookie domain is not specified
         'domain' => env('SESSION_DOMAIN', null),
         //cookie path starts from root
        'path' => '/',
        //send cookies over HTTPS
         'secure' => env('SESSION_SECURE_COOKIE'),
         //sedn cookies over HTTP protocol only
         'http_only' => true,
         //security policy
         'same_site' => 'lax',

```

To make sure we always use secure cookies in production change from

```php
 'secure' => env('SESSION_SECURE_COOKIE'),
```

To

```php
 'secure' => env('SESSION_SECURE_COOKIE', true),
```

And Add the following in .env

change

```ini
SESSION_SECURE_COOKIE=false
```

This way even if we forget to include SESSION_SECURE_COOKIE in production .env file we will be protected.

Also if we serve multiple domains and want to limit the session cookie only to one of them we can add SESSION_DOMAIN to our production .env file and set its value to the domain name.

Finally we can add SESSION_COOKIE to the .env file if we want to set a custom cookie name.

## Takeaway

Together the driver, connection and cache store specify all that is needed to use a redis server as session store.
Setting the driver to `redis` specifies the `redis` driver specified in the config/database.php file.
Setting the connection `session` specifies the `session` connection within the `redis` driver in the config/database.php file.

## Appendix

This appendix shows the Laravel framework version 8 code that hard codes the `redis` store from the config/cache.php nd overrides the selected store connection using the `connection` setting value in config/session.php

[SessionManager.php](https://github.com/laravel/framework/blob/8.x/src/Illuminate/Session/SessionManager.php)

```php

class SessionManager extends Manager
{
    protected function createRedisDriver()
    {
        //passes in the hard coded parameter `redis` as the store
        $handler = $this->createCacheHandler('redis');

        //gets the cache store from the handler
        //sets the connecton of the store to the `session.connection` configuration
        //thereby overriding the origially configured connection of the store
        $handler->getCache()->getStore()->setConnection(
            $this->config->get('session.connection')
        );

        return $this->buildSession($handler);
    }

    protected function createCacheHandler($driver)
    {
        //The $driver is actually a store. So it is a misnamed.

        // unless we explicily define the
        //`session.store` (which is the store setting in the config/session.php file)
        //$store is set to  the hard coded 'redis' parameter passed in via $driver
        $store = $this->config->get('session.store') ?: $driver;

        return new CacheBasedSessionHandler(
            //gets the cache from the container
            //assuming we have not configured the session.store,
            //sets the cache store to the 'redis' store
            //clones the cache and sets the clone into the handler it is returning
            clone $this->container->make('cache')->store($store),
            $this->config->get('session.lifetime')
        );
    }
}
```
