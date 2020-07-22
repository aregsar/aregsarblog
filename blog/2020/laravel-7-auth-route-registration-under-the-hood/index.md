# Laravel 7 Auth Route Registration Under The Hood

July 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

July 16, 2020

[Laravel 7 Auth Route Registration Under The Hood](https://aregsar.com/blog/2020/laravel-7-auth-route-registration-under-the-hood)

Starts with the auth call on the router class instance
It does not have that method
So it calls the dynamic call instance method
Call method calls the auth method of the ui package authroutes class auth method
That was plugged into the router class using the macroable trait
To understand how this happens we first need a detour
First we have to know about dynamic missing method call static and instance
Then learn about facades
Then about macroable

## Authentication routes in Laravel\UI package

In Laravel 7 the authentication routes were moved to the `Laravel\Ui` package.

The routes are added when we run the artisan ui command to configure one of the ui package flavors with the `--auth` flag:

```bash
php artisan ui tailwindcss --auth
```

After running this command we will find the authentication route registration functions in the `Laravel\Ui\AuthRouteMethods` class in the `Laravel\Ui` package under the `vendors` directory.

Below I have annotated the `auth()` entry point method for auth route registration in the `AuthRouteMethods` class:

```php
//from vendor/laravel/ui package
namespace Laravel\Ui;

class AuthRouteMethods
{
    //This is the method that is eventually called starting from the
    //Auth::routes() call in routes/web.php
    //It first registers the login/logout routes
    //Next it registers the resgiter route. (This can be turned off via options flag)
    //It in turn calls the resetPassword and confirmPassword methods by default to 
    //register routes for those features. (Each of these calls can be turned off via options flag)
    //Finally it calls the emailVerification method only if it is explicitly turned on via options flag
    //to register email verification routes
    public function auth()
    {
        return function ($options = []) {
            // Authentication Routes...
            $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
            $this->post('login', 'Auth\LoginController@login');
            $this->post('logout', 'Auth\LoginController@logout')->name('logout');

            // Registration Routes...
            if ($options['register'] ?? true) {
                $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
                $this->post('register', 'Auth\RegisterController@register');
            }

            // Password Reset Routes...
            if ($options['reset'] ?? true) {
                $this->resetPassword();
            }

            // Password Confirmation Routes...
            if ($options['confirm'] ??
                //call confirmPassword to register routes only if Auth\ConfirmPasswordController exists
                class_exists($this->prependGroupNamespace('Auth\ConfirmPasswordController'))) {
                $this->confirmPassword();
            }

            // Email Verification Routes...
            if ($options['verify'] ?? false) {
                $this->emailVerification();
            }
        };
    }

    public function resetPassword()
    {
        return function () {
            $this->get('password/reset', 'Auth\ForgotPasswordController@showLinkRequestForm')->name('password.request');
            $this->post('password/email', 'Auth\ForgotPasswordController@sendResetLinkEmail')->name('password.email');
            $this->get('password/reset/{token}', 'Auth\ResetPasswordController@showResetForm')->name('password.reset');
            $this->post('password/reset', 'Auth\ResetPasswordController@reset')->name('password.update');
        };
    }

    public function confirmPassword()
    {
        return function () {
            $this->get('password/confirm', 'Auth\ConfirmPasswordController@showConfirmForm')->name('password.confirm');
            $this->post('password/confirm', 'Auth\ConfirmPasswordController@confirm');
        };
    }

    public function emailVerification()
    {
        return function () {
            $this->get('email/verify', 'Auth\VerificationController@show')->name('verification.notice');
            $this->get('email/verify/{id}/{hash}', 'Auth\VerificationController@verify')->name('verification.verify');
            $this->post('email/resend', 'Auth\VerificationController@resend')->name('verification.resend');
        };
    }
}
```

> Note: We can copy the routes registered in these functions to create our own route definitions directly in the `routes/web.php` file and remove the `Auth::routes()` call in the `routes/web.php` file. This way all our routes will be in the our application and not in the framework code.

Below I show how the routes from the installed `Laravel\Ui` package get added to the `Illuminate\Routing\Router` class by the framework.

It all starts with the `Auth::routes()` call in `routes/web.php`:

```php
//from routes/web.php
Illuminate\Support\Facades\Auth::routes();
```

The Auth facade in `routes/web.php` calls the `Illuminate\Routing\Router` service classes `auth()` method which in turn creates the Authentication routs.

```php
class Auth extends Facade
{
    public static function routes(array $options = [])
    {
        //calls the Illuminate Router service class auth() method
        static::$app->make('router')->auth($options);
    }
}
```

The `auth()` method of the `Illuminate\Routing\Router` class will end up calling the `auth()`,`resetPassword()`,`confirmPassword()` and `emailVerification()` methods of the `Laravel\Ui\AuthRouteMethods` class from the `Laravel\Ui` package. Those methods all return a function that when invoked will register the routs. Each of the methods returns a function that when invoked will register the routs for a specific authentication feature.

> Note: the emailVerification() method of the `Laravel\Ui\AuthRouteMethods` class will only be called if we pass the `['verified' => true]` option to `routes` method of the `Auth` facade class in `routes/web.php` that trickles down to the authentication route registration functions defined in `Laravel\Ui\AuthRouteMethods` class

If we look at the source code of the `Illuminate\Routing\Router` service class we don't see an `auth()` method. This is because the `auth()` method is added dynamically to the `Illuminate\Routing\Router` class using the dynamic `__call` method that is called whenever we call a instance method on a class that does not have the called method implemented.

```php
namespace Illuminate\Routing;

use Illuminate\Support\Traits\Macroable;

class Router implements BindingRegistrar, RegistrarContract
{
    use Macroable {
        //overrides the __call method name inside the Macroable trait to macroCall
        //so that it does not conflict with the __call method name inside this Router class
        __call as macroCall;
    }

    //when we call an Router instance method that does not exist, this method is called instead of
    //the __call method of the Macroable trait included above. This is because this method hides the
    //__call method of the included Macroable trait.
    //That is why we changed the __call method name of the Macroable trait to macroCall
    public function __call($method, $parameters)
    {
        if (static::hasMacro($method)) {
            //actually calls the __call method of the Macroable trait because we overrode
            //__call as macroCall in the Macroable trait included above
            return $this->macroCall($method, $parameters);
        }

        if ($method === 'middleware') {
            return (new RouteRegistrar($this))->attribute($method, is_array($parameters[0]) ? $parameters[0] : $parameters);
        }

        return (new RouteRegistrar($this))->attribute($method, $parameters[0]);
    }
}

```

## Inspecting the Authentication routs using artisan

We can also run the artisan command to show us the routs that are defined for authentication:

```bash
php artisan route:list
```

The command will show the route listing below:

```bash
+--------+----------+------------------------+------------------+------------------------------------------------------------------------+--------------+
| Domain | Method   | URI                    | Name             | Action                                                                 | Middleware   |
+--------+----------+------------------------+------------------+------------------------------------------------------------------------+--------------+
|        | GET|HEAD | /                      | home.index       | App\Http\Controllers\HomeController@index                              | web          |
|        | GET|HEAD | api/user               |                  | Closure                                                                | api,auth:api |
|        | GET|HEAD | home                   | home.welcome     | App\Http\Controllers\HomeController@welcome                            | web,auth     |
|        | GET|HEAD | login                  | login            | App\Http\Controllers\Auth\LoginController@showLoginForm                | web,guest    |
|        | POST     | login                  |                  | App\Http\Controllers\Auth\LoginController@login                        | web,guest    |
|        | POST     | logout                 | logout           | App\Http\Controllers\Auth\LoginController@logout                       | web          |
|        | GET|HEAD | password/confirm       | password.confirm | App\Http\Controllers\Auth\ConfirmPasswordController@showConfirmForm    | web,auth     |
|        | POST     | password/confirm       |                  | App\Http\Controllers\Auth\ConfirmPasswordController@confirm            | web,auth     |
|        | POST     | password/email         | password.email   | App\Http\Controllers\Auth\ForgotPasswordController@sendResetLinkEmail  | web          |
|        | GET|HEAD | password/reset         | password.request | App\Http\Controllers\Auth\ForgotPasswordController@showLinkRequestForm | web          |
|        | POST     | password/reset         | password.update  | App\Http\Controllers\Auth\ResetPasswordController@reset                | web          |
|        | GET|HEAD | password/reset/{token} | password.reset   | App\Http\Controllers\Auth\ResetPasswordController@showResetForm        | web          |
|        | GET|HEAD | register               | register         | App\Http\Controllers\Auth\RegisterController@showRegistrationForm      | web,guest    |
|        | POST     | register               |                  | App\Http\Controllers\Auth\RegisterController@register                  | web,guest    |
+--------+----------+------------------------+------------------+------------------------------------------------------------------------+--------------+
```

## Laravel UI package

The code below from the installed `Laravel\Ui` package shows how the `mixin` method of the `Illuminate\Support\Traits\Macroable` trait included in the `Illuminate\Routing\Router` class is called through the `Illuminate\Support\Facades\Route` facade to ultimately mix in the route registration methods implemented in the `AuthRouteMethods` class of the `Laravel\Ui` package into the `Illuminate\Routing\Router` class implemented in the core Laravel framework.

```php
//in vendor/laravel/ui package
namespace Laravel\Ui;

use Illuminate\Support\Facades\Route;
use Illuminate\Support\ServiceProvider;

class UiServiceProvider extends ServiceProvider
{

    public function boot()
    {
        //The  Route::mixin() method called on the Route facade
        //causes a call to the  static __call method of the Route facade
        //because a mixin() method does not exist on the Route Facade class
        //the __call method gets an instance of the Router service class
        //from the application container
        //then calls mixin() method on the Router service class instance which is
        //actually a call to the static mixin() method of the Macroable trait of
        //the Router service class.(Note: in PHP a static method of a class can be called by an instance
        //of the class)
        //the mixin() method of the Macroable trait then calls each method of the AuthRouteMethods
        //instance given to the mixin method (using reflection) and puts the closure function
        //returned from each method in the macros hash array using the name of
        //the method that returns the closue as the hash key
        Route::mixin(new AuthRouteMethods);
    }
}
```

```php
namespace Illuminate\Support\Facades;
class Route extends Facade
{

    protected static function getFacadeAccessor()
    {
        return 'router';
    }
}
```

```php
namespace Illuminate\Filesystem;

use Illuminate\Support\Traits\Macroable;

class Filesystem
{
    use Macroable;
}
```

```php
namespace Illuminate\Support\Traits;

trait Macroable
{

    protected static $macros = [];


    public static function macro($name, $macro)
    {
        static::$macros[$name] = $macro;
    }

  
    public static function mixin($mixin, $replace = true)
    {
        $methods = (new ReflectionClass($mixin))->getMethods(
            ReflectionMethod::IS_PUBLIC | ReflectionMethod::IS_PROTECTED
        );

        foreach ($methods as $method) {
            if ($replace || ! static::hasMacro($method->name)) {
                $method->setAccessible(true);
                static::macro($method->name, $method->invoke($mixin));
            }
        }
    }

    public static function hasMacro($name)
    {
        return isset(static::$macros[$name]);
    }

    public static function __callStatic($method, $parameters)
    {
        if (! static::hasMacro($method)) {
            throw new BadMethodCallException(sprintf(
                'Method %s::%s does not exist.', static::class, $method
            ));
        }

        $macro = static::$macros[$method];

        if ($macro instanceof Closure) {
            return call_user_func_array(Closure::bind($macro, null, static::class), $parameters);
        }

        return $macro(...$parameters);
    }

    public function __call($method, $parameters)
    {
        if (! static::hasMacro($method)) {
            throw new BadMethodCallException(sprintf(
                'Method %s::%s does not exist.', static::class, $method
            ));
        }

        $macro = static::$macros[$method];

        if ($macro instanceof Closure) {
            return call_user_func_array($macro->bindTo($this, static::class), $parameters);
        }

        return $macro(...$parameters);
    }
}
```



