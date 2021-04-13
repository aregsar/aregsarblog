# Installing PHP on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

## Using Homebrew to install PHP

> Instructions for installing Homebrew can be found [here](https://brew.sh)

Before installing or upgrading PHP we need to update brew itself.

```bash
brew update
```

## Installing PHP for the first time

Ensure the paths `/usr/local/bin` and `/usr/local/sbin` are in the system path.

> After the installation the path `/usr/local/bin` will hold the `php` symlink that points to the actual php installation directory for the version of php that is installed. Similarly, the path `/usr/local/sbin` will hold the `php-fpm` php application server symlink.

Run the brew command to install php:

```bash
brew install php
```

Show the installed php version:

```bash
php -v
```

List all installed versions of PHP:

```bash
brew list | grep php
```

## Upgrading to the latest version

Run the brew command to upgrade php:

```bash
brew upgrade php
```

This will install the latest version of PHP replacing the old installation.

Show the installed php version:

```bash
php -v
```

List all installed versions of PHP by running:

```bash
brew list | grep php
```

## Installing and switching between multiple versions of PHP

Instead of the standard `brew install php` and `brew upgrade php` commands shown above, we can use the `shivammathur/php` brew tap to install multiple versions of PHP and switch between them.

The [shivammathur/homebrew-php](https://github.com/shivammathur/homebrew-php) page lists all the available PHP versions.

First add the brew tap for the plugin:

```bash
brew tap shivammathur/php
```

Next install the desired version of PHP.

```bash
brew install shivammathur/php/php@8.0
```

The command above installs PHP version 8.0.

Any number of versions can be installed using the same command.

List all installed versions of PHP by running:

```bash
brew list | grep php
```

Now we can set the version we want as the active version:

```bash
brew link --overwrite --force php@8.0
#brew unlink php && brew link --overwrite --force php@8.0
```

Show the active php version:

```bash
php -v
```

To change the active version simply use the same command to link to the version we want to change to.

```bash
brew link --overwrite --force php@7.4
```

Show the active php version:

```bash
php -v
```

## Making it easy to switch between PHP versions

Here is a bash alias to switch between versions:

```bash
phpver() {
    brew link --force --overwrite php@$1
}
```

Once we add the bash alias to our shell profile we can use it as shown:

```bash
#switch to version 7.4
phpver 7.4
#switch to version 8.0
phpver 8.0
```

Another tool named [PHP Monitor](https://github.com/nicoverbruggen/phpmon), allows you to switch versions straight from the MacOS menu bar. However it only works if you are using [Laravel Valet](https://github.com/laravel/valet) running on MacOS.

## Installing PHP extensions for multiple installed PHP versions

We need to make sure that we individually install PHP extensions for each active version of PHP.

We can do so by first by switching to the version that we want to install an extension for, then installing the extensions for that version.

Same goes for making changes to the configuration files.

See [Installing PHP Extensions](https://aregsar.com/about) for details of installing php extensions using the pecl cli.

The pecl cli is the php extension installer.

## Running a local PHP development Server

We can run a PHP development server using the following command:

```bash
php serve -S locahost:8080
```

Here we are running the server on post 8080.

## Getting information about PHP

### Getting information about the PHP installation

Display the PHP info page in the console for the active PHP version:

```bash
php -i
```

Another way to get the same result is by executing php code inline:

```bash
php -r "phpinfo();"
```

Run the following, to list the installed PHP extensions for the active PHP version:

```bash
php -m
```

### Getting the PHP configuration file locations

Get information on PHP configuration file directories for the active version of PHP.

The configuration files are located at the `/usr/local/etc/php/<version>/` directory.

The command below prints out the locations:

```bash
php --ini
```

The response of the above command is shown below for version 8.0:

```bash
Configuration File (php.ini) Path: /usr/local/etc/php/8.0
Loaded Configuration File: /usr/local/etc/php/8.0/php.ini
Scan for additional .ini files in: /usr/local/etc/php/8.0/conf.d
Additional .ini files parsed: /usr/local/etc/php/8.0/conf.d/ext-opcache.ini
```

The `php.ini` file contains the configuration for the active php installation. The `conf.d` file contains the configuration for the `php-fpm` php process manager for running a php application server.

### PHP installation locations

Run the following commands to show the location of the php, php-fpm and pecl symlinks:

```bash
which php
which pecl
which php-fpm
```

The symlinks for php 8.0 installation are shown below:

```bash
/usr/local/bin/php -> /usr/local/Cellar/php/8.0.1_1/bin/php

/usr/local/bin/pecl -> /usr/local/Cellar/php/8.0.1_1/bin/pecl

/usr/local/sbin/php-fpm -> /usr/local/Cellar/php/8.0.1_1/sbin/php-fpm
```
