# Installing PHP on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Using Homebrew to install PHP

Insructions for installing Homebrew can be found [here](https://homebrew)

Before installing PHP we need to update brew itself.

```bash
brew update
```

## installing PHP for the first time

```bash
brew install php
#check the version
php -v
```

> Make sure the path `/usr/bin/local` is in the system path and precedes the path of the default PHP installation that came with your MacOS. The path `/usr/bin/local` has the `php` symlink that points to the actual php installation directory for the version of php that was just installed.

## Upgrading to the latest version

```bash
brew upgrade php
#check the version
php -v
```

This will install the latest version of PHP and update the `php` symlink in `usr/local/bin` to point to this new PHP installation.

## Installing and using multiple versions

Instead of the normal `brew install php` and `brew upgrade php` commands we can use the `shivammathur/php` brew tap to install and switch between multiple versions of PHP:

First add the tap for the plugin:

```bash
brew tap shivammathur/php
```

Next install the desired version:

```bash
brew install shivammathur/php/php@8.0
```

As many versions as you like can be installed withe the above command.

Finally set the version we want as the default:

```bash
brew link --overwrite --force php@8.0
#check the version
php -v
```

## Getting information about PHP

Display the PHP info page in the console:

```bash
php -r "phpinfo();"
```

Get information on PHP installation directory of the current installed version of PHP and the paths for the php.ini and any other .ini configuration files:

```bash
php --ini
```

The response of the above command is shown below:

```bash
Configuration File (php.ini) Path: /usr/local/etc/php/7.4</hljs>
Loaded Configuration File: /usr/local/etc/php/7.4/php.ini
Scan for additional .ini files in: /usr/local/etc/php/7.4/conf.d
Additional .ini files parsed: /usr/local/etc/php/7.4/conf.d/ext-opcache.ini,
/usr/local/etc/php/7.4/conf.d/php-memory-limits.ini
```
