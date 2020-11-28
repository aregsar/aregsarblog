# laravel fortify auth with blade and bootstrap

[laravel fortify auth with blade and bootstrap](https://aregsar.com/blog/2020/laravel-fortify-auth-with-blade-and-bootstrap)

php artisan route:list --compact

## Setup Laravel Fortify

```bash
laravel new myapp
cd mayapp
composer require laravel/fortify
php artisan vendor:publish --provider="Laravel\\Fortify\\FortifyServiceProvider"
php artisan migrate
```

Register the FortifyServiceProvider in `config/app.php` by adding the `FortifyServiceProvider::class` to the providers array in `'providers'` section :

```php
'providers' => [
    App\Providers\FortifyServiceProvider::class,
]
```

### Fortify Published Files

The publish command publishes the app/Providers/FortifyServiceProvider.php and config/fortify.php files used to configure Fortify.

The publish command also publishes Fortify auth action classes to the app/actions/Fortify directory. These actions are registered with Fortify as described in the boot method of the FortifyServiceProvider as described in the next section.

### Registering auth actions with the FortifyServiceProvider

FortifyServiceProvider boot method contains Fortify calls to register the auth action classes with Fortify. These action classes encapsulate the authentication logic for each Fortify authentication feature.

Example:

```php
Fortify::createUsersUsing(CreateNewUser::class)
```

### Enabling auth features using Fortify features Config

config/fortify.php

'features' section has and array of Fortify features that we can turn on by adding the feature to the array.

here is the out of the box configuration

```php
'features'=>[
    Features::registration(),
    Features::resetPasswords(),
];
```

## Setup Bootstrap 5

```bash
npm i bootstrap@next popper.js sass sass-loader --save-dev
#@next since bootstrap 5 is not released yet
#may not need to install popper.js when bootstrap 5 is released
```

in webpack.mix.js

```js
mix.js'resources/js/app.js','public/js')
   .sass('resources/sass/app.scss','public/css');
   .sourceMaps();
```

in the project

create the resources > sass > app.scss file
open the file and add on first line:

```scss
@import "~bootstrap/scss/bootstrap";
```

```bash
#compile bootstrap
npm run dev

#open another terminal tab and run
#instead of constantly running npm run dev to rebuild assets
npm run watch
```

## Configuring Blade Views for Fortify

in app/providers/FortifyServiceProvider.php
in the register() method add:

```php
Fortify::registerView(fn()=> view('auth.register'));

Fortify::loginView(fn()=> view('auth.login'));
```

### Create the Register Blade View

create views/auth/register.blade.php
create views/auth/login.blade.php
in the register view make sure bootstrap form input tags
have a name attribute set to name,email.password.password_confirmation

<input name="name" type="text" @error('name') is-invalid @enderror class="form-control" id="name" value="{{old('name')}}">

and add:
@error('name')
<span class="invalid-feedback" role="alert">
{{ $message }}
</span>
@enderror

### Create the Register Blade View

TBD

### Create the Main Blade Layout View

in welcome.blade.php (create a views/layouts folder
and put in that folder renamed to main.blade.php)

add a meta tag

<meta name="csrf-token" content="{{csrf_token()}}">
<title>{{config('app.name')}}</title>
<link href="{{asset('css/app.css')}}" rel="stylesheet">
<script src="{{asset('js/app.js')}}" defer></script>

    @auth


    <a href="{{route('logout')}}"
    	onclick="event.preventDefault();
    	document.getElementById('logout-form').submit();">logout</a>

    <form id-"logout-form"
    	action="{{route('logout')}}"
    	method="POST"
    	style="display:none">@csrf</form>

    @else
    	<a href="{{route('login')}}">login</a>
    	<a href="{{route('register')}}">register</a>

    @endif

<main class="container">
	@yield('content')
</main>

### Create the index blade view

create views/home/index.blade.php
@extends('layouts.main')

@section('content')

<h1>Hello</h1>
@endsection

Add route to index view

in routs.php

```php
Route::get('/', fn() => view('index'));
```
