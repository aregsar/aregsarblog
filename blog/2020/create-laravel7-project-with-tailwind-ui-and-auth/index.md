# Create Laravel7 Project With Tailwind UI and Auth

April 4, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Create Laravel7 Project With Tailwind UI and Auth](https://aregsar.com/blog/2020/create-laravel7-project-with-tailwind-ui-and-auth)

https://grrr.tech/posts/installing-homebrew-php-extensions-with-pecl/

composer create-project --prefer-dist laravel/laravel myapp

cd myapp && php artisan --version

#require the UI scaffolding
composer require laravel/ui
php artisan ui --help
php artisan ui bootstrap --auth

npm install && npm run dev

#need to create a myapp database using tableplus 
#and add database user and password setting from tableplus to .env file  
php artisan migrate

#test serve on localhost:8000
#or if valet is parked then just open myapp.test
php artisan serve 

###########################

composer create-project --prefer-dist laravel/laravel myapp

cd myapp && php artisan --version

#require the tailwindcss preset
#The preset also require laravel/ui so no need to do `composer require laravel/ui` 
composer require laravel-frontend-presets/tailwindcss --dev
php artisan ui --help
php artisan ui tailwindcss --auth

#no need to do `npm install @tailwindcss/ui` because it was done as part of the preset



npm install && npm run dev
php artisan migrate
php artisan serve


#####################

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
