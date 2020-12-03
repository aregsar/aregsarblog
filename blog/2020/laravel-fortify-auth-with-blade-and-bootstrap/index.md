# laravel fortify auth with blade and bootstrap

[laravel fortify auth with blade and bootstrap](https://aregsar.com/blog/2020/laravel-fortify-auth-with-blade-and-bootstrap)

## Create a new Laravel 8 poject

Create the Laravel 8 application using the Laravel CLI:

```bash
laravel new myapp
cd myapp
```

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

In the next post we will setup Laravel Fortify for the project

## Setup Laravel Fortify

In the previous article we setup a new Laravel 8 project and installed Bootstrap 5 in the project. You can find the post here:

[laravel fortify auth with blade and bootstrap](https://aregsar.com/blog/2020/laravel-fortify-auth-with-blade-and-bootstrap)

In this post we will install and setup Laravel Fortify in the project:

First we install Laravel Fortify package

```bash
cd myapp
composer require laravel/fortify
```

Next we publish the FortifyServiceProvider

```bash
php artisan vendor:publish --provider="Laravel\\Fortify\\FortifyServiceProvider"
```

The publish command publishes the following files:

```bash
app/Providers/FortifyServiceProvider.php
config/fortify.php
app/actions/Fortify/xxxaction.php
```

> The `app/actions/Fortify` directory actions are registered with Fortify in the boot method of the `FortifyServiceProvider` class as described in the next section.

Next we register the `FortifyServiceProvider` in `config/app.php` by adding the `FortifyServiceProvider::class` to the `'providers'` array:

```php
'providers' => [
    App\Providers\FortifyServiceProvider::class,
]
```

Now that Fortify is configured lets see what routes it provides for us out of the box

```bash
php artisan route:list
```

## Setup the database

We need to add the Laravel authentication and Fortify tables. To do that we need to stand up a MySQL database and run migrations against it By performing the following steps:

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

> To shut down the docker containers run `docker-compose down`. Because we configured a Docker Volume, the database data is persisted on local host after the containers are shut down and will be reloaded once we startup the database container again.

## Registering auth actions with the FortifyServiceProvider

FortifyServiceProvider boot method contains Fortify calls to register the auth action classes with Fortify. These action classes encapsulate the authentication logic for each Fortify authentication feature.

Example:

```php
Fortify::createUsersUsing(CreateNewUser::class)
```

These actions will be used in the Fortify Featurs that we implement in future posts

## Enabling auth features using Fortify features Config

The features that we want to enable are listed in the `'features'` array of the `config/fortify.php` file

Here is the out of the box configuration:

```php
'features'=>[
    Features::registration(),
    Features::resetPasswords(),
];
```

We will update this array with additional features that we will implement in the following posts.

In the next post we will setup the registration and login features of Fortify

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

## Add Two Factor Authentication Feature

```php
use Laravel\Fortify\TwoFactorAuthenticatable;

//class User extends Authenticatable implements MustVerifyEmail
class User extends Authenticatable
{
    use HasFactory, Notifiable,  TwoFactorAuthenticatable;
}
then add the two factor fields to the hidden array of the model

protected $hidden = [
    'two_factor_recovery_codes',
    'two_factor_secret',
];

```

===========
//put this on a dedicated page where user can go to enable two factor authentication
//enable the confirm password middleware on route that renders this view to protect

@if(! auth()->user()->two_factor_secret)
Enable two factor

<form method="post" action="{{ url('user/two-factor-authentication') }}">
@csrf
<button type="submit" class="btn btn-primary">enable</button>
</form>
@else
two factor enabled
<form method="post" action="{{ url('user/two-factor-authentication') }}">
@csrf
@method('DELETE')
<button type="submit" class="btn btn-primary">disable</button>
</form>

    //why do we need to check status if we check two_factor_secret above
    //because the session status only happens when we first enable the 2fa
    //how to regenerate new recovery codes(disable and re-enable two factor?)
    @if(session('status') === 'two-factor-authentication-enabled')
        enabled please scan QR code
        {!! auth()->user()->twoFactorQrCodeSvg() !!}
        //factor out $authenticatedUser = auth()->user()
        //the route to the action that displays this page must use the auth or verified middleware
        //so we don't have to check if auth user exists in the view
        // $codesArray = json_decode(decrypt($authenticatedUser->two_factor_recovery_codes,true))
        @foreach(json_decode(decrypt($authenticatedUser->two_factor_recovery_codes,true)) as $code)
            {{trim($code)}} <br>
        @foreach
    @endif

@endif

=============
//view that asks for two factor code or recovery code after submitting the login form
Fortify::twoFactorChallengeView(fn()=>view('auth.two-factor-challenge'));

//put this in the auth.two-factor-challenge view
//this form is only for posting the two factor code

<form method="post" action="{{ url('/two-factor-challenge') }}"
@csrf
<input type="text" name="code" >

//from auth.two-factor-challenge view have a link that shows a separate view to
//post the a 2fa recovery code if we dont have the 2fa codetead of code.
//only difference is the input name is recovery_code instead of code, but the url posted
//to is the same

<form method="post" action="{{ url('/two-factor-challenge') }}"
@csrf
<input type="text" name="recovery_code" >

=============
//setup confirmpassword view
Fortify::confirmPasswordView(fn()=>view('auth.password-confirm'));

in the auth.password-confirm view

<form action="{{url('user/confirm-password')}}">
<input type="password" name="password" id="password">

## Password Confirmation For Deleting Accounts

fortify does not have a password confirm feature but when we add the password.confirm middleware and register a passwordconfirm view and add the view and add a deleteUserControllers and apply the middleware to it then it works

fortify works with any xxxcontroller that we add that has the password.confirm middleware to present the registered password confirm view to the user
