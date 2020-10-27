# How To Create New Laravel 8 Project

September 29, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I will show two ways in which you can create a new Laravel 8 project.

> It is assumed that you already have PHP and Composer installed. I have posts how to get them both installed on this blog.

## Using Composer Create Project command

The easiest way to create a basic Laravel application is using the composer create-project command.

```bash
composer create-project --prefer-dist laravel/laravel blog
```

This will create a new project named `blog`, using the latest `laravel/framework` composer package version, in the current directory.

## Using the Laravel CLI

We can also install the the Laravel CLI and use it to create a new Laravel project.

First we need use composer to globally install the latest version of the Laravel installer:

```bash
#uninstall current installed version if any
composer global remove laravel/installer
#install the latest version of the installer
composer global require laravel/installer
```

Check the installer version:

```bash
laravel --version
```

The installer is installed globally in my MacBooks `~/.composer` directory.

I can see it installed by dumping the content of the `~/.composer/composer.json` file:

```bash
cat ~/.composer/composer.json
```

We can see the `laravel/installer` dependency along with its version.

```json
{
  "require": {
    "laravel/installer": "^4.0"
  }
}
```

Now that the installer is installed, we can create a new Laravel project by running:

```bash
laravel new blog
```

This will create a new project named `blog`, using the latest `laravel/framework` composer package version, in the current directory.

## Building assets and running the project

After the project is created, you can build the assets and run the application to ensure a successful installation.

```bash
cd blog
#show framework version
php artisan -V
#install node modules
npm install
#build public/app.js and public/css.js from the resources directory.
npm run dev
#serve the application on localhost:8000
php artisan serve
```

Navigate to localhost:8000 to see the Laravel home page.

> To check the `Laravel\Framework` package version installed a part of the generated `Laravel\Laravel` project scaffold, open the composer.json file and search for `Laravel\Framework` text to see the package version.

## Installing PHP Redis Extension using PECL

If you intend to use Redis with your Laravel application you will need to install the PHP Redis extension using the PHP PECL CLI.

> The PECL CLI is installed as part of your PHP installation.

```bash
# update the pecl repo
pecl channel-update pecl.php.net
# clear cached extensions
pecl clear-cache
# install the extension (use the --force flag to install latest version even if the extension cache was not cleared)
pecl install --force xdebug
# make sure the extension was installed by listing the installed extensions filtered by xdebug
php -m | grep 'xdebug'
```

Now you should be able to configure and use the redis driver in your Laravel applications.

## Installing Laravel Jetstream using the Laravel CLI

One of the reasons that you may use the Laravel CLI to create a new Laravel project instead of the composer
create-project command, is to use the installer to also create scaffolding for a new project.

For example we can use the installer to create a new Laravel Jetstream project which creates a new Laravel project with
scaffolding that uses the [TALL Stack](https://tallstack.dev) or [InertiaJS](https://inertiajs.com) and can optionally
include `teams` functionality.

To do so we can use variations of the new project command:

```bash
# create a new Laravel jetstream project
laravel new blog --jet
#create a new Laravel jetstream project using the livewire stack
laravel new blog --jet --stack=livewire
#create a new Laravel jetstream project using the livewire stack and teams functionality
laravel new blog --jet --stack=inertia --teams
```

You will be prompted to select the `stack` and `teams` options, when not specified as command line options.

## Resources

[updating-the-laravel-installer](https://laravel-news.com/updating-the-laravel-installer)

[laravel-installer-now-includes-support-for-jetpack](https://laravel-news.com/laravel-installer-now-includes-support-for-jetpack)
