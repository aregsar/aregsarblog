# Use Redli CLI To Connect To Redis

April 30, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article we will download and use a `Go` Language based Redis CLI that mimics the standard `redis-cli` client commands and adds TLS connection capability that will be needed when connecting to a production `Redis` instance.

The `Redli` CLI documentation can be found at [redli](https://github.com/IBM-Cloud/redli).

## Installation on macOS

```bash
wget https://github.com/IBM-Cloud/redli/releases/download/v0.4.4/redli_0.4.4_darwin_amd64.tar.gz
tar xvf redli_0.4.4_darwin_amd64.tar.gz
mv redli /usr/local/bin/
rm redli_0.4.4_darwin_amd64.tar.gz
```

> Note: If you get a `dyld: Library not loaded` message when using `wget` as I did, you can run `brew upgrade wget` to resolve the issue.

## Installation on ubuntu

```bash
wget https://github.com/IBM-Cloud/redli/releases/download/v0.4.4/redli_0.4.4_linux_amd64.tar.gz
tar xvf redli_0.4.4_linux_amd64.tar.gz
sudo mv redli /usr/local/bin/
rm redli 0.4.4_linux_amd64.tar.gz
```

## Connecting to local macOS Redis instance

```bash
redli -h 127.0.0.1 -p 6379
```

By default redli connects to localhost 127.0.0.1 so we could just run:

```bash
redli -p 6379
```

By default redli connects to port 6379 so we could also just run:

```bash
redli
```

## Connecting to Redis server instance running in Docker container

Assumeing the standard redis port 6379 for the Redis server instance running in the container is mapped to port 8000 on the localhost:

```bash
redli -p 8000
```

## Connecting to the Digitalocean managed Redis instance

```bash
# the --tls option connects to managed redis database over SSL
redli --tls -h host -a password -p port
```

## The redis-cli and redli command prompts

When we connect to the Redis server using the standard `redis-cli` redis client we get command prompt:

```bash
127.0.0.1:6379>
```

However when we connect using the `redli` client we will get the following command prompt:

```bash
Connected to 5.0.8
>
```

To exit the prompt and terminate the connection session we can type `Ctrl+c` in the terminal.

## Executing Redis commands after connecting to the Redis server

Regardless of which CLI you use to connect to the redis server the following redis commands will work the same.

Command to check the redis connection:

```bash
> ping
PONG
```

Example commands to set and get values:

```bash
> set name joe
OK
> get name
"joe"
> del name
(integer) 1
> get name
nil
> set name "john doe"
OK
> get name
"john doe"
> keys *
 1) "name"
> set f:name joe
OK
> get f:name
"joe"
> keys *
 1) "name"
 2) "f:name"
> flushdb
OK
> keys *
>
```

## Useful links

[How to Connect to Redis Database Clusters with redli](https://www.digitalocean.com/docs/databases/redis/how-to/connect)

[redli](https://github.com/IBM-Cloud/redli)

