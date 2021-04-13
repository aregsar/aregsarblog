# Installing Composer on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

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

## Composer global packages home

When you install composer a composer home directory is created where global packages (non project specific packages) can be installed.

Run the following command to see the location location of the symlinks to global package executables:

```bash
composer global config bin-dir --absolute
```

The result on my machine is:

```bash
~/.composer/vendor/bin
```

Make sure to add this path to your system $PATH so that we can execute the commands from any directory

Below are symlinks for two cli packages installed globally on my machine. They point to the vendor directory where the packages are installed:

```bash
~/.composer/vendor/bin/laravel -> ~/.composer/vendor/laravel/installer/bin/laravel
~/.composer/vendor/bin/valet -> ~/.composer/vendor/laravel/valet/valet
```

The global composer configuration file and vendor directory paths are in composer home directory.

```bash
~/.composer/composer.json
~/.composer/composer.lock
~/.composer/vendor/
```

## composer commands and the composer pipeline

Composer has a set of commands for adding, removing, updating and installing packages and their dependancies.

These commands can be executed as part of a pipeline to ultimately install or remove installed pakages.

Composer also has a command for generating autoload classmap files for then classes in your application. This command runs at the end of the pipeline.

Each command that is executed performs its task then calls the next command in the pipeline.

Each command when executed generates an artifact that can be used by any of the commands after it in the pipeline.

The left to right sequence of commands in the pipeline is shown below.

The first command in the sequence can be either the `composer require` or the `composer remove` command.

For composer require we have the sequence below:

```bash
composer require -> composer update -> composer install -> composer dump-autoload
```

And here is the sequence for composer remove:

```bash
composer remove -> composer update -> composer install -> composer dump-autoload
```

Below each command step of the pipeline starting from the left, is shown in a single line to detail the artifact created or updated by the command that will be used by the next command in the pipeline:

```bash
composer remove  (removes specified package from composer.json)
composer require (adds package with version constraint to composer.json, creates composer.json if it not exist)
composer update (uses composer.json to update composer.lock, creates composer.lock if it does not exists)
composer install (uses composer.lock to download and install packages in the vendor directory, creates the vendor directory if it does not exist)
composer dump-autoload (generates autoload classmap files from composer.json, then runs any scripts in composer.json scripts section)
```

We can start at any point in the pipeline, calling any of the commands in the flow to regenerate the artifact it is responsible as long as the artifact it depends on exists and is in sync with previously generated artifacts.

Below composer commands are described in detail in its own section.

## creating the initial composer.json file

This will interactively create a new composer.json file:

```bash
composer init
```

If you don't create a composer.json file before requiring packages, then the composer.json file will automatically be created for you when you require your first package.

## requiring packages

Install packages using the require command:

```bash
#the latest version
composer require vendor/package
```

This will add the package to the `require` section of the composer.json file and then call `composer update` described in the section below to install the package.

We can also specify an optional version number using a version constraint.

```bash
#specifying a version
composer require vendor/package:<version>
```

We can install multiple packages as well by separating them with a space.

To add a package as a development dependency we can use the --dev option:

```bash
composer require vendor/package --dev
```

This will add the package to the `require-dev` section of the composer.json file and then call `composer update` described in the section below to install the package.

To add a package globally we can use the global modifier:

```bash
composer global require vendor/package
```

The global require command adds the package to the composer.json in the composer home directory (~/.composer) and then call `composer global update` described in the section below to install the package inside the vendor directory in the composer home directory.

The `composer require` command will create the composer.json file if it does not exist.

## removing packages

The remove command will remove the installed package from the composer.json file and then call `composer update` described in the section below to uninstall the package.

```bash
composer remove vendor/package
```

Add the global modifier to remove a globally installed package

```bash
composer global remove vendor/package
```

Multiple package names separated by a space character can be specified.

The global remove command removes the package from the composer.json in the composer home directory (~/.composer) and then call `composer global update` described in the section below to uninstall the package.

## updating packages

The composer update command resolves dependancies from packages specified in composer.json and updates the composer.lock file with specific versions of packages to install then calls `composer install` command to install the packages. It will create the composer.lock file if it does not exist.

If there are changes in composer.json from the last version that created or updated the composer.lock file, then the update command will update the composer.lock file based on resolution of the changes in composer.json.

Also if there are new package versions released that satisfy the version roll forward constraint of the package in composer.json then the update command will update the composer.lock file with the latest version of the package that satisfies the version constraint.

```bash
composer update
```

To update global packages we can use the global subcommand which uses the composer.json from our composer home directory to create or update composer.json file in the composer home directory and then call composer global install to install global packages installed into the vendor directory in the composer home directory:

```bash
#update all global packages
composer global update
```

We can use the dry run flag to see the effects of the update before we actually run it

```bash
composer update --dry-run
```

> The composer require and composer remove commands call the composer update command after adding or removing packages in composer.json

## installing packages

composer dump-autoload generates autoload files PHP autoload files specified the composer.json autoload section, then runs the scripts from the scripts section of composer.json

Run the install command to install the specific versions of packages specified in the composer.lock file into the vendor directory:

```bash
composer install
```

composer install downloads and installs the package versions specified in the composer.lock file into the vendor directory. If the vendor directory does not exist it will be created. It then calls composer dump-autoload to generate the PHP autoload files. If the --optimize-autoload option is specified on the command line or the optimize-autoloader setting in composer.json is set to true, then it calls composer dump-autoload with the --optimize option.

To install packages globally we can use the global modifier.

```bash
composer global install
```

The global install uses the composer.lock in the composer home directory to install the package versions specified in the composer.lock file into the vendor directory of the composer home directory. If the vendor directory does not exist it will be created.

The global install command also creates symlinks to executables within the installed packages, in the vendor/bin directory of the home directory.

## Composer install Options for production builds

```bash
composer install --no-dev --optimize-autoloader --no-interation --no-scripts --no-progress --prefer-dist
```

The --no-dev option only installs non development dependancies in a production environment.

The --no-script option makes sure that the scripts in the scripts section of the composer.json file do not run.

The --optimize-autoloader generates optimized PHP autoload files.

> note --prefer-dist tries to install from packagist.org and falls back to git repository. Replace it with --prefer-source to try to install from the git repository then fallbask to packagist.org

## Updating installed package version constraint

We saw earlier that we can use the `composer update` command to upgrade package to the latest version within their specified version roll forward constraint.

> When no package version is specified in composer.json then the default constraint is to move to the latest version of the package (which is no constrain).

However sometimes we want to change the version constraint for the package or we want to downgrade or upgrade to a specific version.

In that case we can remove the package using composer remove and then re-install the package using composer require with the specific package version that we want or with the new package version constraint.

Another way is to manually update the package version in composer.json and then run the composer update command.

Sometime either method may fail due to version conflicts. How to overcome that is described in the next section.

## Issues with version conflict when using composer update

If there are package version conflict when running composer update or running composer require which runs composer update then the solution is to first delete the vendor directory and composer.lock file.

```bash
rm -rf vendor
rm composer.lock
```

Then open the composer.json and add, remove or update package versions in the require and require-dev sections. Then either run composer update or composer require (if you want to add additional packages).

This will run composer update which will then generate a new composer.lock file and call composer install which will create the vendor directory and add the new resolved package versions from the composer.lock file into the vendor directory.

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

Specifying the --optimize or -o option will optimize the generated classmap.

```bash
composer dump-autoload --optimize
```

Instead of the command line option we can specify the optimization through setting in composer.json

```json
{
  "config": {
    "optimize-autoloader": true
  }
}
```

> Running compose install will call composer dump-autoload. Passing in --optimize-autoloader to composer install will call composer dump-autoload --optimize

## dumping the autoload configuration in production

To dump an optimized version of the autoloader for production we can use the -o or --optimize option

```bash
#generate classmap file vendor/composer/autoload_psr4.php
composer dump-autoload --optimize
```

As previously noted, when we run the `composer install` command it will call composer dump-autoload to generate the autoload classmap after installing the packages. So in production we can use the -o or --optimize-autoloader option when running composer install to have composer dump-autoload be called with the --optimize option.

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

Using the example above, The `app` directory in the root of the project will map to the `\App` namespace and the `app/config` subdirectory will map to the `\App\Config` namespace. Therefore a
a class named `User` in the file `app/user.php` will have a fully qualified class name of `\App\User` and a class named `Database` in the file `app/configs/database.php` will have a fully qualified class name of `\App\Config\Database`.

Note that the fully qualified namespace names start with a backslash signifying the root namespace.

You can always use the fully qualified namespace in your PSR-4 files. However when a PSR-4 namespace is declared within the file, then the code can reference other classes within that namespace and the hierarchy below it using relative class names. For example we might have a file `app/book.php` and a file `app/user.php` that both declare `namespace App`. Then we in `User` class can reference the the `Book` class without its fully qualified namespace and vice versa. Assuming `app/configs/database.php` declares `namespace App\Config` then it can be referenced from the either class as `Config\Database` which is qualified relative to the `App` namespace.

## The Composer.json structure

The composer.json schema below will illustrate components of a composer.json file:

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
  },
  "scripts": {
    "post-autoload-dump": [
      "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
      "@php artisan package:discover --ansi"
    ],
    "post-root-package-install": [
      "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
    ],
    "post-create-project-cmd": ["@php artisan key:generate --ansi"]
  },
  "bin": ["valet"],
  "extra": {
    "laravel": {
      "dont-discover": []
    }
  },
  "config": {
    "optimize-autoloader": true,
    "preferred-install": "dist",
    "sort-packages": true
  },
  "minimum-stability": "dev",
  "prefer-stable": true
}
```

require section is for loading packages. It also can specify the PHP version to use.

The require-dev section is for packages only used for development that are installed with the --dev option. The --no-dev option for the install command in production will bypass loading those packages.

The autoload section is for our PSR-4 classes and generates vendor/composer/autoload_psr4.php

the autoload-dev section is for loading our PSR-4 classes that are only used for development. The --no-dev option for the install command in production will bypass generating an autoload file for those mapping.

classmap section is for loading classes in directories that are in a directory path outside the psr4 directory and generates vendor/composer/autoload_classmap.php

Composer, starting with the project root scans the specified classmap files and directories for php classes and requires them.

files section is for loading files with global php functions

repositories section is for instructing composer to look for a package to install from the repo location first

the bin section lists binary executables installed by the package in a vendor/bin directory

the config section contains the following settings:

The optimize-autoloader set to true makes composer dump-autoload optimize the autoload php classmap files

The preferred-install wet to "dist" attempts to download packages for packegist.org before falling back to the package repository

## Package versioning version constraints

Using semantic versioning the ^ allows both minor and patch versions to increment to future minor and patch releases starting with the specific minor and patch version specified

^2 (2.0.0) is equivalent to version that is >=2.0.0 but <3.0.0

^2.6 (2.6.0) is equivalent to version that is >=2.6.0 but <3.0.0

^2.6.4 is equivalent to version that is >=2.6.4 but <3.0.0

Using semantic versioning the ~ allows only patch version to increment to future patch releases starting with the patch version specified:

~2 (2.0.0) is equivalent to version that is >=2.0.0 but <2.1.0

~2.6 (2.6.0) is equivalent to version that is >=2.6.0 but <2.7.0

~2.6.4 is equivalent to version that is >=2.6.4 but <2.7.0

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

The classes in files in the directory hierarchy map accordingly:

app\User.php => namespace App;

app\Config\Storage.php => namespace App\Config;

app\Helpers\Time.php => namespace App\Helpers;

The namespace in the files matches the directory structure starting with the PSR-4 mapped namespace

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

in app\Config\Storage.php

```php
<?php
namespace App\Config;
use App\User;
use App\Helpers\Time;
class Storage{}
```

Also note that the namespace references using the `use` keyword start from the PSR-4 mapped namespace and follow the directory hierarchy.

Finally we have a bootstrap file named index.php that autoloads all classes by including the vendor/autoload.php file

```php
require_once __DIR__ . '/../vendor/autoload.php';
use App\User;
use App\Config\Storage;
$user = new User;
$db = new Storage;
```

Again note that the namespace references using the `use` keyword start from the PSR-4 mapped namespace and follow the directory hierarchy.
