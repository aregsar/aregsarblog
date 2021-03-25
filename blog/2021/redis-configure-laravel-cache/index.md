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

Within this driver we see two connection settings `default` and `cache`. The later is the one that is selected by the `connection` setting of the `redis` store in the config/cache.php file.

> We can add any number of redis connections here, named any way we want. We can then use those connections from any cache stores we add to other configuration files.

## Setting the redis driver connection to a local redis service

To configure the `cache` connection
