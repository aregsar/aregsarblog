# Connecting Laravel To Digitalocean Managed Redis Cluster

April 20, 2020 by [Areg Sarkissian](https://aregsar.com/about)

This post is meant as a quick list reference of links and commands for connecting to the DigitalOcean managed Redis database.

## Laravel environment settings

Connection settings listed at [how-to-setup-laravel-with-digitalocean-managed-redis-cluster](https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster)

```ini
REDIS_HOST=tls://your_redis_host.db.ondigitalocean.com
REDIS_PASSWORD=your_redis_password
REDIS_PORT=25061
```

Test the connection by adding the following to any Laravel route or controller action:

```php
    try{
        var_dump(Redis::connection()->ping());
    } catch (Exception $e){
        $e->getMessage();
    }
```

## Using the Redli CLI client

The default Redis command line client, redis-cli does not support SSL So Digitalocean recommends using the redli client that natively supports TLS connection required by the managed Redis database.

Instructions for using the Redli Redis client are listed at [how-to-connect-to-managed-database-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-database-ubuntu-18-04)

You can check out more info about the Redis redli CLI here:

[How to Connect to Redis Database Clusters with redli](https://www.digitalocean.com/docs/databases/redis/how-to/connect)

[redli](https://github.com/IBM-Cloud/redli)

[redli releases](https://github.com/IBM-Cloud/redli/releases)

[redli usage](https://github.com/IBM-Cloud/redli#usage)

To setup redli CLI on our local machine we first we need to download and install redli using instructions below:

```bash
wget https://github.com/IBM-Cloud/redli/releases/download/v0.4.4/redli_0.4.4_linux_amd64.tar.gz
tar xvf redli_0.4.4_linux_amd64.tar.gz
sudo mv redli /usr/local/bin/
rm redli 0.4.4_linux_amd64.tar.gz
```

Now we can connect to the managed redis instance over SSL:

```bash
# the --tls option connects to managed redis database over SSL
redli --tls -h host -a password -p port
```

## Using the standard Redis CLI with workarounds

If you want to use the standard redis client, there are some workarounds to be able to connect to the managed Redis instance by setting up a SSL tunnel.

One option to set SSL tunnel is to use `Stunnel`.

Instructions to do so are at [https://helpmanual.io/help/redis-cli](https://helpmanual.io/help/redis-cli)

Using redis-cli in cluster mode:

```bash
redis-cli -c
```

More links:

[Install-stunnel-on-Mac-with-Brew](https://brewinstall.org/Install-stunnel-on-Mac-with-Brew)

[how-to-connect-to-managed-redis-over-tls-with-stunnel-and-redis-cli](https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-redis-over-tls-with-stunnel-and-redis-cli)

[stunnel-secure-redis-ssl](https://redislabs.com/blog/stunnel-secure-redis-ssl)

[sending-redis-traffic-through-an-ssl-tunnel-with-stunnel](https://bencane.com/2014/02/18/sending-redis-traffic-through-an-ssl-tunnel-with-stunnel)

[how-to-install-and-secure-redis-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04)

## Restricting access to managed Redis cluster

While SSL secures the data transmitted to and from the Redis cluster, it does not restrict who can connect to the cluster.

The Managed Redis database allows restricting network access based on client IP Address and VPC (Virtual Private Cloud) settings that can be configured via the Digitaloacean console or API.

## Using a jump host to proxy requests to redis

We can use jump hosts to connect to our managed Redis database running in a VPC.

The links below will provide information on how to setup a jump host.

[ssh-proxy-bastion-proxyjump](https://www.redhat.com/sysadmin/ssh-proxy-bastion-proxyjump)

[ssh-proxyjump](https://www.madboa.com/blog/2017/11/02/ssh-proxyjump)

[jumping-seamlessly-thorough-tunnels-a-guide-to-ssh-proxy](https://starkandwayne.com/blog/jumping-seamlessly-thorough-tunnels-a-guide-to-ssh-proxy)

[Proxies_and_Jump_Hosts](https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts)
