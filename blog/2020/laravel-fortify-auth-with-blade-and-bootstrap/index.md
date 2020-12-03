# laravel fortify auth with blade and bootstrap

[laravel fortify auth with blade and bootstrap](https://aregsar.com/blog/2020/laravel-fortify-auth-with-blade-and-bootstrap)

## Setup Bootstrap 5

Install Bootstrap 5 using NPM:

```bash
#@next since bootstrap 5 is not released yet
#may not need to install popper.js when bootstrap 5 is release
npm i bootstrap@next popper.js sass sass-loader --save-dev
```

Create the application Bootstrap scss file importing bootstrap:

```bash
echo '@import "~bootstrap/scss/bootstrap";' > resources/sass/app.scss
```

Update the `webpack.mix.js` file to compile the scss file:

```js
mix.js('resources/js/app.js','public/js')
   .sass('resources/sass/app.scss','public/css');
   .sourceMaps();
```

In `resources/js/app.js` add:

```js
window.Popper = require("popper.js").default;
require("./bootstrap");
```

Build assets:

```bash
npm run dev
```

Open a new terminal tab and run npm watch to automatically rebuild assets as we update the scss file

```bash
cd myapp
npm run watch
```

## Setup Laravel Fortify

```bash
laravel new myapp
cd myapp
```

```bash
composer require laravel/fortify
```

```bash
php artisan vendor:publish --provider="Laravel\\Fortify\\FortifyServiceProvider"
```

The publish command publishes:

```bash
app/Providers/FortifyServiceProvider.php
config/fortify.php
app/actions/Fortify/xxxaction.php
```

The `app/actions/Fortify` directory actions are registered with Fortify in the boot method of the `FortifyServiceProvider` class as described in the next section.

Register the FortifyServiceProvider in `config/app.php` by adding the `FortifyServiceProvider::class` to the providers array in the `'providers'` array:

```php
'providers' => [
    App\Providers\FortifyServiceProvider::class,
]
```

Now that Fortify is configured lets see what routes it provides for us out of the box

```bash
php artisan route:list
```

## Setup database

Add a docker compose file

```yaml
a
```

Run docker compose to start the database server

```bash
docker-compose up
```

Configure the database settings in the .env file

```ini
db=a
```

Run the artisan migrate command to add the tables

```bash
php artisan migrate
```

To shut down the docker containers run:

```bash
docker-compose down
```

> Note the database data is persisted on local host

## Registering auth actions with the FortifyServiceProvider

FortifyServiceProvider boot method contains Fortify calls to register the auth action classes with Fortify. These action classes encapsulate the authentication logic for each Fortify authentication feature.

Example:

```php
Fortify::createUsersUsing(CreateNewUser::class)
```

## Enabling auth features using Fortify features Config

config/fortify.php

'features' section has and array of Fortify features that we can turn on by adding the feature to the array.

here is the out of the box configuration

```php
'features'=>[
    Features::registration(),
    Features::resetPasswords(),
];
```

## Configuring Blade Views for Fortify

'/' # unauthenticated users
'/home' #dashboard where logged in user is redirected to
#also will redirect to /home when account is registered and user is immidiately logged in but if email verification middleware protects the /home then the logged in user is redirected to the verify email page where they can resent the verification link

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

```html
<input
  class="form-control @error('name') is-invalid @enderror"
  id="name"
  name="name"
  type="text"
  value="{{ old('name') }}"
/>

@error('name')
<span class="invalid-feedback" role="alert"> {{ $message }} </span>
@enderror
```

### Create the Register Blade View

register input names: name, email,password,password_confirmation

```html
<input
  class="form-control @error('name') is-invalid @enderror"
  id="name"
  name="name"
  type="text"
  value="{{ old('name') }}"
/>

@error('name')
<span class="invalid-feedback" role="alert"> {{ $message }} </span>
@enderror
```

### Create the Main Blade Layout View

in welcome.blade.php (create a views/layouts folder
and put in that folder renamed to main.blade.php)

add a meta tag

```html
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
```

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

## Add Reset Password Feature

Fortify::requestPasswordResetLinkView(fn()=> view('auth.forgotpassword'));

//Fortify sets this status message in the session,
//when the send reset link action completes and redirects back to
//the view that has the reset password form:
//"we have emailed your password reset link!"
@if(session('status'))
{{session('status')}}
@endif

---

Fortify::resetPasswordView(fn($request)=>view('auth.resetpassword',['request'=>$request]));

//in auth.resetpassword get values that fortify redirected with
//$request->route('token') could just be $request->token ???
<input type="hidden" name="token" value="{{$request->route('token')}}">
<input type="email" name="email" value="{{request->email}}">

---

//once user class inherits from MustVerifyEmail and we add
the middleware->['auth','verifyemail'] to proteted routes
fortify will redirect to the auth.verifyemail view where we have the resend verification email form

Fortify::verifyEmailView(fn()=> view('auth.verifyemail'));

//form for resending verificaion link

<form method="POST" action="{{route('verification.send')}}">

//show "verification link sent" status message after submiting the form
@if(session('status'))
{{session('status')}}
@endif

## XX
