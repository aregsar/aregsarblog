# collection of bash tips

October 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[collection of bash tips](https://aregsar.com/blog/2020/collection-of-bash-tips)

## check processes running on system

ps aux

a = show processes for all users
u = display the process's user/owner
x = also show processes not attached to a terminal

```bash
ps aux

ps aux | grep nginx

# Check what user PHP-FPM is running as (it's www-data)
ps aux | grep php

#only display user,pid and command columns in output
ps -Ao user,pid,command | grep -v grep | grep 127
```

## check servers running on ports

```bash
netstat -an | grep "\.80" | grep LISTEN

netstat -anv | grep 9000
```

## watching processes

Watch command to watch the foo process

watch 'ps aux | grep foo'

#this wont work since it will execute watch over 'ps' command instead of result of entire command chain
watch ps aux | grep foo

## tailing logs

tail

## assorted commands

## bash scripts

file descriptors 0, 1 and 2
stdin #fd 0
stdout #fd 1
stderr #fd 2

bash script error flags

### set environment variable used by executed script:

FOO=bar bash -c 'command1 args'

FOO=bar bash -c 'command1 args | command2'

============

### Variable interpolation

the ${} means contents of the variable var is interpolated with the surrounding text
a${var}text

Otherwise if we dont use ${} then becomes contents of the variable avartext,
$avartext

============

Command evaluation

export PYTHON_HOME=\$(which python)

==========

open html file or URL in default browser

$ open index.html
$ open https://www.google.com

==========

$*, $@, $# ,$0, \$1 used in bash script to refer to script and arguments

$ script $arg1 \$arg2

# "$*" quotes the entire list of arguments "$arg1 \$arg2" output as a string

# "$@" quotes all arguments individual "$arg1" "\$arg2" output as a string

# When they are not quoted, $* and $@ are result in the same argument list string.

# Should not use without quotes because it will break if arguments contain spaces or wildcards.

# number of arguments

\$#

# script text

\$0

# first argumemt

\$1

=============

give executable permissions to a script then execute it

$ chmod +x myscript.sh
$ ./myscript
================

#!env /usr/local/bin/node

#!/usr/local/bin/node

echo "hello\n" > test
cat message

\$ cat > hello << EOL

> !/bin/bash
> hello
> world
> EOL

\$ cat hello

\$ cat >> hello << EOL

> once
> more
> EOL

\$ cat hello

============

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
curl -o - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash

wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -qO - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -q -O- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -q -O - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
