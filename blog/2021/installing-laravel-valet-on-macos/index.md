# Installing Laravel Valet on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Prerequisites

install homebrew
install php (should install php-fpm)
install composer

add /usr/local/bin to your system path

Valet will install dnsmasq and nginx using brew

## Installing and configuring valet

composer global require laravel/valet

valet --version

valet install

valet trust

valet use php

valet start

## updating and reintstalling

valet stop

valet uninstall --force

composer global update

valet --version

valet install

valet trust

valet use php

valet start

Note `valet start` will also start the dnsmaq and nginx services

## Resetting valet to use a new php version

When we swich php versions, the php command point to the new php version.
Just to be safe we may have to inform valet about this by doing the following:

valet stop

#php now point to a different version of php
valet use php

valet start

Also repeat the same steps anytime you install a new PHP extension.

## uninstalling and removing valet

valet stop

valet uninstall --force

composer global remove laravel/valet

sudo rm -rf ~/.config/valet

rm /usr/local/bin/valet

Additionally can stop and uninstall the valet installed dnsmasq and nginx services
sudo brew services stop nginx
sudo brew services stop dnsmasq
brew uninstall dnsmasq
brew uninstall nginx

## Issues with composer when running composer update

composer global update is run anytime run composer global remove or composer global require
we can also directly run composer global update

In either case sometimes composer global update fails with version conflict warnings

If this happens when trying to install, update or remove valet you may have to delete your composer.lock file and the vendor directory in the composer home directory ~/.composer

Then manually remove the valet package from composer.json and run:

```bash
composer global require laravel/valet
```

If you are unable to install due to composer errors, you may need to delet the composer.json and reinstall all the
installed global packages including laravel valet.

See the composer post for handling version conflict error details

## Starting, stoping and restarting

Starting stoping and restarting valet also starts stops and restarts the dnsmasq and nginx services

valet start

valet stop

valet restart

## adding websites

linking all projects within a directory

cd ~/myprojects

valet park

linking a single project

cd ~/myspecialprojects/myapp

valet link

Each project will be served locally using the project name and .test domain

http://<project_name>.test

The site will also respond to any subdomain

http://<subdomain>.<project_name>.test

we can show the linked sites

valet links

Also if we check the file `~/.config/valet/config.json` we will see that `~/myspecialprojects` directory is added to the paths property.

We can also the tld (top level domain) property is set to test which allows serving using .test domain. We can change this and run `restart valet` to serve using a different domain. (just make sure it does not conflict with a real domain since dnsmasq will redirect it to the local nginx server)

```bash
cat ~/.config/valet/config.json
```

```json
{
  "tld": "test",
  "paths": ["/Users/aregsarkissian/dev"]
}
```

## Removing websites

We can remove individual sites within a parked directory by entering the project and using the unlink command:

cd ~/myprojects/myotherapp

valet unlink

To unlink all projects within a parked directory enter the directory and use the forget command

cd ~/myprojects
valet forget

## Valet executable

requiring valet globally installs the valet executable file at:

~/.composer/vendor/laravel/valet/valet

The installation also adds the symlinks:

~/.composer/vendor/bin/valet -> ~/.composer/vendor/laravel/valet/valet

Usually we add ~/.composer/vendor/bin to our system $PATH variable to be able globally run executables symlinks that composer
sets up in `~/.composer/vendor/bin`.

However Valet installs a symlink in `/usr/local/bin` that points to the symlink in `~/.composer/vendor/bin`.

/usr/local/bin/valet -> /Users/aregsarkissian/.composer/vendor/laravel/valet/valet

So as long as we have `/usr/local/bin` in our system $PATH we will be able to at least run the valet command globally.

## Valet configuration files

When we run the `valet install` command, valet creates the configiration directory `~/.config/valet/`

Important configuration files and directroies

The `~/.config/valet/config.json` file lists the directories that hosts the Laravel projects.

Here is a list of other notable configuration locations:

```bash
~/.config/valet/config.json

~/.config/valet/Sites/

~/.config/valet/dnsmasq.d/

~/.config/valet/Nginx/

/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf

~/.config/valet/valet.sock
```

for a full list and descriptions see laravel valet documentation

https://laravel.com/docs/8.x/valet

## How valet works

the `valet install` command will use Homebrew to download and install Nginx and Dnsmasq servers and configure Valet to use them. Installing Nginx and Dnsmasq will add the nginx and dnsmasq services on your system and start them up.

run `brew list` to see if nginx and dnsmaq are installed

we can manually install and start them if for some reason valet is unable to install them

brew install dnsmasq
brew install nginx
sudo brew services start dnsmasq
sudo brew services start nginx
