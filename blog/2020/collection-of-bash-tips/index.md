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

## File descriptors 0, 1 and 2

Many bash commands use stdin,stdout and stderr that are represented as file descriptor (fd) numbers:

stdin => 0
stdout => 1
stderr => 2

### dump all environment variables

```bash
env
```

### Variable interpolation

To interpolate the variable `$var` use `${var}`.

The `{}` brackets wrapped around text portion of a variable `$var` means that the contents of the variable `var` is interpolated with the surrounding text.

Otherwise `$varabcd` would be assumed the `$varabcd` variable, instead of the `$var` variable followed by the text `abcd` like `${var}abcd` would.

### Command evaluation

To evaluate a command and interpolate its output with surrounding text use the \$() wrapper around the command.

The format is \$(command):

Example:

```bash
#interpolate the path of the python executable determined using the `which python` command.
export PYTHON_HOME=\$(which python)
```

### Open html file or URL in default browser

$ open index.html
$ open https://www.google.com

### Insert newline into file

The `echo` command will by default add a newline after the text we echo to a file. So everytime we append another string to the file using echo, the new string will appear on the next line.

```bash
echo "hello" > test
echo "world" >> test
cat test
```

Adding the `\n` character at the end of the string will insert an additional newline.

```bash
echo "hello\n" > test
echo "world" >> test
cat test
```

We can add as many `\n` characters at the end of the string to insert more newlines

```bash
echo "hello\n\n\n\n" > test
echo "world" >> test
cat test
```

The newline can placed anywhere within the string as well to insert newline between text

```bash
echo "hello\nworld" > test
cat test

echo "\n\nhello\nworld" > test
cat test
```

### multiline

By using the `EOL` symbol with the `cat` command we can interactively add more lines until the closing `EOL` symbol on the last line completes the command.

Create file and add multiple lines:

```bash
cat > test << EOL
>!/bin/bash
> hello
> world
> EOL

cat test
```

Keep appending more lines:

```bash
cat >> test << EOL
> once
> more
> EOL

cat test
```

### Curl command piping

Variations of the same command

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
curl -o - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

### Wget command piping

Variations of the same command

```bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -qO - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -q -O- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -q -O - https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

## bash script tips

Various script file tips

### Give executable permissions Script

Assume we have a script file called myscript:

```bash
#make the script file executable
chmod +x myscript

#execute the script
./myscript
```

### set environment variable used by executed script

FOO=bar bash -c 'command1 args'

FOO=bar bash -c 'command1 args | command2'

### Setting Script language

starting a script file with the `#!` line tells the shell process running the script the interpreter to use to execute the script.

Bash script example:

Use explicit bash interpreter

```bash
#!/bin/bash
```

Alternate form uses default bash interpreter

```bash
#!/usr/bin/env bash
```

PHP script example:

```bash
#!/usr/local/bin/php
```

### Script error flags

Use the set command to set the error handling strategy of the script

```bash
#!/bin/bash
set -euo pipefail
```

“-e” ensures that your script stops on first command failure.
“-u” ensures that your script exits on the first unset variable encountered.
“-o pipefail” ensures that if any command in a set of piped commands failed, the overall exit status is the status of the failed command

### Script command line arguments

$*, $@, $# ,$0, \$1 used in bash script to refer to script and arguments

$ script $arg1 \$arg2

"$*" quotes the entire list of arguments "$arg1 \$arg2" output as a string

"$@" quotes all arguments individual "$arg1" "\$arg2" output as a string

When they are not quoted, $* and $@ are result in the same argument list string.

Should not use without quotes because it will break if arguments contain spaces or wildcards.

number of arguments

\$#

script text

\$0

first argument

\$1

## resources

https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-a-linux-vps

https://ashishb.net/all/the-first-two-statements-of-your-bash-script-should-be

https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail

https://nickjanetakis.com/blog/executing-code-after-an-error-occurs-with-bash-when-using-set-e

https://nickjanetakis.com/blog/allowing-for-errors-in-bash-when-you-have-set-e-defined

https://nickjanetakis.com/blog/here-is-why-you-should-quote-your-variables-in-bash
