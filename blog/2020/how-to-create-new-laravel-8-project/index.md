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
