# Create Laravel Project With Tailwind UI and Auth

April 4, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed newer versions of Laravel 

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

Check your Github repo to see the committed files.

> Note: `composer create-project` also includes the `users` and `failed jobs` database migration files in `database/migrations`

## Add Tailwind UI with Authentication Scaffolding

```bash
composer require laravel-frontend-presets/tailwindcss --dev
php artisan ui tailwindcss --auth
npm install && npm run dev
```

> The `--auth` option adds the `password resets` database migration file in `database/migrations`
> Note: `composer require laravel/ui` and `npm install @tailwindcss/ui` are part of the `laravel-frontend-presets/tailwindcss` preset

## Using the Bootstrap alternative to tailwind

You can use Bootstrap UI with Authentication Scaffolding instead of the previous steps:

```bash
composer require laravel/ui
php artisan ui bootstrap --auth
npm install && npm run dev
```

## Running database migrations to add authentication

Before running the migrations we need to create a `myapp` database using your favorite database manager CLI or GUI. I used TablePlus `https://tableplus.com/` to add a database named `myapp`.

Next we need to add the database user and password settings to `.env`.
Assuming we created a database with default settings:

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=root
DB_PASSWORD=
```

Open `.env` file

Replace:

```ini
DB_DATABASE=laravel
```

With:

```ini
DB_DATABASE=myapp
```

Also replace:

```ini
APP_NAME=Laravel
```

With:

```ini
APP_NAME=Myapp
```

Then run:

```bash
php artisan migrate
```

After you execute the script, you should see:

```bash
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (0.02 seconds)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (0.02 seconds)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (0.01 seconds)
```

## Serving the site

If we are serving the app using Laravel valet, we can navigate with our browser to `myapp.test`:

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome myapp.test
```

If we are serve using `php artisan serve` we can navigate with our browser to `localhost:8000`:

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome localhost:8000
```

## Tailwind configuration files

Finally, below is the content of the `tailwind.config.js` that we need to have to use tailwind.

```js
module.exports = {
  theme: {
    extend: {}
  },
  variants: {},
  plugins: [
    require('@tailwindcss/custom-forms')
  ]
}
```

```js
In webpack.mix.js preset added

mix.js('resources/js/app.js', 'public/js')
   .postCss('resources/css/app.css', 'public/css')
   .tailwind('./tailwind.config.js');
```

```css
#in resources/css/app.css preset added:
@tailwind base;
@tailwind components;
@tailwind utilities;
```
