# Configure Laravel Cache To Use Redis

April 9, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Configure Laravel Cache To Use Redis](https://aregsar.com/blog/2020/configure-laravel-cache-to-use-redis)


##################################################################################################

config/cache.php:

has list of stores e.g redis or file stores
each store has a driver like 'redis' (declared in config/database.php) or 'file'
and possibly a connection(from one of the named connections associated with the driver
in config/database.php) like 'default' or 'cache' if the driver is one that is
specified in config/database.php
 
   'stores' => [
        # we can explicitly
        'redis' => [
            # uses the redis driver defined in config/database.php
            'driver' => 'redis',#the 'redis' driver in config/database.php
            'connection' => 'cache',#uses the 'cache' configuration of the 'redis' driver in config/database.php
        ],
        'file' => [
            #uses the file driver
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],

    ],


#select a store from stores specified in config/cache.php
$value = Cache::store('file')->get('foo');

    #uses the 'default' store specified in config/cache.php by default
    $value = Cache::get('key');

    #setting in config/cache.php selects the file store
    'default' => 'file'

    #setting in config/cache.php selects the redis store by default
    'default' => 'redis'

    #setting in config/cache.php  selects a store based on env variable
    #the second argument sets the default to the file store if
    #the env variable is not present
    'default' => env('CACHE_DRIVER', 'file'),

    The last statement that uses the env variable is present when a new larave
    project is created

---------------------
    #config for the Cache facade used in the application
config/cache.php:

    # when using cache facade without specifying store this will be used to refer to
    # the 'store' we are using by default
    # set CACHE_DRIVER=redis in .env to set the default store to 'redis'
    # refers to one of the stores specified in the 'stores' array in this file
    # if env var not specified, the second param 'file' refers to the 'file' store in the 'stores' array in this file
    'default' => env('CACHE_DRIVER', 'file'),
   
    'stores' => [
        # we can explicitly
        'redis' => [
            # uses the redis driver defined in config/database.php
            'driver' => 'redis',#the 'redis' driver in config/database.php
            'connection' => 'cache',#uses the 'cache' configuration of the 'redis' driver in config/database.php
        ],
        'file' => [
            #uses the file driver
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],

    ],


######################################

 config/queue.php has conections
'redis' => [
            'driver' => 'redis',
            'connection' => 'default',
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],

also has a default  connection selected by name from the connections in  config/queue.php
'default' => env('QUEUE_CONNECTION', 'sync'),

#uses the default connection setting in config/queue.php which includes the queue name to be used
#the connection also specifies the driver and associated connection specified in config/database.php
Queue::push(new InvoiceEmail($order));

#names the queue explicitly but uses the default connection setting in config/queue.php
#overrides the queue name specified in the default connection setting
Queue::pushOn('emails', new InvoiceEmail($order));

#explicitely selecting a queue connection from config/queue.php connections
$connection = Queue::connection('connection_name')->push(new InvoiceEmail($order));

#Alternatively can get a queue instance from queue manager
$queueManager = app('queue');
$queue = $queueManager->connection('redis');
$queue->push(new InvoiceEmail($order));


----------------------

config/queue.php:
 #selects default sync connection from connections array in this file
 #QUEUE_CONNECTION=redis in .env file
 #existing QUEUE_CONNECTION=sync in .env file
 'default' => env('QUEUE_CONNECTION', 'sync'),

 'connections' => [

        #sync connection
        'sync' => [
            'driver' => 'sync',
        ],

        #redis connection
        'redis' => [
            #can add and set from QUEUE_REDIS_DRIVER in .env
            'driver' => 'redis',#hardcodes 'redis' driver from config/database.php
            #can add and set from QUEUE_REDIS_CONNECTION in .env
            'connection' => 'default',#hard coded 'default' connection from 'redis' driver connection in config/database.php
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],
]




# Laravel Managed MySql connection settings


In my .env add:

MYSQL_ATTR_SSL_CA="/full/path/to/ca-certifcate.crt" 
SSL_MODE=required


In config/database.php under mysql connection add:

'ssl_mode' => env('SSL_MODE'),

Then run:

php artisan config:cache


## Changing MySql Connection password mode

Install instructions MySQL Secure Shell to connect using the mysqlsh cli:

https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install-osx-quick.html

DigitalOcean Managed Databases using MySQL 8+ are automatically configured to use caching_sha2_password authentication by default

Connect using mysqlsh then run the MySql command:
ALTER USER use_your_user IDENTIFIED WITH mysql_native_password BY 'use_your_password';

> Note: according to `https://www.digitalocean.com/docs/databases/mysql/resources/troubleshoot-connections/` you only need to run this command if your mysql client is PHP7.1 or earlier.
So you should not need to do this for PHP7.4

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-scalable-laravel-6-application-using-managed-databases-and-object-storage

