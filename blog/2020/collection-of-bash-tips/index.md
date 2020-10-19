# collection of bash tips

October 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[collection of bash tips](https://aregsar.com/blog/2020/collection-of-bash-tips)

## list processes running on system

Command to list processes running on the system:

```bash
ps aux
```

Flags:

a = show processes for all users
u = display the process's user/owner
x = also show processes not attached to a terminal

Examples:

```bash
ps aux | grep nginx

# Check what user PHP-FPM is running as (it's www-data)
ps aux | grep php

#only display user,pid and command columns in output
ps -Ao user,pid,command | grep -v grep | grep 127
```

## check servers running on ports

Command to show network statistics:

```bash
netstat -anv
```

Flags:

-a Displays all active TCP connections and the TCP and UDP ports on which the computer is listening.
-n Displays active TCP connections, however, addresses and port numbers are expressed numerically
-v Displays active TCP connections and includes the process ID (PID) for each connection

Example to list servers listening on port 80:

```bash
netstat -anv | grep "\.80" | grep LISTEN
```

Example to list server using port 9000:

```bash
netstat -anv | grep 9000
```

## Watching processes

The watch command can be used along with the `ps aux` command to poll and display process statistics continuously:

> Watch can be used to monitor other commands whose output would change over time.

Install on Mac

```bash
brew install watch
```

Examples:

```bash
#Watch command to watch the foo process

watch 'ps aux | grep foo'

#this wont work since it will execute watch over 'ps' command instead of result of entire command chain
watch ps aux | grep foo

#watch all processes
watch 'ps aux'
```

## tailing files

The tail command allows you to tail updates to files. Useful in monitoring log files

Example of how it works:

Create and tail a new test file:

```bash
touch testfile
tail -F testfile
```

launch another terminal tab and change to the `testfile` file directory:

```bash
echo 'abcd' >> testfile
# check the content of the file
cat testfile
```

Switch back to the previous tab an see the output of tail command.

## bash command tips

Various bash command tips

## file descriptors 0, 1 and 2

stdin #fd 0
stdout #fd 1
stderr #fd 2

### Variable interpolation

the ${} means contents of the variable var is interpolated with the surrounding text
a${var}text

Otherwise if we dont use ${} then becomes contents of the variable avartext,
$avartext

### Command evaluation

export PYTHON_HOME=\$(which python)

### Open html file or URL in default browser

$ open index.html
$ open https://www.google.com

### multiline file input

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

### Curl command piping

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
curl -o - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash

### Wget command piping

wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -qO - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -q -O- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -q -O - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash

## bash script tips

Various script file tips

### Give executable permissions Script

$ chmod +x myscript.sh
$ ./myscript

### set environment variable used by executed script

FOO=bar bash -c 'command1 args'

FOO=bar bash -c 'command1 args | command2'

### Setting Script language

```bash
!env /usr/local/bin/node

!/usr/local/bin/node
```

### Script error flags

```bash
-e
```

### Script command line arguments

$*, $@, $# ,$0, \$1 used in bash script to refer to script and arguments

$ script $arg1 \$arg2

"$*" quotes the entire list of arguments "$arg1 \$arg2" output as a string

"$@" quotes all arguments individual "$arg1" "\$arg2" output as a string

When they are not quoted, $* and $@ are result in the same argument list string.

Should not use without quotes because it will break if arguments contain spaces or wildcards.

number of arguments

\$#

# script text

\$0

# first argumemt

\$1
