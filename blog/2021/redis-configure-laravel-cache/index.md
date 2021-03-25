# Redis Configure Laravel Cache

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Cache driver and cache store confusion due to setting misnaming

There is a misnomer in the cache config file regarding the default cache store env var causing confusion.
The CACHE_DRIVER env setting needs to be renamed to CACHE_STORE

### Original mis-named cache store configuration

The `CACHE_DRIVER` setting in `.env` file that is used in the `config/cache.php` file is mis-named.

```ini
# misnamed env var that selects the cache store (not the cache driver)
CACHE_DRIVER=file
```

The `CACHE_DRIVER` setting is misnamed since it is used to select a cache store, not a cache driver.

Below I am showing a snippet of the cache stores section in the `config/cache.php` file to show the misnaming issue.
I have annotated the settings to describe the issue.

```php
    //Default Cache Store used by laravel caching funcions
    //Misnamed env var that selects the cache store (not cache driver)from the Cache Stores array below
    'default' => env('CACHE_DRIVER', 'file'),
    'stores' => [
        //this is the 'file' cache store selected by the 'default' setting above when CACHE_DRIVER=file in .env file
        'file' => [
                'driver' => 'file', //this is the actual cache driver which is not what is being selected
                'path' => storage_path('framework/cache/data'),
            ],
    ]
```

The `default` setting above uses the `CACHE_DRIVER` env variable to select a named cache store from the `stores` array.

So in reality it is selecting a cache store not a cache driver which is confusing.

Therefore the `CACHE_DRIVER` env setting needs to be renamed to `CACHE_STORE` to avoid confusion.

### Correcting (re-nameing) the cache store configuration

Open the `.env` file and rename the CACHE_DRIVER setting name to CACHE_STORE

```ini
#correctly named environment variable that selects the cache store
CACHE_STORE=file
```

Open the config/cache.php file and change and rename CACHE_DRIVER to CACHE_STORE

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the file cache store from the cache 'stores' array below using correctly named CACHE_STORE env setting
    'default' => env('CACHE_STORE', 'file'),
    'stores' => [
        //the 'file' cache store that is selected by the 'default' setting above when CACHE_STORE=file in .env file
        'file' => [
                'driver' => 'file', //the actual cache driver for the 'file' store
                'path' => storage_path('framework/cache/data'),
            ],
    ]
```

## Selecting the redis store as our default store

Now that we have properly renamed the CACHE_DRIVER env variable to CACHE_STORE we can change its value to select the redis cache store setting within the `stores` array

Open the `.env` file and change the value to the `redis` store.

```ini
CACHE_STORE=redis
```

Open the config/cache.php file and optionally change the second (default/fallback) parameter of the env() function to `redis`.

With these changes the the `default` cache store now selects the `redis` store.

The selected `redis` store in the `stores` section is now shown as well.

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the cache redis store from the Cache Stores array below using correctly named CACHE_STORE setting
    'default' => env('CACHE_STORE', 'redis'),
    'stores' => [
        'file' => [
                'driver' => 'file',
                'path' => storage_path('framework/cache/data'),
            ],
        //the 'redis' cache store that is selected by the setting above when CACHE_STORE=redis in .env file
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

## The redis driver connection

I have shown below the `redis` driver section in the config/database.php file.
This is the actual driver that is selected by `driver` setting of the `redis` store in the config/cache.php file

Within this driver we see two connection settings `default` and `cache`. The later is the one that is selected by the `connection` setting of the `redis` store in the config/cache.php file.

```php
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
            'url' => env('REDIS_URL'),//This setting is not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],

    ],
```

> We can add any number of redis connections here, named any way we want. We can then use those connections from any cache stores we add to other configuration files.

## Setting the redis driver connection to a local redis service

Step 1 we need to make a change to the `cache` connection in confi/database.php as shown below:

```php
'redis' => [
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

We changed the database setting and added a prefix setting. All keys using this connection will automatically get a `c:` prefix.

> redis clusters do not support multiple databases so we are distinguishing this connection using the redis key prefix instead.

Step 2 -Setup the redis sever docker service

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

Step 2- Open the project `.env` file and update the connection settings for the mailhog local mail service.

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=radar
REDIS_PORT=8002
```

> docker-compose.yml also uses settings from the .env file

Step 3 - Startup the docker service

```bash
docker-compose up
```

Step 4 - Use the cache service

In laravel when we use the `cache()` helper of the `Cache::` facade, they use the `default` cache store from config/cache.php which in turn uses the `cache` connection in the config/database.php as configured by.

We can create additional redis stores in the stores array in config/cache.php configure them in the same way as we configured the `redis` cache store, and use these stores explicitly by name with the Laravel cache helper or facade api.

Here is an example:

In config/database.php add a `cache2` connection with a `c2` key prefix

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

> Note that both cache connections use the same cache server settings. We could if we needed for scalability add a different `REDIS_HOST_2` env setting to use a completely separate redis server.

In config/cache.php add a redis2 store that uses the new cache2 connection

```php
    'stores' => [

        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
         'redis2' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache2',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

With this setup we could explicitly pass the string `redis2` to the cache helper or facade to use the redis server accessed via the `cache2` connection.

When done you can shutdown the docker service

```bash
docker-compose down
```
