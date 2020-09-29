# How To Create New Laravel 8 Project

September 29, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[How To Create New Laravel 8 Project](https://aregsar.com/blog/2020/how-to-create-new-laravel-8-project)

In this post I will show two ways in which you can create a new Laravel 8 project.

> It is assumed that you already have PHP and Composer installed. I have posts how to get them both installed on this blog.

## Using Composer Create Project command

The easiest way to create a basic Laravel application is the composer create-project command.

```bash
composer create-project --prefer-dist laravel/laravel blog
```

This will create a new project, named `blog`, in the current directory.
It will use the latest version of the `laravel/laravel` composer package.

To use a different version of the composer package we just add the package version:

```bash
composer create-project --prefer-dist laravel/laravel:8.* blog
```

This will create a project with the latest version of Laravel 8.

After the project is installed you can build the assets and run the application to ensure a successful installation.

```bash
cd blog
#install npm modules
npm install
#build public/app.js and public/css.js from the resources directory.
npm run dev
#serve the application on localhost:8000
php artisan serve
```

Navigate to localhost:8000 to see the Laravel home page.

## Using the Laravel Installer CLI

First we need to install the latest version of the Laravel installer:

```bash
#uninstall current installed version if any
#(optional - has no effect if there is'nt an installed version)
composer global remove laravel/installer
#install the latest version of the installer
composer global require laravel/installer
```

Check the installed version:

```bash
laravel --version
```

The installer is installed globally in my MacBook `~/.composer` directory.

I can see it installed by dumping the content of the `~/.composer/composer.json` file:

```bash
cat ~/.composer/composer.json
```

We can see the `laravel/installer` dependency along with its version.

```json
{
  "require": {
    "laravel/valet": "^2.11",
    "laravel/installer": "^4.0"
  }
}
```

Now that the installer is installed, we can create a new Laravel project by running:

```bash
laravel new blog
```

This will create a project with the latest version of Laravel 8.

After the project is installed you can build the assets and run the application to ensure a successful installation.

```bash
cd blog
#install npm modules
npm install
#build public/app.js and public/css.js from the resources directory.
npm run dev
#serve the application on localhost:8000
php artisan serve
```

Navigate to localhost:8000 to see the Laravel home page.

## Insalling Laravel Jetstream using the Laravel installer

One of the reasons that you may use the laravel installer to create a new Laravel project instead of the composer
create-project command is to use the installer to also create scaffolding for a new project.

For example we can use the installer to create a new Laravel Jetstream project which creatres a new Laravel project with
scaffolding that uses the [TALL](https://laravel-news.com/updating-the-laravel-installer) stack.

To do so we can use variations of the new project command:

```bash
# create a new Laravel jetstream project (defaults to livewire)
laravel new blog --jet
#create a new Laravel jetstream project using the livewire stack
laravel new blog --jet --stack=livewire
#create a new Laravel jetstream project using the livewire stack and teams functionality
laravel new blog --jet --stack=livewire --teams
# create a new Laravel jetstream project that uses the inertia stack (alternative to TALL stack)
laravel new blog --jet --stack=inertia
```

## Resources

https://laravel-news.com/updating-the-laravel-installer
