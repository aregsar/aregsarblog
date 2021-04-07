# Installing Laravel CLI on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Installing Laravel installer for the first time

Install the latest version of the installer:

```bash
composer global require laravel/installer
```

Check the installer version:

```bash
laravel --version
```

The installer is installed globally in my MacBooks ~/.composer directory.

We can see it installed by dumping the content of the ~/.composer/composer.json file:

```bash
cat ~/.composer/composer.json
```

We can see the laravel/installer dependency along with its version.

```json
{
  "require": {
    "laravel/installer": "^4.2",
    "laravel/valet": "^2.14"
  }
}
```

## Reinstalling laravel installer

uninstall current installed version

composer global remove laravel/installer

install the latest version of the installer

composer global require laravel/installer

Check the installer version:

```bash
laravel --version
```

## Reinstalling when running into package dependancy issues

if issues removing and re installing

```bash
cd ~/.composer
rm -rf vendor
rm composer.lock
```

remove laravel/installer entry from the composer.json file then run:

```bash
composer global update
```

If still having issues take note of all packages installed in composer.json then delete composer.json
and then for all the previously installed packages run:

```bash
composer global require vendor/package
```

You can require multiple packages separated by a space character in single line or require them individually

## Using the Laravel installer

Since the installer is installed globally we can run it from any directory

Display available project creation options:

```bash
laravel new --help
```

The most basic way of using the installer is:

```bash
cd path/to/my/laravel/projects
laravel new mynewapp
```

This will create a new Laravel project mynewapp directory

Note that as it starts installing it prints the version of the laravel/laravel package which is the laravel project scaffold that it is creating:

```bash
Creating a "laravel/laravel" project at "./mynewapp"
Installing laravel/laravel (v8.5.15)
```

The laravel/laravel project requires the laravel/framework package which is the actual laravel framework.
Once the project is created we can check the framework version using the artisan command:

```bash
cd mynewapp
php artisan --version
```

The result is:

```bash
Laravel Framework 8.36.2
```

we can also see the required Laravel Framework version and PHP version in the composer.json and composer.lock files:

```bash
cat composer.json
```

```json
{
  "require": {
    "php": "^7.3|^8.0",
    "laravel/framework": "^8.12"
  }
}
```

```bash
cat composer.lock
```

```json
{
  "name": "laravel/framework",
  "version": "v8.36.2",
  "source": {
    "type": "git",
    "url": "https://github.com/laravel/framework.git",
    "reference": "0debd8ad6b5aa1f61ccc73910adf049af4ca0444"
  },
  "dist": {
    "type": "zip",
    "url": "https://api.github.com/repos/laravel/framework/zipball/0debd8ad6b5aa1f61ccc73910adf049af4ca0444",
    "reference": "0debd8ad6b5aa1f61ccc73910adf049af4ca0444",
    "shasum": ""
  }
}
```

Notice in composer.json we the semantic version constraint ^8.12 is specified which allows for newer versions starting from 8.12.0 to less then 9.0.0 be upgraded to when generating the composer.lock file.

We see that the current version upgrade in composer.lock for my install is the exact version v8.36.2

## The installed project scaffold version

The installer always installs the latest release of the project scaffold.

If you need a specific version of the project scaffold then you need to install it via composer specifying version:

```bash
composer create-project --prefer-dist laravel/laravel:^6 mynewapp
```

Omitting the version will install the exact same version that the Laravel installer installs.

If you need to user a specific version of the Laravel framework package, you need to change the `laravel/framework` package version require d in composer.json manually and then running composer update or remove the package using composer remover `laravel/framework` and the requiring the specific package version using composer require `laravel/framework:8.2.0` for instance.

## Using the command line git and github options

Create a new laravel project and then make a local git repo for it

```bash
laravel new mynewapp --git
```

Create a new laravel project, make a local git repo for it and then create a private remote repositiory for it pushing up to a tracking master branch

```bash
laravel new example-app --github
```

The same as above but creats a public remote repo

```bash
laravel new example-app --github="--public"
```

Create the rempote repo for an organization

```bash
laravel new example-app --github --organization="myorg"
```

The remote repo will be installed at:

`https://github.com/<your-account>/mynewapp.com`

The github option uses the github gh cli that you can install using Homebrew:

```bash
brew install gh
```

You need to authnticate with Github before creating the project

```bash
gh auth login -h github.com --web
```

After creating the project you can view the remote repo

```bash
cd mynewapp
#use the current directory name as the repo name
gh repo view --web
```

To logout of github after creating the project and viewing the remote repo:

```bash
echo "Y" | gh auth logout -h github.com
```

## Jetstream options

To install Laravel jetstream we can use the --jet option. This can be combined with the git options.

Also we can specify the UI stack that the jetstream project, livewire or inertia, should use so we dont have to specify it interactively

Finally we can add the --teams option of Laravel jetstream to add team account functionality.

Below are various combinations of these options:

create a new Laravel jetstream project

```bash
laravel new mynewapp --jet --github
```

create a new Laravel jetstream project using the livewire UI stack

```bash
laravel new mynewapp --jet --stack=livewire --github
```

create a new Laravel jetstream project using the inertia UI stack

```bash
laravel new mynewapp --jet --stack=inertia --github
```

create a new Laravel jetstream project using the livewire UI stack and teams functionality

```bash
laravel new mynewapp --jet --stack=livewire --teams --github
```
