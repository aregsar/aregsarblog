# installing composer php package manager

[installing composer php package manager](https://aregsar.com/blog/2019/installing-composer-php-package-manager)

## Introduction

In this post I will go over the instructions at `https://composer.org` on installing the Composer PHP package manager on MacOS and Linux operating systems.

The pre-requisite for installing Composer is that you need to have PHP installed.

Check out my posts for details on installing PHP on MacOS and Ubuntu linux:

[installing_php_on_macos](https://aregsar.com/blog/2019/installing_php_on_macos)

[installing php on ubuntu](https://aregsar.com/blog/2019/installing-php-on-ubuntu)

## removing older installation

You can run the following bash commands to remove an older installation, if it exists, to do a fresh install:

```bash
# rmove the composer CLI binary
cd /usr/local/bin
rm composer
# delete the .composer directory from your home directory
cd ~
rm -rf .composer
```

## Installing composer using bash commands

To get the bash commands to install composer you can check out:

`https://composer.org`

I will repeat the bash installation commands here with comments that describe how they work:

```bash
#download the composer-setup.php installation PHP script file to the current directory
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
#optional command that verifies the sha384 hash signature of the downloaded file.
#check composer.org/downloads to make sure you have the hash value of the latest release
php -r "if (hash_file('sha384', 'composer-setup.php') === '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
#run the composer-setup.php script using PHP and specify the installation directory and file name of the composer binary
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
# delete the composer-setup.php that was downloaded
php -r "unlink('composer-setup.php');"
```

## Note for PHP v7.3

If you have not updated the JIT compiler setting configuration for you PHP v7.3 installation, you need to update it before running composer. This needs to be done to avoid an error caused by the setting.

Here are the instructions to update the setting for the Ubuntu installation from the PHP installation on Ubuntu article

`sed -i 's/;pcre.jit=1/pcre.jit=0/g' /etc/php/7.3/cli/php.ini`
`sed -i 's/;pcre.jit=1/pcre.jit=0/g' /etc/php/7.3/fpm/php.ini`

The instructions for updating the setting for the macOS installation of PHP can be found in the PHP installation on macOS article:

`sed -i 's/;pcre.jit=1/pcre.jit=0/g' /7.3/php.ini`

## Installation directories

Ubuntu installation location

/root/.composer
/usr/local/bin/composer

macOS installation location

~/.composer
/usr/local/bin/composer

## Running composer from Docker container

Run in docker as interactive bash shell:

`docker run -ti --rm composer bash`

Run as a one shot command execution and exit:

`docker run --rm composer <composer command>`

where `<command>` is any composer command that you can run using a locally installed composer CLI.

## Conclusion

In this post I detailed the steps required to install Composer the PHP package manager.

Thanks for reading.