# Use Redli CLI To Connect To Redis

April 30, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article we will download and use a GO Language based Redis CLI that mimics the original redis-cli client and adds TLS connection capability that will be needed when connecting to a production Redis instance.

The Redli CLI can be found at `https://github.com/IBM-Cloud/redli` 

## Installation macOS

```bash
wget https://github.com/IBM-Cloud/redli/releases/download/v0.4.4/redli_0.4.4_darwin_amd64.tar.gz
tar xvf redli_0.4.4_darwin_amd64.tar.gz
mv redli /usr/local/bin/
rm redli_0.4.4_darwin_amd64.tar.gz
```

> Note: If you get a `dyld: Library not loaded` message when using `wget` you should run `brew upgrade wget` to resolve the issue

## Installation ubuntu

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

By default redli connects to localhost 127.0.0.1 so we could just do:

```bash
redli -p 6379
```

By default redli connects to port 6379 so we could just do:

```bash
redli
```

## Connecting to redis instance running in Docker container

Assume the standard redis port 6379 for the redis instance running in the container is mapped to port 8000 on the localhost

```bash
redli -p 8000
```

## Connecting to Digitalocean managed Redis instance

```bash
# the --tls option connects to managed redis database over SSL
redli --tls -h host -a password -p port
```

## The redli command prompt

When connecting with the standard redis client `redis-cli` we get command prompt:

```bash
127.0.0.1:6379>
```

However when we run the redli client we get the command prompt:

```bash
Connected to 5.0.8
>
```

To exit the prompt and terminate the connection type ctrl C.

## Executing Redis commands after connecting

Commands to check the redis connection:

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
