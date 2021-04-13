# Installing Laravel CLI on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

## Installing Laravel installer for the first time

Install the latest version of the installer:

```bash
composer global require laravel/installer
```

Check the installer version:

```bash
laravel --version
```

The installer is installed globally in the composer home directory which on the Mac is at `~/.composer` directory.

If we look inside the `~/.composer/composer.json` file we can see we can see the `laravel/installer` dependency along with its version.

```json
{
  "require": {
    "laravel/installer": "^4.2",
    "laravel/valet": "^2.14"
  }
}
```

## Reinstalling the Laravel installer

uninstall the current installed version

```bash
composer global remove laravel/installer
```

Install the latest version of the installer

```bash
composer global require laravel/installer
```

Check the installer version:

```bash
laravel --version
```

## Reinstall workaround when running into package dependency issues

If you have dependency conflict issues when trying to remove or install the installer you can try to reinstall all the global packages from scratch.

First remove the vendor directory and composer.lock file from the composer home directory:

```bash
cd ~/.composer
rm -rf vendor
rm composer.lock
```

Next remove the `laravel/installer` entry from the `composer.json` file then run:

```bash
composer global update
```

This should reinstall the installer.

> If you have Laravel Valet installed, you may have to remove the `laravel/valet` entry from `composer.json` as well and then re-install it.

If you are still having issues, take note of all packages installed in `composer.json` and install them again after deleting `composer.json`.

To install all the previously installed packages after deleteing composer.json, run:

```bash
composer global require vendor/package
```

You can require all the packages in single line separated by a space character or require them all individually

## Using the Laravel installer

Since the installer is installed globally we can run it from any directory.

Display available project creation options:

```bash
laravel new --help
```

The most basic way of using the installer is:

```bash
cd path/to/my/laravel/projects
laravel new mynewapp
```

This will create a new Laravel project directory named mynewapp.

Note that as it the installer starts installing, it prints the version of the `laravel/laravel` package that is being installed.

```bash
Creating a "laravel/laravel" project at "./mynewapp"
Installing laravel/laravel (v8.5.15)
```

the `laravel/laravel` package is the Laravel project scaffold that contains the directory structure of the project being created.

The `laravel/laravel` packages `composer.json` file requires the `laravel/framework package` which is the actual Laravel framework being installed.

Once the project is created we can check the `laravel/framework` Laravel framework version using the artisan command:

```bash
cd mynewapp
php artisan --version
```

The result is:

```bash
Laravel Framework 8.36.2
```

We can see that it has its own version number that is different form the `laravel/laravel` project scaffold.

We can see the required Laravel Framework version and PHP version in the installed project's `composer.json` and `composer.lock` files:

```bash
cat composer.json
```

Snippet of the output showing the semantic framework version constraint:

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

Snippet of the output showing the exact framework version:

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

The current version in composer.lock for my install is the exact version v8.36.2.

## The installed project scaffold version

The installer always installs the latest release of the project scaffold.

If you need a specific version of the project scaffold then you need to install it via composer and specify the version you need installed:

```bash
composer create-project --prefer-dist laravel/laravel:^6 mynewapp
```

Omitting the version will install the exact same version that the Laravel installer installs.

Also, if you need to user a specific version of the Laravel framework package that the Laravel project requires in composer.json, you can manually change the `laravel/framework` package version in `composer.json` and then run `composer update`.

Alternatively, instead of making a manual change, you can remove the package entry using `composer remove laravel/framework` and then requiring the specific package version using `composer require laravel/framework:8.2.0` for instance.

## Using the command line git and github options

You can use the Laravel installer to create a new laravel project and then make a local git repo for it in a single command:

```bash
laravel new mynewapp --git
```

You can also make it create a private remote repository and push up the local master branch to a tracking master branch at Github by using the --github option instead of --git.

```bash
laravel new example-app --github
```

The remote repo will be installed at:

`https://github.com/<your-account>/mynewapp.com`

The following is the same as above but creates a public repository instead:

```bash
laravel new example-app --github="--public"
```

Adding the --organization option allows you to create the remote repo under any of your Github organization:

```bash
laravel new example-app --github --organization="myorg"
```

The --github option uses the Github `gh` cli that you can install using Homebrew:

```bash
brew install gh
```

You need to authenticate with Github with `gh` before running the installer command that uses the --github option:

```bash
gh auth login -h github.com --web
```

The command will give you code in the terminal and launch the browser where you can paste in the code to authenticate:

After creating the project you can view the remote repo in your default web browser:

```bash
cd mynewapp
#use the current directory name as the repo name
gh repo view --web
```

This command will launch and navigate your default browser to the repo URL.

Logout of Github after creating the project and viewing the remote repo:

```bash
echo "Y" | gh auth logout -h github.com
```

## Jetstream options

To install Laravel jetstream we can use the --jet option. This can be combined with the git or github options from the last section.

Also we can specify the UI stack, livewire or inertia, that the jetstream project should use so we don't have to specify it interactively.

Finally we can add the --teams option of Laravel jetstream to add team account functionality. If we don't specify --teams the installer will prompt us. The default choice is no teams functionality if we just hit the return key.

Below are various combinations of these options:

Create a new Laravel jetstream project and create a Github repo.

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

bcreate a new Laravel jetstream project using the livewire UI stack and teams functionality

```bash
laravel new mynewapp --jet --stack=livewire --teams --github
```
