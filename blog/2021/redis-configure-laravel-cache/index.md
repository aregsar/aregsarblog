# Redis Configure Laravel Cache

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Steps to setup a redis cache store for Laravel

#### Step 1

Open the `.env` file and rename the `CACHE_DRIVER` setting to `CACHE_STORE` to accurately reflect that it is selecting one of the available cache stores from the `stores` array in `config/cache.php`.

Before:

```ini
#incorrectly named environment variable that selects the cache store
CACHE_DRIVER=file
```

After:

```ini
#correctly named environment variable that selects the cache store
CACHE_STORE=file
```

#### Step 2

Open the `config/cache.php` file and rename the `CACHE_DRIVER` variable to `CACHE_STORE`.

I am showing the relevant snippet from `config/cache.php` file below:

Before:

```php
    //Default Cache Store used by laravel caching funcions
    //Misnamed env var that selects the cache store (not cache driver)from the Cache Stores array below
    'default' => env('CACHE_DRIVER', 'file'),
    'stores' => [
        //this is the 'file' cache store selected by the 'default' setting above when CACHE_DRIVER=file in .env file or by the second argument when no key is specified
        'file' => [
                'driver' => 'file', //this is the actual cache driver which is not what is being selected
                'path' => storage_path('framework/cache/data'),
            ],
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

After:

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the file cache store from the cache 'stores' array below using correctly named CACHE_STORE env setting
    'default' => env('CACHE_STORE', 'file'),
    'stores' => [
        //this is the 'file' cache store selected by the 'default' setting above when CACHE_STORE=file in .env file or by the second argument when no key is specified
        'file' => [
                'driver' => 'file', //the actual cache driver for the 'file' store
                'path' => storage_path('framework/cache/data'),
            ],
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

### Step 3

Update the `CACHE_STORE` setting in the .env file to `redis`.

Before:

```ini
#correctly named environment variable that selects the cache store
CACHE_STORE=file
```

After:

```ini
#correctly named environment variable that selects the cache store
CACHE_STORE=redis
```

The setting value is used by the `default` setting in `config/cache.php` file to select the `redis` store.

The `default` setting is used by the Laravel cache helper, facade and cache manager by default.

### Step 4

For the `default` setting in the `config/cache.php` file, update the default argument of the env() helper to `redis`.

Before:

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the file cache store from the cache 'stores' array below using correctly named CACHE_STORE env setting
    'default' => env('CACHE_STORE', 'file'),
```

After:

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the cache redis store from the Cache Stores array below using correctly named CACHE_STORE setting
    'default' => env('CACHE_STORE', 'redis'),
```

Now that we updated this default argument to `redis` the `default` setting will select the `redis` store even if the `CACHE_STORE` setting is not provided in the .env file.

Within the `redis` store setting there is a driver and connection settings.

This is shown below:

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the cache redis store from the Cache Stores array below using correctly named CACHE_STORE setting
    'default' => env('CACHE_STORE', 'redis'),
    'stores' => [
        //the 'redis' cache store that is selected by the setting above when CACHE_STORE=redis in .env file
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

The driver setting refers to the `redis` driver section in the `config/database.php` file.

The connection setting refers to one of the available redis connections within the `redis` driver section in the `config/database.php` file.

The following steps will verify that these driver and connection settings exist in the `config/database.php`.

Also we will slightly modify these driver and connection settings to support redis clusters when in production.

### Step 5

Verify the redis driver setting in `config/database.php`

There should exist a `redis` setting at the root settings level in `config/database.php` snippet shown below. This is the `redis` driver referred to from the `redis` store in `config/cache.php`.

```php
'redis' => [

        //this specifies the redis client used by Laravel
        //By default REDIS_CLUSTER is not specified in the .env file so falls back to using the 'phpredis' value.
        //the 'phpredis' value requires installing the redis php extension (phpredis.so) extension using: pecl install redis.
        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            //this setting is only effective when using a managed redis cluster. No impact if redis cluster is not used.
            //falls back to the 'redis' value since by default the REDIS_CLUSTER setting is not specified in the .env file
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            //This setting adds a application specific prefix to the cache keys to distinguish between data from multiple apps using the same redis server
            //I have commented out the setting as I will use a different redis server per application
            //'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

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

Within the `redis` driver setting array there should be two connections named `default` and `cache`.

The `cache` connection is the actual connection that is referred to from the `redis` store in the `config/cache.php` file.

So we have verified that both the driver and connection that settings that we used in `config/cache.php` exist in `config/database.php`.

> Note: We can add additional connection settings in `redis` section of `config/database.php` as long as we give them their own unique name. In fact for scalability reasons we can add and use separate redis connections, from the ones used by the Laravel cache, to be used by Laravel queues and Laravel sessuon

### Step 6

In this step we will configure the `redis` driver `cache` connection in the `config/database.php` to point to a redis server docker service running in our local development environemnt.

Make changes to the `cache` and `default` connections of the `redis` driver in `config/database.php` as shown below:

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

For the `cache` connection We changed the database setting and added a prefix setting. All keys using this connection will automatically get a `c:` prefix.

> redis clusters do not support multiple databases so we are distinguishing this connection using the redis key prefix instead.

We also made the same change to the `default` connection except we use a `d:` prefix for it to distinguish it from the `cache` connection.

> As an aside, the `default` connection is used

### Step 7 -Setup the redis sever docker service

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

> If you already have a docker-compose.yml file in the project root then just add the redis service portion of the above yml code under the services section.

### Step 8- Open the project `.env` file and update the connection settings for the mailhog local mail service.

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=radar
REDIS_PORT=8002
```

> docker-compose.yml also uses settings from the .env file

### Step 9 - Startup the docker service

```bash
docker-compose up -d
```

### Step 10 - Use the cache service

In laravel when we use the `cache()` helper of the `Cache::` facade, they use the `default` cache store from config/cache.php which in turn uses the `cache` connection in the config/database.php as configured by.

When done you can shutdown the docker service

```bash
docker-compose down
```

## Creating and using non default redis cache stores

We can create additional redis stores in the stores array in config/cache.php file and configure them in the same way as we configured the default `redis` cache store.

We can then use these stores explicitly by name with the Laravel cache helper, facade or cache manager api.

Here is an example:

In config/database.php add a `cache2` connection with a `c2` key prefix.

Both the original and new cache connections are shown below:

```php

'redis' => [

        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c:',
        ],

        'cache2' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c2:',
        ],

    ],
```

> Note that both cache connections use the same cache server settings. We could if we needed for scalability add a different `REDIS_HOST_2` env setting to use a completely separate redis server in production.

Next in config/cache.php add a `redis2` store that uses the new `cache2` connection.

Both the default and new redis stores are shown below:

```php
    'default' => env('CACHE_STORE', 'redis'),
    'stores' => [
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
        //additional redis store that we can explicitly pass for example to the laravel cache facade
         'redis2' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache2',//the new redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

With this setup we could explicitly pass the string `redis2` to the cache helper or facade to use the redis store that uses the `cache2` connection to access the redis server.

## Key takeaways

The way the caches connect to your application in Laravel are through cache stores specified in the config/cache.php file.

Each cache store has an underlying driver. If the driver for a cache store is `redis` then the cache store will have a underlying connection that specifies the connection parameters of the redis server it will connect to.

This connection should be set to one of the available connections in the `redis` driver array within the config/database.php file.
