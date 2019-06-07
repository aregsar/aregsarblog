# Installing Composer PHP Package Manager

May 20, 2019 by [Areg Sarkissian](https://aregsar.com/about)

## Introduction

In this post I will go over the instructions at [https://getcomposer.org](https://getcomposer.org) on installing the Composer PHP package manager on macOS and Linux operating systems.

The pre-requisite for installing Composer is that you need to have PHP installed.

Check out my posts for details on installing PHP on MacOS and Ubuntu Linux:

[Installing PHP on macOS Redux](https://aregsar.com/blog/2019/installing-php-on-macos-redux)

[Installing PHP on Ubuntu](https://aregsar.com/blog/2019/installing-php-on-ubuntu)

## Removing existing Composer installation

You can run the following bash commands to remove an older installation, if it exists, before doing a fresh install:

```bash
# rmove the composer CLI binary
cd /usr/local/bin/
rm composer

# delete the .composer directory from your home directory
cd ~
rm -rf .composer
```

## Installing Composer

To install Composer, get the latest installer file download and execution commands (including the latest file hash value) from:

[https://getcomposer.org/download](https://getcomposer.org/download)

I will repeat the bash installation commands here with comments that describe how they work:

```bash
#download the composer-setup.php installation PHP script file to the current directory
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
#optional command that verifies the sha384 hash signature of the downloaded file.
#check https://getcomposer.org/download/ to make sure you have the latest hash value
php -r "if (hash_file('sha384', 'composer-setup.php') === '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
#run the composer-setup.php script using PHP and specify the installation directory and file name of the composer binary
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
# delete the composer-setup.php installer file that was downloaded
php -r "unlink('composer-setup.php');"
```

## Fixing Composer JIT compiler error for PHP v7.3

If you have not updated the JIT compiler setting configuration for you PHP v7.3 installation, you need to update it before running composer.

This needs to be done to avoid the JIT compiler setting error: `preg_match(): JIT compilation failed: no more memory` when running Composer commands.

For macOS you can run the following command to disable the JIT compiler:

`sed -i '' 's/;pcre.jit=1/pcre.jit=0/g' /usr/local/etc/php/7.3/php.ini`

For Ubuntu you can run the following commands to disable the JIT compiler:

`sed -i 's/;pcre.jit=1/pcre.jit=0/g' /etc/php/7.3/cli/php.ini`

`sed -i 's/;pcre.jit=1/pcre.jit=0/g' /etc/php/7.3/fpm/php.ini`

## The Composer installation directories

The composer executable is installed at:

`/usr/local/bin/composer`

The directory where global packages will be installed was created under the home directory:

`~/.composer`

## Running Composer from Docker

Sometimes as an alternative to installing composer we can just execute some composer commands by using the official Composer docker container.

We can run composer interactively from within the docker container using a bash shell:

`docker run -ti --rm composer bash`

Or we can run it as a one shot command execution:

`docker run --rm composer <composer command>`

Where `<command>` is any composer command that you can run using a locally installed composer CLI.

With one shot execution the container exits right after executing the command.

> Note: For executing Composer scripts that execute PHP code, you may need to install additional PHP extensions in the container using your own docker file that extends the official Composer Docker image.

## Conclusion

In this post I detailed the steps required to install the Composer PHP package manager on macOS and Ubuntu Linux.

Thanks for reading.
