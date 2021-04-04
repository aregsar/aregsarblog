# Installing Composer on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## installing composer

The following steps are described at [Composer](https://getcomposer.org/download)

Prior to installing make sure that `/usr/local/bin` is in your system $PATH variable.

1-Download the setup script

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
```

2-Optionally, verify the script hash

```bash
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

> Make sure to replace the hash code above with the latest code from [Composer](https://getcomposer.org/download) or from [Composer Public Keys](https://composer.github.io/pubkeys.html).

3-Run the setup script

```bash
php composer-setup.php --filename=composer --install-dir=/usr/local/bin
```

4-Remove the downloaded setup script

```bash
php -r "unlink('composer-setup.php');"
```

5-Verify installed version

```bash
composer --version
```

> The composer script install should have created a `~/.composer` directory that holds the global package installations and added a `COMPOSER_HOME` environment variable with that path to your system $PATH in your bash profile.

## Installing composer the quick install way

This alternative installation method uses curl to install composer and does not verify the hash

```bash
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

## Upgrading composer

Upgrade composer to the latest version:

```bash
composer self-update
```

Verify the updated version

```bash
composer --version
```

## removing composer

Just delete the composer files to remove composer.

```bash
rm /usr/local/bin/composer
rm -rf ~/.composer
```

## creating the initial composer.json file

This will interactively create a new composer.json file:

```bash
composer init
```

If you don't create a composer.json file before requiring packages, then it when the composer.json file automatically be created for you when you require your first package.

## requiring packages

Install packages using the require command:

```bash
#the latest version
composer require vendor/package
```

This will add the package to the `require` section of the composer.json file and will install the latest version of the package.

We can also specify an optional version number according to semantic versioning schema.

```bash
#specifying a version
composer require vendor/package:<version>
```

We can install multiple packages as well by separating them with a space.

To add a package as a development dependency we can use the --dev option:

```bash
composer require vendor/package --dev
```

This will add the package to the `require-dev` section of the composer.json file.

To add a package globally we can use the global modifier:

```bash
composer global require vendor/package
```

The global require uses the composer.json and composer.lock and vendor directory in the ~/.composer directory.

Example installing phpunit including the version as a development dependency

```bash
composer require phpunit/phpunit:^8 --dev
```

> Note `composer require` command updates the composer.json with the package and updates the composer.lock file based on the composer.json file changes, creating the files if they do not exist. Then it installs the package in the `vendor` directory based on the specific package version from the composer.lock file, creating the directory if it does not exist.

## removing packages

The remove command will remove and installed package

```bash
composer remove vendor/package
```

Add the global modifier to remove a globally installed package

```bash
composer global remove vendor/package
```

Multiple package names separated by a space character can be specified.

To change the version of the package that is installed, run the require command with the new version.

The global remove uses the composer.json and composer.lock and vendor directory in the ~/.composer directory.

> The remove command removes the package from composer.json and updates the composer.lock file based on changes in the composer.json file. It then removes the package from the vendor directory.

## updating packages

```bash
composer update
```

To update global packages we can use the global subcommand which uses the composer.json from our ~/.composer directory:

```bash
composer global update
```

The global update uses the composer.json and composer.lock and vendor directory in the ~/.composer directory.

> The update command updates the composer.lock file based on manual changes to composer.json (added, removed or version modified packages in the require or require-dev sections). It also updates the package versions in composer.lock, if the package has an updated version in the package repository that satisfies the package version constraint specified in composer.json. It then reconciles the differences between the currently installed packages in the vendor directory and their versions and any package or version changes in the composer.lock file and adds and removes packages in the vendor directory accordingly.

## installing packages

Run the install command to install the specific versions of packages specified in the composer.lock file into the vendor directory:

```bash
composer install
```

Running the composer install command also generates the PHP autoload files specified the composer.json autoload section. Also any scripts in the scripts section of composer.json will be run.

To install or reinstall packages globally use the global modifier.

```bash
composer global install
```

The global install uses the composer.json and composer.lock and vendor directory in the ~/.composer directory.

> The install command installs the specific package versions from the composer.lock file into the vendor directory. It reconciles the differences between the currently installed packages in the vendor directory and their versions and any package or version changes in the composer.lock file and adds and removes packages in the vendor directory accordingly. The command will not run if the package.json file is out of sync with the package.lock, which happens when composer.json required package list manually updated. In this scenario it will display a warning message instructing the user to run `composer update`, which will update the composer.lock file and install the updated packages in the vendor directory. Running `composer install` afterward will only re-generate the autoload files, since the packages will have been installed by the `composer update` command.

## Composer install Options for production builds

```bash
composer install --no-dev --optimize-autoloader --no-interation --no-scripts --no-progress --prefer-dist
```

The --no-dev option only installs non development dependancies in a production environment.

The --no-script option makes sure that the scripts in the scripts section of the composer.json file do not run.

The --optimize-autoloader generates optimized PHP autoload files.

> note --prefer-dist tries to install from packagist.org and falls back to git repository. Replace it with --prefer-source to try to install from the git repository then fallbask to packagist.org

## displaying composer.lock file dependency tree

```bash
composer show -t
```

## dumping the autoload configuration in development

Whenever we update the PSR-4 autoloader configuration in composer.json file we need to make composer aware of the changes.

We can do this by dumping the autoloader configuration. The command generates and\or updates the autoload PHP files in the vendor directory. The autoload files contain a `classmap` mapping between PHP classes and their file location.

```bash
composer dump-autoload
```

## dumping the autoload configuration in production

To dump an optimized version of the autoloader for production we can use the -o or --optimize option

```bash
#generate classmap file vendor/composer/autoload_psr4.php
composer dump-autoload --optimize
```

As previously noted, when we run the `composer install` command we can dump the autoload configuration at the same time. So production we can use the -o or --optimize-autoloader option when running composer install and there will be no need to also call the composer dump-autoload command.

```bash
composer install --optimize-autoloader --no-dev --no-interation --no-scripts --no-progress --prefer-dist
```

> if project does not use dynamic classes than --classmap-authoritative optimizer option does a better optimization than --optimize-autoloader option

## PSR-4 autoloader configuration

The autoloader configuration in the composer.json file maps a one or more class namespace names to a specific directory name under the root directory of the project. The namespace name on the left side of the colon is arbitrary and does not need to match the directory name on the right side.

Here is an example that maps two namespaces:

```json
{
  "PSR-4": {
    "\\App": "/app",
    "\\Resources\\Views": "/resources/views"
  }
}
```

> Note the class namespace in the mappings use a `\\` escape sequence.

The PSR-4 class namespacing convention then follows the directory hierarchy under the PSR-4 mapping, starting from the PSR-4 mapped directory on down.

Using the example above, The app directory in the root of the project will map to the `\App` namespace and a configs subdurectory of the app directiory will map to the `\App\Config` namespace. Therefore a
a class `User` in the file app/user.php will have a fully qualified class name of `\App\User` and a class Database in a file app/configs/database.php file will have a fully qualified class name of `\App\Config\Database`.

## The Composer.json structure

require section is for loading packages

semantic versioning: avoids breaking updates
^2.6 == >=2.6.4 <3.0.0

semantic versioning: any patch level that might introduce a breaking change
~2.6 == >=2.6 <3.0

autoloading section is for our PSR-4 classes and generates vendor/composer/autoload_psr4.php

classmap is for loading classes in directories that are in a
directory path outside the psr4 directory starting with the project root
scans classmap specified files and direvctories for php classes
classmap generates vendor/composer/autoload_classmap.php

files is for loading files with global php functions

repositories is for instructing composer to look for a package to install from the repo location first

````bash

## The composer.json file

```json
{
  "classmap": ["database/Seeders", "database/Factories", "factories/"],
  "files": ["global/functions.php"],
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  },
  "autoload-dev": {
    "psr-4": {
      "App\\Tests": "test/"
    }
  },
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/Seldaek/monolog"
    }
  ],
  "require": {
    "php": ">=7.4",
    "symfony/yaml": "2.6.4",
    "monolog/monolog": "1.12.0"
  },
  "require-dev": {
    "phpunit/phpunit": "^8"
  }
}
````

## composer autoloader and PHP namespaces relationship

In the index.php bootstrap file:

```php
require_once __DIR__ . '/../vendor/autoload.php';
```

## psr 4 directory structure and namespace mapping

assuming PSR-4 mapping in composer.json

```json
{
  "PSR-4": {
    "\\App": "/app"
  }
}
```

app\User.php => namespace App;
app\Config\Storage.php => namespace App\Config;
app\Helpers\Time.php => namespace App\Helpers;

in app\User.php

```php
<?php
namespace App;
use App\Helpers\Time;
class User{}
```

in app\Helpers\Time.php

```php
<?php
namespace App\Helpers;
class Time{}
```

## key takeways

Composer along with its configuration files composer.json and composer.lock is used to install and update packages used in PHP applications.
But composer is also used to dump php autoload files based on PSR-4 mapping section in composer.json for your own code.
Composer can also specfify classmapping for classes outside the PSR-4 directory mapping hiererchy and can also specify include files for files that define global functions.
In addition it can also run installation scripts from a script section in composer.json.
Finally you can specify repositories where composer can download packages from when developing and testing your own packages.
