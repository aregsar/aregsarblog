# Laravel Development MacBook Setup

October 15, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I detail the PHP and Node packages that I install globally on my Mac for local Laravel development.

Prior to this post I have the Homebrew package manager installed as detailed at [my-macbook-developer-setup/](https://aregsar.com/blog/2020/my-macbook-developer-setup). The Homebrew `brew` command is used in this post to install CLI tools and server binaries.

Also after I setup my Mac for PHP and Laravel development, I have a related post on how to configure VSCode for PHP and Laravel development at [my-vscode-php-development-setup](https://aregsar.com/blog/2020/my-vscode-php-development-setup).

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

added php binaries here
/usr/local/Cellar/php/7.4.9/

php cli binary
/usr/local/Cellar/php/7.4.9/bin/php

pecl cli binary
/usr/local/Cellar/php/7.4.9/bin/pecl

php-fpm binary
/usr/local/Cellar/php/7.4.9/sbin/php-fpm

added config files here
/usr/local/etc/php/7.4/

added /usr/local/bin/php symlink to /usr/local/Cellar/php/7.4.9/bin/php

added /usr/local/bin/pecl symlink to /usr/local/Cellar/php/7.4.9/bin/pecl

added /usr/local/sbin/php-fpm symlink to /usr/local/Cellar/php/7.4.9/sbin/php-fpm

## Install xdebug and redis php extensions

```bash
#install xdebug extension
pecl install --force xdebug
```

installed xdebug in the following locations
/usr/local/Cellar/php/7.4.9/pecl/20190902/xdebug.so
/usr/local/lib/php/pecl/20190902/xdebug.so

```bash
#install php-redis extension
pecl install --force redis
```

installed redis in the following locations
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
```

installs valet cli in the following directory
~/.composer/vendor/laravel/valet

adds symlink to valet cli so need to add ~/.composer/vendor/bin to path
~/.composer/vendor/bin/valet -> ~/.composer/vendor/laravel/valet/valet

Add the following lines to the ~/.zshrc file:

```ini
#this is where the symlinks for valet and other composer global package binaries are installed
#export PATH="~/.composer/vendor/bin:$PATH"
export COMPOSER_GLOBAL_PACKAGE_HOME=~/.composer/vendor/bin
export PATH=$COMPOSER_GLOBAL_PACKAGE_HOME:$PATH
```

```bash
#check valet version
valet --version
```

Now that Valet is installed and accessible globally, run the valet commands to configure Valet:

install valet to have all valet commands available:

```bash
valet install
```

You should see similar to the following output:

```bash
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
```

Run valet trust so password is not required for valet commands:

```bash
valet trust
```

You should see the following output:

```bash
Sudoers entries have been added for Brew and Valet.
```

make valet use current php version:

```bash
valet use php
```

You should see the following output:

```bash
Stopping php...
Unlinking current version: php
Linking new version: php
Updating PHP configuration...
Restarting php...
Restarting nginx...
Valet is now using php@7.4.
```

## Valet Park Laravel projects

To be able to server all Laravel projects using `myprojectname.test` URL we need to park all the projects in their parent directory.

```bash
cd directory_where_all_my_laravel_projects_reside
#park all the projects
valet park
#list parked projects
valet parked
```

You should see the output below:

```bash
+-------+-----+-------------------+------------------------------------------------------+
| Site  | SSL | URL               | Path                                                 |
+-------+-----+-------------------+------------------------------------------------------+
| blog  |     | http://blog.test  | /directory_where_all_my_laravel_projects_reside/blog |
| todo  |     | http://todo.test  | /directory_where_all_my_laravel_projects_reside/todo |
+-------+-----+-------------------+------------------------------------------------------+
```
