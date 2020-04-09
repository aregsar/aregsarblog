# Configure Laravel Queue To Use Redis

April 9, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Configure Laravel Queue To Use Redis](https://aregsar.com/blog/2020/configure-laravel-queue-to-use-redis)




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

