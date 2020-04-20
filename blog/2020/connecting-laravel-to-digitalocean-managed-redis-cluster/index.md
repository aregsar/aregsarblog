# Connecting Laravel To Digitalocean Managed Redis Cluster

April 20, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Laravel environment settings

From `https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster`

```ini
REDIS_HOST=tls://your_redis_host.db.ondigitalocean.com
REDIS_PASSWORD=your_redis_password
REDIS_PORT=25061
```

Test the connection by adding the following to any laravel route or action

```php
    try{
        var_dump(Redis::connection()->ping());
    } catch (Exception $e){
        $e->getMessage();
    }
```

## Using the Redli CLI client

From https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-database-ubuntu-18-04

The default Redis command line client, redis-cli does not support SSL So DigitalOcean recommends using the redli client that natively supports TLS connection required by the managed Redis database.

You can check out redli here:

How to Connect to Redis Database Clusters with redli
https://www.digitalocean.com/docs/databases/redis/how-to/connect/

https://github.com/IBM-Cloud/redli

https://github.com/IBM-Cloud/redli/releases

https://github.com/IBM-Cloud/redli#usage

First we need to download and install redli:

```bash
wget https://github.com/IBM-Cloud/redli/releases/download/v0.4.4/redli_0.4.4_linux_amd64.tar.gz
tar xvf redli_0.4.4_linux_amd64.tar.gz
sudo mv redli /usr/local/bin/
rm redli 0.4.4_linux_amd64.tar.gz
```

Now we can connect over SSL

```bash
# the --tls option connects to managed redis database over SSL
redli --tls -h host -a password -p port
```

## Using the standard Redis CLI with workarounds

If you want to use the standard redis clinet, there are some workarounds by setting up a SSL tunnel.

One option to set SSL tunnel is to use Stunnel

From `https://helpmanual.io/help/redis-cli/`

Using Redis CLI in cluster mode:

```bash
redis-cli -c
```

https://brewinstall.org/Install-stunnel-on-Mac-with-Brew/

https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-redis-over-tls-with-stunnel-and-redis-cli

https://redislabs.com/blog/stunnel-secure-redis-ssl/

https://bencane.com/2014/02/18/sending-redis-traffic-through-an-ssl-tunnel-with-stunnel/

## Restricting access to managed Redis cluster

While SSL secures the data transmitted to and from the Redis cluster, it does not restrict who can connect to the cluster.

The Managed Redis database allowes restricting network access based on client IP Address and VPC (Virtual Private Cloud) settings that can be configured via the Digitaloacean console or API

## Using a jump host to proxy requests to redis

https://www.redhat.com/sysadmin/ssh-proxy-bastion-proxyjump

https://www.madboa.com/blog/2017/11/02/ssh-proxyjump/

https://starkandwayne.com/blog/jumping-seamlessly-thorough-tunnels-a-guide-to-ssh-proxy/

https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts