# Laravel Development MacBook Setup

October 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I detail the PHP and Node packages that I install globally on my Mac for local Laravel development.

Prior to post I have the Homebrew package manager installed as detailed at [my-vscode-php-development-setup](https://aregsar.com/blog/2020/my-vscode-php-development-setup).

## Install Node Version Manager

To compile assets and install client side packages we will need node, npm and npx.

So the first thing I install is the node version manager, nvm:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

## Install php, pecl and php-fpm

```bash
brew install php
```

```bash
#show installed php versions
brew list | grep php
```

#added php binaries here
/usr/local/Cellar/php/7.4.9/

#php cli binary
/usr/local/Cellar/php/7.4.9/bin/php

#pecl cli binary
/usr/local/Cellar/php/7.4.9/bin/pecl

#php-fpm binary
/usr/local/Cellar/php/7.4.9/sbin/php-fpm

#added config files here
/usr/local/etc/php/7.4/

added /usr/local/bin/php symlink to /usr/local/Cellar/php/7.4.9/bin/php

added /usr/local/bin/pecl symlink to /usr/local/Cellar/php/7.4.9/bin/pecl

added /usr/local/sbin/php-fpm symlink to /usr/local/Cellar/php/7.4.9/sbin/php-fpm

## Install xdebug and redis php extensions

```bash
#install xdebug extension
pecl install --force xdebug
```

#installed xdebug in the following locations
/usr/local/Cellar/php/7.4.9/pecl/20190902/xdebug.so
/usr/local/lib/php/pecl/20190902/xdebug.so

```bash
#install php-redis extension
pecl install --force redis
```

#installed redis in the following locations
/usr/local/Cellar/php/7.4.9/pecl/20190902/redis.so
/usr/local/lib/php/pecl/20190902/redis.so

```bash
#list installed php extenstions
#should show xdebug and redis
php -m
```

```bash
#show content of php.ini file
cat /usr/local/etc/php/7.4/php.ini
```

```ini
#should see following at top of the file:
extension="redis.so"
zend_extension="xdebug.so"
```

```bash
#use pecl to shoe extension versions
pecl list
#should show the following:
redis 5.3.1 stable
xdebug 2.9.6 stable
```

## Install Composer PHP package manager

```bash
cd

#download the composer-setup.php installation PHP script file to the current directory
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

#optional command that verifies the sha384 hash signature of the downloaded file.
#check https://getcomposer.org/download/ to make sure you have the latest hash value
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e5325b19b381bfd88ce90a5ddb7823406b2a38cff6bb704b0acc289a09c8128d4a8ce2bbafcd1fcbdc38666422fe2806') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

#should respond with: Installer verified

#run the composer-setup.php script using PHP and specify the installation directory and file name of the composer binary
php composer-setup.php --install-dir=/usr/local/bin --filename=composer

# delete the composer-setup.php installer file that was downloaded

php -r "unlink('composer-setup.php');"

#show installed version
composer --version
```

## Install Laravel Valet

```bash
composer global require laravel/valet

#installs valet cli in the following directory
~/.composer/vendor/laravel/valet

#adds symlink to valet cli so need to add ~/.composer/vendor/bin to path
~/.composer/vendor/bin/valet -> ~/.composer/vendor/laravel/valet/valet

#check valet version
valet --version
```

Now that Valet is installed and accessible globally, run the valet commands:

```bash
#install valet to have all valet commands available
valet install
```

Stopping nginx...
Installing nginx...
[nginx] is not installed, installing it now via Brew... 🍻
Installing nginx configuration...
Installing nginx directory...
Updating PHP configuration...
Restarting php...
Installing dnsmasq...
[dnsmasq] is not installed, installing it now via Brew... 🍻
Updating Dnsmasq configuration...
Restarting dnsmasq...
Valet is configured to serve for TLD [.test]
Restarting nginx...
Valet installed successfully!

```bash
#run valet trust so password is not required for valet commands
valet trust
```

Sudoers entries have been added for Brew and Valet.

```bash
#make valet use current php version
valet use php
```

Stopping php...
Unlinking current version: php
Linking new version: php
Updating PHP configuration...
Restarting php...
Restarting nginx...
Valet is now using php@7.4.

## Valet Park Laravel projects

```bash
cd code/laravel
valet park
valet parked
```
