# Redis Configure Laravel Queue

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

In this post we will configure Laravel to use a Redis server running in a local docker container to queue jobs for background workers.

Out of the box the Laravel queue is configured to use synchronous mode which handles queued jobs inline as part of each web request.

## Steps to configure Laravel to use Redis for Queues

To start, go to the root of your Laravel project then follow steps below:

### Step 1 - Install the Laravel Redis driver

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 2 - Setup the Redis sever docker service

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 3 - Set the connection settings for the docker Redis service

Refer to step 2 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 4 - Configure the Redis driver settings

Refer to step 4 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step
