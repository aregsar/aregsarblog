# Create Laravel 7 Project With Tailwind UI and Auth

April 4, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Create Laravel7 Project With Tailwind UI and Auth](https://aregsar.com/blog/2020/create-laravel7-project-with-tailwind-ui-and-auth)

## Installation steps

> before following instructions here, create an empty Github repository named myapp 

Then cd into your local development folder (where you have used valet park command) and type the following:

```bash
composer create-project --prefer-dist laravel/laravel myapp
cd myapp && php artisan --version
git init
git add -A
git commit -m "first commit"
git remote add origin git@github.com:aregsar/myapp.git
git push -u origin master
```

Check your github repo to see the committed files.

> Note: `composer create-project` also includes the `users` and `failed jobs` database migration files in `database/migrations`


## Add Bootsrap UI and Authentication Scaffolding

```bash
composer require laravel/ui
php artisan ui bootstrap --auth
npm install && npm run dev
```

> The `--auth` option adds the `password resets` database migration file in `database/migrations`

## Add Tailwind UI and Authentication Scaffolding

```bash
composer require laravel-frontend-presets/tailwindcss --dev
php artisan ui tailwindcss --auth
npm install && npm run dev
```

> The `--auth` option adds the `password resets` database migration file in `database/migrations`
> Note: No need to do `composer require laravel/ui` since the tailwind UI preset does this.
> no need to do `npm install @tailwindcss/ui` because it was done as part of the preset

## Running database migrations to add authentication

Before running the migrations we need to create a `myapp` database using `tableplus` and add database user and password settings to `.env`

```bash
php artisan migrate
```

## Serving the site

serve using artisan

```bash
php artisan serve
#open chrome localhost:8000
```

Serve using valet 

```bash
#open chrome myapp.test
```

## Tailwind configuration files

#in tailwind.config.js preset added
module.exports = {
  theme: {
    extend: {}
  },
  variants: {},
  plugins: [
    require('@tailwindcss/custom-forms')
    #,require('@tailwindcss/ui'), #instead of ui has custom-forms? 
  ]
}


#in webpack.mix.js preset added
mix.js('resources/js/app.js', 'public/js')
   .postCss('resources/css/app.css', 'public/css')
   .tailwind('./tailwind.config.js');

#in resources/css/app.css preset added:
@tailwind base;
@tailwind components;
@tailwind utilities;
