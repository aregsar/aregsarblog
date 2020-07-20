# Laravel 7 Auth Route Registration Under The Hood

July 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

July 16, 2020

[Laravel 7 Auth Route Registration Under The Hood](https://aregsar.com/blog/2020/laravel-7-auth-route-registration-under-the-hood)

## Authentication routes in Laravel\UI package

In Laravel 7 the authentication routes were moved to the `Laravel\Ui` package.

The routes are added when we run the artisan ui command to configure one of the ui package flavors with the `--auth` flag:

```bash
php artisan ui tailwindcss --auth
```

After running this command we will find the authentication route registration functions in the `Laravel\Ui\AuthRouteMethods` class in the `Laravel\Ui` package under the `vendors` directory.

```php
//from vendor/laravel/ui package
namespace Laravel\Ui;

class AuthRouteMethods
{
    /**
     * Register the typical authentication routes for an application.
     *
     * @param  array  $options
     * @return void
     */
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
                class_exists($this->prependGroupNamespace('Auth\ConfirmPasswordController'))) {
                $this->confirmPassword();
            }

            // Email Verification Routes...
            if ($options['verify'] ?? false) {
                $this->emailVerification();
            }
        };
    }

    /**
     * Register the typical reset password routes for an application.
     *
     * @return void
     */
    public function resetPassword()
    {
        return function () {
            $this->get('password/reset', 'Auth\ForgotPasswordController@showLinkRequestForm')->name('password.request');
            $this->post('password/email', 'Auth\ForgotPasswordController@sendResetLinkEmail')->name('password.email');
            $this->get('password/reset/{token}', 'Auth\ResetPasswordController@showResetForm')->name('password.reset');
            $this->post('password/reset', 'Auth\ResetPasswordController@reset')->name('password.update');
        };
    }

    /**
     * Register the typical confirm password routes for an application.
     *
     * @return void
     */
    public function confirmPassword()
    {
        return function () {
            $this->get('password/confirm', 'Auth\ConfirmPasswordController@showConfirmForm')->name('password.confirm');
            $this->post('password/confirm', 'Auth\ConfirmPasswordController@confirm');
        };
    }

    /**
     * Register the typical email verification routes for an application.
     *
     * @return void
     */
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
        __call as macroCall;
    }

    public function __call($method, $parameters)
    {
        if (static::hasMacro($method)) {
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
        //The mixin method called on the Route facade
        //uses the Route facade to return an instance of the Router class
        //from the application container via dynamic __call
        //then calls mixin method on the Router instance which is
        //actually a call to the static mixin method of the Macroable trait of 
        //the Router class 
        //the mixin method calls each method of the AuthRouteMethods
        //instance given to the mixin method and puts the closure function
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



