# Collection Of Bash Tips

October 22, 2020 by [Areg Sarkissian](https://aregsar.com/about)

This post list a collection of bash commands and scripts that I have collected over the years to be used as a reference.

I also list resources on various bash topics at the end of the post.

## list processes running on system

Use the `ps` command to list processes running on the system:

```bash
ps aux | grep php
```

Flags:

a = show rows processes for users attached to terminal session
u = display the process's user/owner
x = also show rows of processes not attached to a terminal

Using any the above flags will display all columns.

Instead we can use the `-Ao` flag with only the specific columns names that we want displayed:

```bash
# only display uid,user,pid and command columns in output
ps -Ao uid,user,pid,command | grep php
```

## Check servers running on ports

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

### stdout, stderr and redirects

Many bash commands use stdin, stdout and stderr that are represented using file descriptor numbers:

stdin => 0
stdout => 1
stderr => 2

File Descriptor symbols allow redirects to point to the stdout and stderr file descriptors.

Below are the file descriptor symbols for stdin and stdout:

```bash
# filedescriptor 1 symbol for stdout
&1

# filedescriptor 2 symbol for stder
&2
```

Redirect symbols point the specified or default descriptor to a destination file where the content written to that descriptor will be sent to:

Below are the redirect symbols for stdin and stdout:

```bash
#redirect stdout file descriptor 1
1>

#redirect stdout file descriptor 1. File descriptor 1 is assumed as the default.
>

#redirect stderr file descriptor 2
2>
```

These redirect symbols will overwrite the content of the destination they redirect to, if the command is re executed.
If we want to redirect and append the outputs of the command we need to use the append version of the symbols shown below:

```bash
#redirect and append stdout file descriptor 1
1>>

#redirect and append stdout file descriptor 1. File descriptor 1 is assumed as the default.
>>

#redirect and append stderr file descriptor 2
2>>
```

We can use the stdout redirect symbol to redirect output of a command written to stdout to a file instead using the default stdout redirect symbol:

```bash
some_command > some_file
```

Or the equivalent using the explicit stdout redirect symbol:

```bash
some_command 1> some_file
```

We can also redirect any errors from a command written to stderr to a file instead.

```bash
some_command_with_errors 2> some_file
```

Some examples below:

```bash
#echo text to standar output
echo 'abcd'
#echo text to standard output that is redirected to a file
echo 'abcd' > testfile
echo 'xyz' >> testfile

#echo text to standard output that is redirected to a file
echo 'abcd' 1> testfile
echo 'xyz' 1>> testfile

#redirect error output of the failed ls command to the test file
ls -0 2> testfile
ls -0 2>> testfile
```

We can redirect the stderr to the stdout file descriptor as well
using the expression `2>&1` which is the concatenation of the `2>` stderr redirect to the `&1` stdout file descriptor.
This will be useful in commands with multiple redirects that we will see in the next section.

### Multiple redirects

The redirect symbol `>` or append redirect symbols `>>` can be used more then once at the end of the command line to redirect stdin or stderr.

```bash
#echo text to standard output, standard error is redirected to standard output and standard output is redirected to file
#the first redirect symbol redirets the output of some_command to somefile
# the secont redirect symbol in 2>&1 redirects the standard error represented by the file descriptor 2 to the standard
#outout represented by the &1
some_command > somefile 2>&1

#the reverse will not work to send the stderror content into somefile since the the stderror is redirected to stdout before
#the standard out is redirected to somefile.
some_command 2>&1 > somefile

#There is a shorthand that is the equivalent:
some_command &> somefile
```

Below are examples:

```bash
#The following three echo commands are equivalent:
echo 'abcd' > testfile 2>&1
echo 'abcd' 1>testfile 2>&1
echo 'abcd' &> testfile

#The following three echo commands are equivalent:
#here the stdout and std in are redirected to /dev/null instead of a file
echo 'abcd' > /dev/null 2>&1
echo 'abcd' 1> /dev/null 2>&1
echo 'abcd' &> /dev/null

#Of course the above will not generate any erros. so a more practical example
#can use the ls command with an invalid flag to generate an error:
#redirect to file and redirect error to output to same file
ls -0 > testfile 2>&1

#or to suppress any output
ls -0 > /dev/null 2>&1

#If we send stdin and stdout to same text file the effect is not the same as the ouput from each will not get interleaved
ls -0 >testfile 2>testfile

#redirect to separate files (note testfile1 will be empty since we only have stderr outout)
ls -0 >testfile1 2>testfile2

#a valid ls command will have an empty testfile1 file because we will only have stdout output)
ls >testfile1 2>testfile2

#A more practical examlple is this command to run the laravel scheduler as a cron job
* * * * * cd /PATH-TO-LARAVEL-APP && php artisan schedule:run >> /dev/null 2>&1
```

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

### multiline interactive input

By using the `EOL` symbol with the `cat` command we can interactively add more lines until the closing `EOL` symbol on the last line completes the command.

the stdin redirect symbol `<<` is used to redirect input lines typed in interactively in the terminal as stdin that then get redirected to stdout using the stdout symbol `>`. The `EOL` as the session terminator keyword. Any other word can be used in its place as long as the word at both ends match. The keywords will not be added to the content of the file.

> Note that if we had used the `<` stdin redirection symbol `cat > test < EOL`, then `EOL` will be treated as a filename that is redirected by stdin, not an interactive session.

Create file and add multiple lines:

```bash
cat > test << EOL
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

Overwrite content:

```bash
cat > test << EOL
> hello
> again
> EOL

cat test
```

We can use any characters as the terminator as long as both sides match.

```bash
cat > test << MYTerm
> hello
> world
> MYTERM

cat test
```

### multiline input in a shell script

First we create a shell script file:

```bash
touch multilinescript
```

Then we add the following to the multilinescript script file:

> We can actually be recursive and interactively add content to the multilinescript file using the iteractive multiline technique we learned in last section!

```bash
#!/bin/bash

cat > test << EOL
hello
world
script
EOL
```

Run the script:

```bash
#give execute permissions to owner
chmod u+x multilinescript
#execute the script
./multilinescript
```

The result will be a file names test created with following content:

```bash
hello
world
script
```

## cat to files

```bash
#create a file with some text
echo 'abcd' > test

#print to output
cat test

#create a new file test1 from test file
cat test > test1

#print to output
cat test1

#append into test 1
cat test >> test1

#print to output
cat test1

#create a new file test2 from test1 file
cat test1 > test2

#print to output
cat test2

#print concatenated files to output
cat test1 test2

#concatenate both files into new file 3
cat test1 test2 > test3

#print to output
cat test3

#append concatenated file to test 3
cat test1 test2 >> test3

#print to output
cat test3
```

## cat standard input

We saw how use the cat command to output to a file, by interactive input through stdin.

We can also use the cat command to get text from stdin and use the stdin redirect to redirect stdin to a file.

Instead of explicitly having to specify `/dev/stdin`, The cat command also accepts a single dash character as a shorthand that is equivalent to `/dev/stding`.

Both forms are shown in the example below:

```bash
cat ~/.ssh/id_rsa.pub | ssh root@<YOUR_IP> 'cat /dev/stdin >> ~/.ssh/authorized_keys'

#shorthand with `cat -`
cat ~/.ssh/id_rsa.pub | ssh root@<YOUR_IP> 'cat - >> ~/.ssh/authorized_keys'
```

The cat command writes the id_rsa.pub to standard output of our local machine which is piped
into the ssh command as standard input of the ssh command. Cat is then run on the remote machine via ssh
command line argument that redirects the stdin to append the authorized_keys file using the `>>` append redirection symbol.

## run command in background

```bash
command &
command &>/dev/null &
ps -eaf | grep php
# -A flag means Select all processes, including those of other users.
ps -Ao uid,user,pid,command | grep php
#pid=1713
kill 1713
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

### Give executable permissions Script for owner user only

Assume we have a script file called myscript:

```bash
#make the script file executable
chmod u+x myscript

#execute the script
./myscript
```

### set environment variable used by executed script

FOO=bar bash -c 'command1 args'

FOO=bar bash -c 'command1 args | command2'

### Setting Script language

starting a script file with the `#!` line tells the shell process running the script the interpreter to use to execute the script.

Use explicit bash interpreter:

```bash
#!/bin/bash
```

Alternate form, using the default bash interpreter:

```bash
#!/usr/bin/env bash
```

PHP script example:

```bash
#!/usr/local/bin/php
```

### Script error flags

Use the set command to set the error handling strategy of the script.
Include it after the first line `#!/usr/bin/env bash`.

```bash
#exit script anywhere the in the command pipeline on error or unset variable
set -euo pipefail
#also prints out commmands and arguments values that are executed
set -euxo pipefail
```

“-e” ensures that your script stops on first command failure.
“-u” ensures that your script exits on the first unset variable encountered.
“-o pipefail” ensures that if any command in a set of piped commands failed, the overall exit status is the status of the failed command

### Script command line arguments

`$*, $@, $# ,$0, $1` are used in bash script to refer to script and arguments

```bash
script $arg1 $arg2
```

When `$\*` and `$@` are quoted:

`"$*"` quotes the entire list of arguments `"$arg1 $arg2"` output as a string

`"$@"` quotes all arguments individually `"$arg1" "$arg2"` output as a string

When `$\*` and `$@` are not quoted they result the argument list as is and not quoted.

Should not use without quotes because it will break if arguments contain spaces or wildcards.

Example of a bash alias I setup in my `.zshrc` file that uses the `"$*"` arguments.

```bash
function gg {
    #quote all arguments as a single message
    #so we can write commit message without quotes
    #gg this is my commit message sans quotes
    #note: commit message can not have quotes or left and right brackets
    if ! [ $# -eq 0 ]
    then
        git add -A
        git commit -am "$*"
        git status
    fi
}
```

This allows me to type in my git commit messages like so:

```bash
gg my new commit
```

Which will transform to:

```bash
git commit -am "my new commit"
```

This is because `$*` representing the entire `my new commit` arguments is wrapped in quotes.

Also note the script checks that the number of arguments to the `gg` command is not zero using:

```bash
#number of arguments
$#
```

We could refer to the script text `gg` using the `$0`.

We could also refer to the `my` argument using `$1`, the `new` argument using `$2` and so on.

## permissions

```bash
echo $USER
sudo chown -R $USER:$USER /var/www
sudo chmod -R 755 /var/www

cd /var/www/myapp
sudo chown -R www-data: storage bootstrap
```

## grep

```bash
cd /usr/local/bin && la | grep code
cd /usr/local/bin && la | grep subl
cd /usr/local/bin && la | grep pstorm
```

## find

```bash
root@localhost:~# find / \( -iname "php.ini" -o -name "www.conf" \)
/etc/php/7.0/fpm/php.ini
/etc/php/7.0/fpm/pool.d/www.conf
/etc/php/7.0/cli/php.ini
```

## input processing

```bash
#non exported variable used in current shell
instr="a,b,c,d,e"
echo "${instr//,/$\n}"  ## Shell parameter expansion with commas replaced by newlins using echo
$ tr ',' '\n' <<<"$instr" ## Shell parameter expansion with commas replaced by newlins using tr

instr="a,b,c,d,e"
echo "${instr//,/ }"  ## Shell parameter expansion with commas replaced by spaces
```

## resources

[how-to-read-and-set-environmental-and-shell-variables-on-a-linux-vps](https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-a-linux-vps)

[the-first-two-statements-of-your-bash-script-should-be](https://ashishb.net/all/the-first-two-statements-of-your-bash-script-should-be)

[safer_bash_scripts_with_set_euxo_pipefail](https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail)

[allowing-for-errors-in-bash-when-you-have-set-e-defined](https://nickjanetakis.com/blog/allowing-for-errors-in-bash-when-you-have-set-e-defined)

[here-is-why-you-should-quote-your-variables-in-bash](https://nickjanetakis.com/blog/here-is-why-you-should-quote-your-variables-in-bash)

[bash-redirect-stderr-stdout](https://linuxize.com/post/bash-redirect-stderr-stdout)

[step-by-step-breakdown-of-dev-null](https://medium.com/@codenameyau/step-by-step-breakdown-of-dev-null-a0f516f53158)

[whats-difference-between-21-dev-null-and-21-dev-null](https://stackoverflow.com/questions/9390124/whats-difference-between-21-dev-null-and-21-dev-null)

[difference-between-dev-null-21-and-dev-null-dev-null](https://unix.stackexchange.com/questions/497207/difference-between-dev-null-21-and-dev-null-dev-null)
