# Installing Composer on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## installing composer

The following steps are described at https://getcomposer.org/download

```bash
#download the setup script
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

#verify the script hash (make sure the hash code is the latest code from: https://composer.github.io/pubkeys.html)
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

#run the setup script
php composer-setup.php --filename=composer --install-dir=/usr/local/bin

#remove the setup script
php -r "unlink('composer-setup.php');"

#check installed version
composer -V
```

## Upgrading composer

```bash
#check installed version
composer -V

#upgrade to new version
composer self-update

#check installed version
composer -V
```

composer install should have created a ~/.composer directory

## removing composer

```bash
rm /usr/local/bin/composer
rm -rf ~/.composer
```

## creating the initial composer.json file

```bash
composer init
```

## requiring packages

```bash
composer require vendor/package
```

This will add the package to the compose.json and install the package.

> If there is no composer.json present, one will be created.

To add the package as a development dependency we can use the --dev option:

```bash
composer require vendor/package --dev
```

To require a package globally we can use the --global option:

```bash
composer require --global vendor/package
```

## installing packages

```bash
composer install
```

If there is no composer.lock file present, Composer resolves all dependencies listed in the composer.json file and downloads the latest version of their files into the vendor directory.

When Composer has finished installing, it writes all of the packages and the exact versions of them that it downloaded to the composer.lock file, locking the project to those specific versions.

## updating all installed packages

```bash
composer update
```

## dumping the autoload configuration

Whenever we update the PSR-4 autoloader configuration in composer.json we need to make composer aware or the changes
by dumping the autoloader configuration. This command updates the autoloader files under the vendor directory.

```bash
composer dump-autoload
```

To dump an optimized version of the autoloader for production we can use the -o or --optimize option

```bash
composer dump-autoload -o
```

## PSR-4 autoloader configuration

The autoloader configuration in the composer.json file maps a class namespace starting from the root of the project where the composer.json file resides to a directory under the root.

```json
{
  "PSR-4": {
    "\\App": "/app",
    "\\Resources\\Views": "/resources/views"
  }
}
```

> Note the root class namespace in the mapping uses a `\\` escape sequence

The class namespacing convention then follows the directory hierarchy starting from the mapped directory on down.
For example the class `User` in the file app/user.php will map to the \App\User namespace and the class Database in the file app/configs/database.php file will map to \App\Config\Database namespace
