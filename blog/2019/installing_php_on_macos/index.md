# installing_php_on_macos

[installing_php_on_macos](https://aregsar.com/blog/2019/installing_php_on_macos)

## Introduction

In this post I will detail the installation steps to install PHP and PHP-FPM on a Mac using the Homebrew package manager.

## Installing homebrew

Before we install PHP we need to install the homebrew package manager, here after referenced simply as brew.

The installations for installing brew are at https://brew.sh/. You simply run the following bash statement:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

The bash script downloads and runs a Ruby language script using the default Ruby installation on your Mac.

## Updating Homebrew

If you already had brew installed, make sure to update it before installing PHP by running `brew update`

> Note: Be careful to not run __brew upgade__ instead, which updates all the packages that brew has installed on your system.

Run `brew doctor` afterwards to make sure brew does not see any issues with current installed packages.

## Upgrading to the latest PHP homebrew installation

If you already have a PHP version installed, you can run `brew upgrade php` to upgrade to the latest released PHP version.

You will notice that we dont specify the php version in the brew command. That is because the latest version of the installed software does not use the version number to name the installation directories.

> Note: Even if you are on the latest minor release of PHP, running `brew upgrade php` will ensure that all the latest patches are applied.

## Upgrading a specific PHP homebrew installation

If you have multiple php versions installed, you can upgrade the older versions with the latest patches by running `brew upgrate php@<version>`.

When installing older versions of software, brew uses the version number to name the installion directories so we have to use the version number in the command to tell brew which version we want to upgrade.

So for instance if you have the latest PHP version (7.3) installed and you also have PHP version 7.2 installed, you can run `brew upgrate php` to upgrade the latest version and run `brew upgrade php@7.2` to upgrade the older version 7.2.

## Removing installed versions of PHP

To do a fresh install, we will remove any existing homebrew PHP installations.
The following is the bash commands I use to clean out my PHP installation directories:

```bash
brew list | grep php | xargs brew uninstall --force
rm -rf /usr/local/etc/php
rm -rf /usr/local/lib/php
rm ~/Library/LaunchAgents/homebrew.mxcl.php*
sudo rm /Library/LaunchDaemons/homebrew.mxcl.php*
brew untap homebrew/php
brew cleanup -s
brew untap homebrew/homebrew-dupes
```

At this point the following directories should not exist:

+ /usr/local/Cellar/php/
+ /usr/local/lib/php/
+ /usr/local/opt/php/
+ /usr/local/etc/php/

and the `/usr/local/bin/php` symlink should not exits either

## Installing the latest PHP version

Start with updating brew itself

`brew update`

Now install the latest PHP version using the command:

`brew install php`

After running this command latest version of php (7.3 at this time ) will be installed. Run the following command to verify.

`php -v`

Run the following to see all information and configuration settings regarding the installed version:

`php -i`

Run the following to list the installed PHP extensions:

`php -m`

Run this command to see the symlink created taht points to the installed php CLI binary:

`which php`

The following directories are the relevent installation

## The installation directories

Homebrew installes the latest php binaries:

`/usr/local/Cellar/php/7.3.5/bin`

So the `/usr/local/bin/php` symlink points to `/usr/local/Cellar/php/7.3.5/bin/php`

This is the php CLI that runs when we type `php` on the command line.

We just need to make sure we have `/usr/local/bin` in our PATH envoronment variable.

Also the `pecl` CLI for installing php extensions is located at:

`/usr/local/Cellar/php/7.3.5/bin/pecl`.

Similarly homebrew installs the system binaries for php in:

`/usr/local/Cellar/php/7.3.5/sbin`.

Therefore the `php-fpm` server is installed at:

`/usr/local/Cellar/php/7.3.5/sbin/php-fpm`.

To be able to access the php cli and php-fpm server binaries, my PATH variable starts with:

`export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin`

We can configure php-fpm to auto-start by running the brew command:

`sudo brew services start php`

Then we can check to see if is running using the brew command:

`brew services list`

## PHP ini and configuration files

The brew installation puts the configuration files for installed php versions in the directory:

`/usr/local/etc/php/<version>/`

So the brew installation for the latest php version on my machine put all configuration files for the php cli and php-fpm in:

`/usr/local/etc/php/7.3/`

So for instance the php.ini file is located at:

`/usr/local/etc/php/7.3/php.ini`

and the php-fpm.conf file at:

`/usr/local/etc/php/7.3/php-fpm.conf`

## Installing PHP extensions using PECL

The pecl cli installed by homebrew can be used to install further php extensions that we requires by running the command `pecl install <extension>`.

For instance we can install the xdebug extension by typing:

`pecl install xdebug`

and we can list the pecl installed php extensions by running:

`pecl list` or `php -m`

The directory where the php extensions are installed depends on the php version.

Each installed php version will have its own php.ini file within which the php extensions directory is specified. The setting in the php.ini file that does this is:

`extension_dir = "/usr/local/lib/php/pecl/20180731"`

So after we run `pecl install xdebug` we can see that xdebug is installed at:

`/usr/local/lib/php/pecl/20180731/xdebug.so`

It may appear that the extensions are also installed in `/usr/local/Cellar/php/7.3.5/pecl/20180731/` as well, but that is because `/usr/local/Cellar/php/7.3.5/pecl/` is a symlink to `/usr/local/lib/php/pecl/`.

## How php detects pecl installed extensions

In order for php to detect pecl installed extensions, we must declare them in the php.ini file. The pecl install command adds the extension to the php.ini file for us.

If we look inside the php.ini file we will see that xdebug is actually installed as a special type of extension that is a zend extension:

`zend_extension="xdebug.so"`

Normally php extensions are installed as standard php extensions and will be reference in the php.ini file as:

`extension="<php-extension>.so"`

So if we run `pecl install memcached` for instance, the extension will be installed at `/usr/local/lib/php/pecl/20180731/memcached.so` and `extension="memcached.so"` will be added to the php.ini file.

> Note: when installing the memcached extension on my machine, a cached version of a previous install was not cleaned up so pecl was detecting memcached was installed when in fact it was not. To solve this I did a `pecl uninstall memcached` and then I re-installed the extension with `pecl install memcached` and that resolved the issue.

> Note: if after installing the xdebug extension, if it is not detected and listed under [Zend Modules] when running `php -m` make sure that `zend_extension="xdebug.so"` is added to the top of the php.ini file, and if any standard installed php extension is not listed under [PHP Modules] when running `php -m` make sure that the `extension="<php-extension>.so"` for that extension name is added to the top of the php.ini file.

Its good practice to run `pecl list` and check the pecl extensions directory after running `pecl install` to make sure the extension is indeed installed. Then you should run `php -m` to make sure the extension is detected by php and if not, make sure that the extension setting for each installed extenion is added to the php.ini file.

## installing older versions of php

The installation steps for installing older versions of php is the same except you specify the version number in the install command

Once installed the installations will be in the equivalent directories under the installed version directory.

So if we install `brew install php@7.2` the installed directories will be under:

`/usr/local/Cellar/php@7.2/7.2.18/`

as apposed to `/usr/local/Cellar/php/7.3.5/` for the latest (7.3) version.

so for v 7.2 the php, pecl and php-fpm binaries will be at:

`/usr/local/Cellar/php@7.2/7.2.18/bin/php`
`/usr/local/Cellar/php@7.2/7.2.18/bin/pecl`
`/usr/local/Cellar/php@7.2/7.2.18/sbin/`

and the configuration files will be installed in:

`/usr/local/etc/php/7.2/`

where the `/usr/local/etc/php/7.2/php.ini` file will specify the pecl installation directory setting for php extensions as:

`extension_dir = "/usr/local/lib/php/pecl/20170718"`

> Note: the extensions directory uses the release date instead of the version number.

Now when we run the v 7.2 pecl command `/usr/local/Cellar/php@7.2/7.2.18/bin/pecl install xdebug` for instance, the xdebug.so extension file will be installed in the `/usr/local/lib/php/pecl/20170718` directory.

> Note if we want to switch to using the older version of php for our projects directly using the `php` command, we have to

## Conclusion

I listed the steps to take to install homebrew and then use homebrew to install php