# Installing PHP on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Using Homebrew to install PHP

> Instructions for installing Homebrew can be found [here](https://brew.sh)

Before installing PHP we need to update brew itself.

```bash
brew update
```

## Installing PHP for the first time

Ensure the paths `/usr/local/bin` and `/usr/local/sbin` are in the system path.

> After the installation the path `/usr/local/bin` will hold the `php` symlink that points to the actual php installation directory for the version of php that is installed. Similarly, the path `/usr/local/sbin` hold the `php-fpm` php server symlink.

Run the brew command to install php:

```bash
brew install php
```

Check the installed php version:

```bash
php -v
```

Show location of the php cli symlink:

```bash
which php
```

List all installed versions of PHP by running:

```bash
brew list | grep php
```

## Upgrading to the latest version

Run the brew command to upgrade php:

```bash
brew upgrade php
```

This will install the latest version of PHP in a new directory named with the new PHP version number and update the `php` symlink in `usr/local/bin` to point to this new PHP installation directory.

Check the installed php version:

```bash
php -v
```

Show location of the php cli symlink:

```bash
which php
```

List all installed versions of PHP by running:

```bash
brew list | grep php
```

## Installing and switching between multiple versions of PHP

Instead of the standard `brew install php` and `brew upgrade php` commands we can use the `shivammathur/php` brew tap to install multiple versions of PHP and switch between them:

[shivammathur/homebrew-php](https://github.com/shivammathur/homebrew-php) lists all the available PHP.

First add the brew tap for the plugin:

```bash
brew tap shivammathur/php
```

Next install the desired version of PHP.

```bash
brew install shivammathur/php/php@8.0
```

As many versions as you like can be installed with the above command.

Now we can set the version we want as the active version:

```bash
brew link --overwrite --force php@8.0
#brew unlink php && brew link --overwrite --force php@8.0
```

To change the active version simply use the same command to link to the version we want.

```bash
brew link --overwrite --force php@7.4
```

I have create a bash alias to switch between versions:

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

## Installing PHP extensions for multiple installed PHP versions

We need to make sure that we install PHP extensions for each active version of PHP.

We can do so by first making a specific version the active version using the brew link command.

Then installing the extensions for that version.

Same goes for changes to making changes to the configuration files.

## Getting information about PHP

### Getting information about the installation

Display the PHP info page in the console for the active PHP version:

```bash
php -i
```

Another way to get the same result is by executing php code inline:

```bash
php -r "phpinfo();"
```

Run the following to list the installed PHP extensions for the active PHP version:

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
