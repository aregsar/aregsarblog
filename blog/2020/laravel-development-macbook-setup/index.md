# laravel development macbook setup

[laravel development macbook setup](https://aregsar.com/blog/2020/laravel-development-macbook-setup)

=============
nvm install
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

---

php pecl composer valet (nginx and dnsmasq) install

export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin

## install php, pecl and php-fpm

brew install php

#show installed php versions
brew list | grep php

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

## install php extensions (xdebug and php-redis)

#install xdebug extension
pecl install --force xdebug

#installed xdebug in the following locations
/usr/local/Cellar/php/7.4.9/pecl/20190902/xdebug.so
/usr/local/lib/php/pecl/20190902/xdebug.so

#install php-redis extension
pecl install --force redis

#installed xdebug in the following locations
/usr/local/Cellar/php/7.4.9/pecl/20190902/redis.so
/usr/local/lib/php/pecl/20190902/redis.so

#list installed php extenstions
#should show xdebug and redis
php -m

#show content of php.ini file
cat /usr/local/etc/php/7.4/php.ini
#should see following at top of the file:
extension="redis.so"
zend_extension="xdebug.so"

#use pecl to shoe extension versions
pecl list
#should show the following:
redis 5.3.1 stable
xdebug 2.9.6 stable

## install composer

#go to home directory
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

## install laravel valet

composer global require laravel/valet

#installs valet cli in the following directory
~/.composer/vendor/laravel/valet

# adds symlink to valet cli

# so need to add ~/.composer/vendor/bin to path

~/.composer/vendor/bin/valet -> ~/.composer/vendor/laravel/valet/valet

# check valet version

valet --version

#install valet to have all valet commands available
valet install

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

#run valet trust so password is not required for valet commands
valet trust
Sudoers entries have been added for Brew and Valet.

#make valet use current php version
valet use php
Stopping php...
Unlinking current version: php
Linking new version: php
Updating PHP configuration...
Restarting php...
Restarting nginx...
Valet is now using php@7.4.

cd code/laravel
valet park
valet parked
