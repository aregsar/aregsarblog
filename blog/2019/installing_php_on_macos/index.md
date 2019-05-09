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

## Removing all brew installed versions of PHP

To do a fresh install, we will first remove any existing homebrew PHP installations.
The following is the bash commands I use to clean out my PHP installation directories:

```bash
brew list | grep php | xargs brew uninstall --force
rm -rf /usr/local/lib/php
rm -rf /usr/local/etc/php
rm ~/Library/LaunchAgents/homebrew.mxcl.php*
sudo rm /Library/LaunchDaemons/homebrew.mxcl.php*
brew untap homebrew/php
brew cleanup -s
brew untap homebrew/homebrew-dupes
```

The first line in the above bash statements goes through all installed versions and removes them all.

> Note: some of the commands above may not be required, but running them all will ensure a clean uninstall.

## Installing the latest PHP version

Now install the latest PHP version using the command:

`brew install php`

After running this command latest version of php (7.3 as of this time) will be installed.

You can run the following command to verify the current php version.

`php -v`

Run the following to see all information and configuration settings regarding the installed version:

`php -i`

Run the following to list the installed PHP extensions:

`php -m`

Run this command to see the symlink created that points to the installed php CLI binary:

`which php`

You can show all installed versions of php by running:

`brew list | grep php`

The result of this command that does not have an extension is the latest version.
On my machine I see two results the first is for the latest php v7.3 and the other is obviously php v7.2:

```bash
php #latest version 7.3
php@7.2
```

## The installation directories

For php v7.3, which is the latest version of php as of this article date, when we run the `brew install php` command, the php files will be installed under the base path of:

`/usr/local/Cellar/php/7.3.5`

> Note: If the latest version of php is v7.3, then running `brew install php` or `brew install php@7.3` will result in the same installation directory. That is, the base directory would still be `/usr/local/Cellar/php/7.3.5/`. So both commands are effectively the same.
However if latest version is 7.3 and we install an older version of php, say for example `brew install php@7.2`, then the base installation directory will be `/usr/local/Cellar/php@7.2/7.2.18/` which as you can see includes the `@7.2` version number in the path and has the latest php 7.2 version number.

Homebrew installes the latest php binaries at:

`/usr/local/Cellar/php/7.3.5/bin`

So the `php` cli is located at `/usr/local/Cellar/php/7.3.5/bin/php` and the `pecl` cli for installing php extensions is located at `/usr/local/Cellar/php/7.3.5/bin/pecl`

Similarly brew installs the latest system binaries for php at:

`/usr/local/Cellar/php/7.3.5/sbin`

Therefore the `php-fpm` server is installed at `/usr/local/Cellar/php/7.3.5/sbin/php-fpm`

Brew also adds symlinks that point to the installed binary files in the `/usr/local/bin/` and `/usr/local/sbin` directories.

For instance it adds the symlink `/usr/local/bin/php` symlink that points to the `/usr/local/Cellar/php/7.3.5/bin/php` binary.

So to be able to run the php commands globally we then need to add `/usr/local/bin` and `/usr/local/sbin` in our PATH envoronment variable.

For example, to be able to access the php cli and php-fpm server binaries, my PATH variable starts with:

`export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin`

## Symlinks to the installation directory

As mentioned before, when installing the latest version of php using the `brew install php` command the brew installation will create symlinks that reference the php binaries under the base `/usr/local/Cellar/php/7.3.5/` installation directory.

Here is a full list of the symlinks:

```bash
/usr/local/sbin/php-fpm -> /usr/local/Cellar/php/7.3.5/sbin/php-fpm

/usr/local/bin/pecl -> /usr/local/Cellar/php/7.3.5/bin/pecl
/usr/local/bin/pear -> /usr/local/Cellar/php/7.3.5/bin/pear
/usr/local/bin/peardev -> /usr/local/Cellar/php/7.3.5/bin/peardev
/usr/local/bin/phar -> /usr/local/Cellar/php/7.3.5/bin/phar
/usr/local/bin/phar.phar -> /usr/local/Cellar/php/7.3.5/bin/phar.phar
/usr/local/bin/php -> /usr/local/Cellar/php/7.3.5/bin/php
/usr/local/bin/php-cgi -> /usr/local/Cellar/php/7.3.5/bin/php-cgi
/usr/local/bin/php-config -> /usr/local/Cellar/php/7.3.5/bin/php-config
/usr/local/bin/hpdbg -> /usr/local/Cellar/php/7.3.5/bin/phpdbg
/usr/local/bin/phpize -> /usr/local/Cellar/php/7.3.5/bin/phpize
```

Brew will also create the following symlinks to the installation directory:

`/usr/local/opt/php -> usr/local/Cellar/php/7.3.5`
`/usr/local/opt/php@7.3 -> usr/local/Cellar/php/7.3.5`

Note that both `/usr/locl/opt/php` and `/usr/locl/opt/php@7.3` reference the same installation.

> Note: The symlinks in the `/usr/local/opt/` directory will not be relevent to this article but in any case I mention them here for your information.

## PHP ini and configuration files

The brew installation puts the configuration files for installed php versions in the directory:

`/usr/local/etc/php/<version>/`

So the brew installation for the latest php version on my machine put all configuration files for the php cli and php-fpm in:

`/usr/local/etc/php/7.3/`

So for instance the php.ini file is located at:

`/usr/local/etc/php/7.3/php.ini`

and the php-fpm.conf file at:

`/usr/local/etc/php/7.3/php-fpm.conf`

To find the location of the php.ini file you can run the following command:

`pear config-show | grep php.ini`

## Installing PHP extensions using PECL

The pecl CLI installed by homebrew can be used to install further php extensions that we may require by running the command `pecl install <extension>`.

For instance we can install the xdebug extension by typing:

`pecl install xdebug`

and we can list the pecl installed php extensions by running:

`pecl list` or `php -m`

The directory where the php extensions are installed depends on the php version. Each installed php version will add a release date directory to the base extentions directory underwhich the extensions will be added.

Each installed php version will has its own php.ini configuration file within which the php extensions directory for that version is specified. The setting in the php.ini file that specifies the installation directory is:

`extension_dir = "/usr/local/lib/php/pecl/20180731"`

So after we run `pecl install xdebug` we can see that xdebug is installed at:

`/usr/local/lib/php/pecl/20180731/xdebug.so`

To show the value of pecl extension directory setting in the php.ini file , you can run the following:

`pecl config-get ext_dir`

To print all pecl configuration settings run the following:

`pecl config-show`

> Note: It may appear that the php extensions are also installed in `/usr/local/Cellar/php/7.3.5/pecl/20180731/`, but that is because `/usr/local/Cellar/php/7.3.5/pecl/` is a symlink to `/usr/local/lib/php/pecl/`.

## How php detects pecl installed extensions

In order for php to detect pecl installed extensions, we must declare them in the php.ini file. The `pecl install` command will add the extension to the php.ini file for us.

If we look inside the php.ini file we will see that xdebug is actually installed as a special type of extension that is a zend extension:

`zend_extension="xdebug.so"`

Normally php extensions are installed as standard php extensions and will be referenced in the php.ini file as:

`extension="<php-extension>.so"`

So if we run `pecl install memcached` for instance, the extension will be installed at `/usr/local/lib/php/pecl/20180731/memcached.so` and `extension="memcached.so"` will be added to the top of the php.ini file.

> Note: when installing the memcached extension on my machine, a cached version of a previous install was not cleaned up so pecl was detecting memcached was installed when in fact it was not. To solve this I did a `pecl uninstall memcached` and then I re-installed the extension with `pecl install memcached` and that resolved the issue.

> Note: if after installing the xdebug extension, it is not detected and listed under [Zend Modules] when running `php -m` make sure that `zend_extension="xdebug.so"` is added to the top of the php.ini file, and if any standard installed php extension is not listed under [PHP Modules] when running `php -m` make sure that the `extension="<php-extension>.so"` for that extension name is added to the top of the php.ini file.

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

## Switching between latest and older installed php versions

In order to run older version of php when we execute the php, pecl commands and also to run the older version of php-fpm server we need to use brew to switch between versons.

Assuming we already installed the the latest(7.3) and 7.2 versions by running brew install php and brew install php@7.2, we can switch the version by running the following commands:

```bash
sudo brew services stop php
brew unlink php && brew link --force php@7.2
sudo brew services start php@7.2
```

> Note: we need to use the --force flag when linking older versions of php

If we look in `/usr/local/bin/` and  `/usr/local/sbin/` before executing the commands we see the following php binary file symlinks pointing to the latest version homebrew install directories:

```bash
/usr/local/sbin/php-fpm -> /usr/local/Cellar/php/7.3.5/sbin/php-fpm

/usr/local/bin/pecl -> /usr/local/Cellar/php/7.3.5/bin/pecl
/usr/local/bin/pear -> /usr/local/Cellar/php/7.3.5/bin/pear
/usr/local/bin/peardev -> /usr/local/Cellar/php/7.3.5/bin/peardev
/usr/local/bin/phar -> /usr/local/Cellar/php/7.3.5/bin/phar
/usr/local/bin/phar.phar -> /usr/local/Cellar/php/7.3.5/bin/phar.phar
/usr/local/bin/php -> /usr/local/Cellar/php/7.3.5/bin/php
/usr/local/bin/php-cgi -> /usr/local/Cellar/php/7.3.5/bin/php-cgi
/usr/local/bin/php-config -> /usr/local/Cellar/php/7.3.5/bin/php-config
/usr/local/bin/hpdbg -> /usr/local/Cellar/php/7.3.5/bin/phpdbg
/usr/local/bin/phpize -> /usr/local/Cellar/php/7.3.5/bin/phpize
```

But after we run the `brew unlink php && brew link --force php@7.2` command we see the symlinks point to the older v 7.3 version installation dirctories

```bash
/usr/local/sbin/php-fpm -> /usr/local/Cellar/php@7.2/7.2.18/sbin/php-fpm

/usr/local/bin/pecl -> /usr/local/Cellar/php@7.2/7.2.18/bin/pecl
/usr/local/bin/pear -> /usr/local/Cellar/php@7.2/7.2.18/bin/pear
/usr/local/bin/peardev -> /usr/local/Cellar/php@7.2/7.2.18/bin/peardev
/usr/local/bin/phar -> /usr/local/Cellar/php@7.2/7.2.18/bin/phar
/usr/local/bin/phar.phar -> /usr/local/Cellar/php@7.2/7.2.18/bin/phar.phar
/usr/local/bin/php -> /usr/local/Cellar/php@7.2/7.2.18/bin/php
/usr/local/bin/php-cgi -> /usr/local/Cellar/php@7.2/7.2.18/bin/php-cgi
/usr/local/bin/php-config -> /usr/local/Cellar/php@7.2/7.2.18/bin/php-config
/usr/local/bin/phpdbg -> /usr/local/Cellar/php@7.2/7.2.18/bin/phpdbg
/usr/local/bin/phpize -> /usr/local/Cellar/php@7.2/7.2.18/bin/phpize
```

Remember that we added `/usr/local/bin/` and  `/usr/local/sbin/` to our PATH variable so when we run the `php` or `pecl` commands we are executing the command that is symlinked to the proper version of the binary file we want to run. And since we are running the symlinked version of `pecl`, the php extensions that we install or uninstall are installed in the proper versions extenion file directory.

Before switching versions we stopped the current versions `php-fpm` service if it was running and started the service for the version that we switched to.

Running `php -v` we can see that the older version is reported.

To switch back to using the latest version we need to reverse the process by executing:

```bash
sudo brew services stop php@7.2
brew unlink php@7.2 && brew link php
sudo brew services start php
```

Running `php -v` we can see that the lates version is reported again.

As an aside, we can list all the services installed by brew to see the php-fpm instances

```bash
brew services list | grep php
```

After running this command I can see the php-fpm services for each version installed on my machine and their status:

```bash
php       started aregsarkissian /usr/local/opt/php/homebrew.mxcl.php.plist
php@7.2   stopped
```

> Note: PHP extensions installed using `pecl install` command are installed for the version of the php that is currently activated. So you must install php extensions you need for each installed version of php seperately. You can however reference the pecl binary for each version directly to install extensions for that version, without having to switch versions. For example you can run `/usr/local/Cellar/php/7.3.5/bin/pecl install xdebug` directly to install extensions for php version 7.2 while the current activated version of php that you are running is set to the latest v7.3.
Also note that the version of an extension that you need to install for older version might not be the latest version of the extension. In that case you need to run install the specific versone of the extenion you need. For instance instead of `pecl install xdebug` you will need to run `pecl install xdebug-2.7.1`. Finally you can upgrade the installed version of an extenion by running `pecl upgrade <extension-name>` for instance `pecl upgrade xdebug`.

## Resolving PHP 7.3 and JIT Compiler error

If you have the php `composer` package manager installed, you may need to deactivate the php v7.3 JIT compiler setting so that the  `composer global update` command does not fail.

The JIT setting is in the `/usr/local/etc/php/7.3/php.ini` file.

You can run the following to see the setting:

```bash
cd cd /usr/local/etc/php/7.3
cat php.ini | grep 'pcre.jit'
```

uncommnet the setting:
;pcre.jit=1

and change its value:
pcre.jit=0

## Conclusion

I listed the steps to take to install homebrew and then use homebrew to install php.

Here is a recap of the directories for the latest and older version of php after installing each version:

After running `brew install php`

#php installed at
/usr/local/Cellar/php/7.3.5/bin/









The following resources were used to produce this article:

[https://medium.com/@romaninsh/install-php-7-2-xdebug-on-macos-high-sierra-with-homebrew-july-2018-d7968fe7e8b8](https://medium.com/@romaninsh/install-php-7-2-xdebug-on-macos-high-sierra-with-homebrew-july-2018-d7968fe7e8b8)

[https://vyspiansky.github.io/2018/11/08/set-up-php-7.2-on-macos-mojave-with-homebrew/](https://vyspiansky.github.io/2018/11/08/set-up-php-7.2-on-macos-mojave-with-homebrew/)

[bhttps://threenine.co.uk/setting-php7-development-mac-osx/](https://threenine.co.uk/setting-php7-development-mac-osx/)

[https://stitcher.io/blog/php-73-upgrade-mac](https://stitcher.io/blog/php-73-upgrade-mac)

[https://tommcfarlin.com/running-multiple-versions-of-php-with-homebrew/](https://tommcfarlin.com/running-multiple-versions-of-php-with-homebrew/)

[https://grrr.tech/posts/installing-homebrew-php-extensions-with-pecl/)](https://grrr.tech/posts/installing-homebrew-php-extensions-with-pecl/)

[https://discourse.brew.sh/t/pecl-with-multiple-php-versions/1977/2](https://discourse.brew.sh/t/pecl-with-multiple-php-versions/1977/2)