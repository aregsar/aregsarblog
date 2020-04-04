# Create Laravel7 Project With Tailwind UI and Auth

April 4, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Create Laravel7 Project With Tailwind UI and Auth](https://aregsar.com/blog/2020/create-laravel7-project-with-tailwind-ui-and-auth)


composer create-project --prefer-dist laravel/laravel myapp

cd myapp && php artisan --version

#require the UI scaffolding
composer require laravel/ui
php artisan ui --help
php artisan ui bootstrap --auth

npm install
npm run dev
php artisan migrate
php artisan serve

###########################

composer create-project --prefer-dist laravel/laravel myapp

cd myapp && php artisan --version

#require the tailwindcss preset
#The preset also require laravel/ui so no need to do `composer require laravel/ui`
composer require laravel-frontend-presets/tailwindcss --dev
php artisan ui --help
php artisan ui tailwindcss --auth
npm install @tailwindcss/ui

// add to tailwind.config.js 
module.exports = 
{
   plugins: [
      require('@tailwindcss/ui'),
   ]
}

npm install
npm run dev
php artisan migrate
php artisan serve


<link rel="stylesheet" href="https://rsms.me/inter/inter.css">
// tailwind.config.js
module.exports = {
   theme: {
     extend: {
      fontFamily: { sans: ['Inter var'] },
     }
   },
   variants: {},
   plugins: [
     require('@tailwindcss/ui'),
   ]
}



