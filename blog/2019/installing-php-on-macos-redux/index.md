# Installing PHP on macOS Reduc

May 13, 2019 by [Areg Sarkissian](https://aregsar.com/about)

## Introduction

This post is an abbreviated form of my previous post on installing PHP on a Mac using the Homebrew package manager.

In that post I went into great detail about installation directories and installing multiple versions of PHP that make it a harder read for someone that simply wants to install the latest version of PHP.

So in this post I will detail the minimal steps necessary to install the latest version of PHP on your Mac using Homebrew package manager for the OSX.

For a more detailed understanding of the impact of the installation, check out [Installing PHP on macOS](https://aregsar.com/blog/2019/installing-php-on-macos).

## Installing Homebrew

Before we install PHP we need to install the Homebrew package manager, here after referenced simply as brew.

The installations for installing brew are located at [https://brew.sh/](https://brew.sh/).

Gathered from the docs, you can run the following bash statement to install brew:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

This bash script downloads and runs a Ruby language script using the default Ruby installation on your Mac.

## Updating Homebrew

If you already have brew installed, make sure to update it before installing PHP by running:

`brew update`

> Note: Be careful to not run __brew upgrade__ instead, which updates all the packages that brew has installed on your system.

Run `brew doctor` afterwards to make sure brew does not see any issues with current installed packages.

## Removing all brew installed versions of PHP

To do a fresh install, we first need to remove any existing brew PHP installations.

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

> Note: some of the commands above may not be required, but running them all will ensure a clean uninstall. You can safely ignore any file or directory does not exist errors.

## Installing the latest PHP version

Now install the latest PHP version using the command:

`brew install php`

After running this command latest version of PHP (7.3 as of this time) will be installed.

You can run the following command to verify the current PHP version.

`php -v`

Run the following to see all information and configuration settings for the installed version:

`php -i`

Run the following to only list the installed PHP extensions:

`php -m`

Run the following command to see the symlink created that points to the installed PHP CLI binary:

`which php`

You can show all installed versions of PHP by running:

`brew list | grep php`

## Adding the binary file symlinks to the PATH variable

To be able to run the PHP and PECL CLI commands globally we need to add the `/usr/local/bin` and `/usr/local/sbin` symlink directories to our PATH environment variable. 

For example, my PATH variable starts with:

`export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin`

These directories contain symlinks to the installed `php`, `pecl` and `php-fpm`  binaries.

## PHP ini and configuration files

The brew installation puts the configuration files for installed PHP versions in the directory:

`/usr/local/etc/php/<version>/`

So the brew installation for the latest PHP version on my machine put all configuration files for the PHP CLI and PHP_FPM in:

`/usr/local/etc/php/7.3/`

So for instance the php.ini file is located at:

`/usr/local/etc/php/7.3/php.ini`

and the php-fpm.conf file at:

`/usr/local/etc/php/7.3/php-fpm.conf`

To show the location of the php.ini file you can run the following command:

`pear config-show | grep php.ini`

Each installed version of PHP will have its own configuration file version directory under `/usr/local/etc/php/`.

## Installing PHP extensions using PECL

The PECL CLI installed by brew can be used to install specific PHP extensions that we may require for our projects.

Installing an extension involves adding an <extension>.so file to the extensions directory and adding either a `zend_extension="<zend-extension>.so"` or a `extension="<extension>.so"` setting to the php.ini configuration file, depending on the extension type. The PECL installed will add the extension and the setting for us.

We can install PHP extensions for the current active version of PHP, by running the command the following command:

`pecl install --force <extension>`

For instance we can install the xdebug extension by typing:

`pecl install --force xdebug`

> Note: We need to use the --force flag to avoid an issue where PECL install fails to install a non existing extension because of a cached setting.

To install a specific version of an extension we can specify the extension version like the example below:

`pecl install --force xdebug-2.7.1`.

To uninstall an extension run:

`pecl uninstall <extension>`

For instance we can uninstall the xdebug extension by typing:

`pecl uninstall xdebug`

We can list the PECL installed PHP extensions by running:

`php -m`

Each installed PHP version will add a PHP release date directory to the base extension directory `/usr/local/lib/php/pecl/` under which the extensions will be added.

The `php.ini` configuration has a setting that specifies the PHP extensions installation directory:

`extension_dir = "/usr/local/lib/php/pecl/20180731"`

So for example, after we run `pecl install xdebug` we can see verify that the `xdebug` extension is installed at:

`/usr/local/lib/php/pecl/20180731/xdebug.so`

You can run the following command to show the value of PHP extension directory setting from the `php.ini` file:

`pecl config-get ext_dir`

To print out all PECL configuration settings run the following:

`pecl config-show`

> The special Opcache extension is installed at `/usr/local/Cellar/php/7.3.5/lib/php/20180731/opcache.so` when PHP is installed.

## Running the php-fpm service using brew

We can check to see all the versions of php-fpm installed on our machine by running:

`brew services list | grep php`

On my machine I see:

```bash
php       stopped
php@7.2   stopped
```  

The first line is the php-fpm service for the latest PHP version.

We can use brew to configure the latest version of php-fpm to auto-start by running the brew command:

`sudo brew services start php`

Now when we run `brew services list | grep php` again and we will see:

```bash
php       started root           /Library/LaunchDaemons/homebrew.mxcl.php.plist
php@7.2   stopped
```

brew will add the file `/Library/LaunchDaemons/homebrew.mxcl.php.plist` when we run the service.

We can stop the service by running `sudo brew services stop php`

Running `brew services list | grep php` again we see the service is stopped and the file `/Library/LaunchDaemons/homebrew.mxcl.php.plist` is removed:

```bash
php       stopped
php@7.2   stopped
```  

The LaunchDaemons files are created under the home directory.

So on my machine, the actual full path of the LaunchDaemons files created for each version of the php-fpm service is:

`/Users/aregsarkissian/Library/LaunchAgents/homebrew.mxcl.php.plist`
`/Users/aregsarkissian/Library/LaunchAgents/homebrew.mxcl.php@7.2.plist`

> Note: We can start or stop either version of php-fpm regardless of which installed version of PHP is configured as our current active version.
So for example if we have both the latest version v7.3 and v7.2 installed and we have configured version 7.3 to be our current active version, we can still run `sudo brew services start php@7.2` to run the 7.2 installed version.
However it recommended to run the php-fpm version that matches your current active version. You can determine the current active PHP version by running `php -v`.

## Resolving PHP 7.3 Composer package manager error

PHP uses a package manager named `Composer` that is a separate install from PHP itself.

If you have the Composer PHP package manager installed, you may need to deactivate the PHP v7.3 JIT compiler setting so that the `composer global update` command does not fail.

The JIT setting is in the `/usr/local/etc/php/7.3/php.ini` file.

You can run the following to see the setting:

```bash
cd /usr/local/etc/php/7.3
cat php.ini | grep 'pcre.jit'
```

Run the following command to uncomment the setting and change its value:

`sed -i 's/;pcre.jit=1/pcre.jit=0/g' /usr/local/etc/php/7.3/php.ini`

## Recap of installation directories for latest version of PHP

Here is a recap of the directories for the latest PHP version

After running `brew install php` we have the following installation directories:

PHP binaries:

`/usr/local/Cellar/php/7.3.5/bin/php`
`/usr/local/Cellar/php/7.3.5/bin/pecl`
`/usr/local/Cellar/php/7.3.5/sbin/php-fpm`

PHP configuration files:

`/usr/local/etc/php/7.3/php.ini`
`/usr/local/etc/php/7.3/php-fpm.conf`

PHP extensions location:

`/usr/local/lib/php/pecl/20180731/`

Opcache extension location:

`/usr/local/Cellar/php/7.3.5/lib/php/20180731/opcache.so`

## Conclusion

In this article I listed the steps to take to install the Homebrew package manager for the macOS followed by the steps to install PHP.
