# Installing PHP on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Using Homebrew to install PHP

> Instructions for installing Homebrew can be found [here](https://brew.sh)

Before installing PHP we need to update brew itself.

```bash
brew update
```

## installing PHP for the first time

```bash
brew install php
```

Check the version:

```bash
php -v
#or
php --version
```

> Make sure the paths `/usr/local/bin` and `/usr/local/sbin` are in the system path. The path `/usr/local/bin` has the `php` symlink that points to the actual php installation directory for the version of php that is installed. The path `/usr/local/sbin` has the `php-fpm` php server symlink. I generally have `/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin` in my $PATH variable.

You can show all installed versions of PHP by running:

```bash
brew list | grep php
```

## Upgrading to the latest version

```bash
brew upgrade php
#check the version
php -v
```

This will install the latest version of PHP and update the `php` symlink in `usr/local/bin` to point to this new PHP installation.

## Installing and using multiple versions

Instead of the normal `brew install php` and `brew upgrade php` commands we can use the `shivammathur/php` brew tap to install multiple versions of PHP and switch between them:

See the available version at [shivammathur/homebrew-php](https://github.com/shivammathur/homebrew-php)

First add the brew tap for the plugin:

```bash
brew tap shivammathur/php
```

Next install the desired version of PHP:

```bash
brew install shivammathur/php/php@8.0
```

As many versions as you like can be installed with the above command.

Finally set the version we want as the default:

```bash
brew unlink php && brew link --overwrite --force php@8.0
#check the version
php -v
```

To change the version simply switch the active php version to another installed version.

For example:

```bash
brew link --overwrite --force php@7.4
#check the version
php -v
```

A bash alias to switch versions:

```bash
#usage:
#switch to version 8.0
#phpver 8.0
#swich to version 7.4
#phpver 7.4
phpver() {
brew unlink php && brew link --force --overwrite php@$1
}
```

> Note: Make sure you independently install PHP extensions for each active version of PHP. Same goes for changes to configuration files.

## Getting information about PHP

### Getting information about the installation

Display the PHP info page in the console for the active PHP version:

```bash
php -i
```

Another way to get the same result is via:

```bash
php -r "phpinfo();"
```

Run the following to only list the installed PHP extensions for the active PHP version:

```bash
php -m
```

### Getting the configuration file locations

Get information on PHP configuration file directories for the active version of PHP.

The configuration files are located at the `/usr/local/etc/php/<version>/` directory.

This command prints out the locations:

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

### PHP installation locations

Executable symlinks and actual locations for php, php-fpm and pecl that are installed.

The pecl tool can be used to install php extensions

/usr/local/bin/php -> /usr/local/Cellar/php/8.0.1_1/bin/php

/usr/local/bin/pecl -> /usr/local/Cellar/php/8.0.1_1/bin/pecl

/usr/local/sbin/php-fpm -> /usr/local/Cellar/php/8.0.1_1/sbin/php-fpm

Run the following command to show the location of the php cli symlink

which php
aregsarkissian@Aregs-MacBook-Pro sbin % which php-fpm
/usr/local/sbin/php-fpm
aregsarkissian@Aregs-MacBook-Pro sbin % which pecl
/usr/local/bin/pecl
aregsarkissian@Aregs-MacBook-Pro sbin % which php
/usr/local/bin/php

Note: If the latest version of PHP is v7.3, then running brew install php or brew install php@7.3 will result in the same installation directory. That is, the base directory would still be /usr/local/Cellar/php/7.3.5/. So both commands are effectively the same. However if the latest version is v7.3 and we install an older version of PHP, say for example brew install php@7.2, then the base installation directory will be /usr/local/Cellar/php@7.2/7.2.18/ which as you can see includes the @7.2 version and the PHP 7.2 release version number in the path.
