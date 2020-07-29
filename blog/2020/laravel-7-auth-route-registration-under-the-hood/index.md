# Laravel 7 Auth Route Registration Under The Hood

July 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I detail how the Laravel authentication routes are installed using the `Laravel\Ui` package and how installing the package adds configures the authentication routes using the Laravel `Macroable` trait facilities.

## Authentication routes in Laravel UI package

In Laravel 7 the authentication routes were moved to the `Laravel\Ui` package.

The routes are added when we run the artisan ui command to configure one of the ui package flavors with the `--auth` flag:

```bash
php artisan ui tailwindcss --auth
```

After running this command you will find the authentication route registration functions in the `Laravel\Ui\AuthRouteMethods` class in the `Laravel\Ui` package in the `vendors` directory.

Below I have annotated the `auth()` entry point method for auth route registration in the `AuthRouteMethods` class:

```php
//from vendor/laravel/ui package
namespace Laravel\Ui;

class AuthRouteMethods
{
    //This is the method that is eventually called starting from the
    //Auth::routes() call in routes/web.php file
    //It first registers the login/logout routes
    //Next it registers the resgiter route. (This can be turned off via options flag)
    //Then in turn calls the resetPassword and confirmPassword methods by default to
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

> Note: We can copy the routes definitions from these functions and paste them directly into our `routes/web.php` file to create our own route definitions directly in the `routes/web.php` file. Then we can remove the `Auth::routes()` call from the `routes/web.php` file. This way all our routes will be in our application and not burried deep in the Laravel framework code.

## Inspecting the Authentication routs using artisan

We can run the artisan command below to show us the routs that are defined for our application:

```bash
php artisan route:list
```

If we used the `--auth` flag when we ran the `php artisan ui` command, the `php artisan route:list` command will show the route listing below:

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

As you can see the authentication routes are listed.

## Laravel UI package

The Routes registration methods in the `AuthRouteMethods` class from the installed `Laravel\Ui` package are mixed into `Illuminate\Routing\Router` class by the boot method of the `UiServiceProvider` class of the`Laravel\Ui` package.

Below I show how these routes registration methods get added to the `Illuminate\Routing\Router` class by the `UiServiceProvider` class.

The Laravel framework's `Illuminate\Routing\Router` class includes the Laravel `Illuminate\Support\Traits\Macroable` trait. The trait adds a `mixin` method to the `Illuminate\Routing\Router` class. The `Macroable` also adds a `$macros` hash array to `Illuminate\Routing\Router` class.

The code below, from the installed `Laravel\Ui` package, shows how the `mixin` method added to the `Illuminate\Routing\Router` class is called through a call to a `mixin` method on the `Illuminate\Support\Facades\Route` facade to add the route registration methods implemented in the `AuthRouteMethods` class of the `Laravel\Ui` package into the `$macros` array of the `Illuminate\Routing\Router` class.

You will need an understanding of how facades work under the hood to see how calling `Illuminate\Support\Facades\Route::mixin()` ends up calling the `Illuminate\Routing\Router::mixin()` instance method.

I have detailed how Facades work under the hood at [How Laravel Facades Work Under The Hood](https://aregsar.com/blog/2020/how-laravel-facades-work-under-the-hood)

Below is the annotated `boot` method call of the `UiServiceProvider` class in the `Laravel\Ui` package that starts off the process of adding in the route registration methods from the package:

```php
//in vendor/laravel/ui package
namespace Laravel\Ui;

use Illuminate\Support\Facades\Route;
use Illuminate\Support\ServiceProvider;

class UiServiceProvider extends ServiceProvider
{

    public function boot()
    {
        //The Route::mixin() method is called on the Route facade.
        //The mixin method call causes a call to the  static __call method of the Route facade,
        //because a mixin method does not exist on the Route Facade class.
        //
        //The __call method gets an instance of the Router service class
        //from the application container
        //then calls the mixin() method on the Router service class instance, which is
        //actually a call to the static mixin() method of the Macroable trait of
        //the Router service class.(Note: in PHP a static method of a class can be called by an instance
        //of the class)
        //
        //The mixin() method of the Macroable trait first uses reflection to get all methods of
        //the AuthRouteMethods class instance passed to it, then iterates through all the methods, calling each method and puting the closure function
        //returned from the method in the $macros hash array using the name of
        //the method as the hash key and the closure function returned by the method as the value
        Route::mixin(new AuthRouteMethods);
    }
}
```

The boot method calls the `mixin` method of the `Illuminate\Support\Facades\Route` Facade.

Here is the `Illuminate\Support\Facades\Route` Facade `getFacadeAccessor()` implementation that returns the alias string `router`:

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

The base Facade class calls the `getFacadeAccessor()` method and uses the returned `router` string to get an instance of `Illuminate\Routing\Router` class from the application container. Then it calls the `mixin` method that was called on the facade, on the `Illuminate\Routing\Router` class instance.

Here is the `mixin` and `macro` methods of the `Illuminate\Support\Traits\Macroable` trait used by the `Illuminate\Routing\Router` service class:

```php
namespace Illuminate\Support\Traits;

trait Macroable
{
    protected static $macros = [];

    public static function mixin($mixin, $replace = true)
    {
        //get all the methods of the `AuthRouteMethods` class of the `Laravel\Ui` package
        $methods = (new ReflectionClass($mixin))->getMethods(
            ReflectionMethod::IS_PUBLIC | ReflectionMethod::IS_PROTECTED
        );

        foreach ($methods as $method) {
            if ($replace || ! static::hasMacro($method->name)) {
                $method->setAccessible(true);
                //call each of the methods in $methods and put the closure function returned
                //from that method into the $macros array
                static::macro($method->name, $method->invoke($mixin));
            }
        }
    }

    public static function macro($name, $macro)
    {
        static::$macros[$name] = $macro;
    }
}
```

The `mixin` method uses the `macro` method to add the route registration closure functions to the `$macros` hash array.

## Registering the authentication routes installed by the Laravel UI package

Mixing in the authentication routes from the UI package into the `Illuminate\Routing\Router` class is only the first part to setup auth routes for your application that happens when `UiServiceProvider::boot()` method is invoked.

The second part is the actual execution of the mixed in `Illuminate\Routing\Router` class authentication route methods to register the authentication routes. This happens when the `Auth::routes()` is called in the `routing/web.php` file.

Below I show how the `Illuminate\Routing\Router` class's mixed in route registration methods are used to register authentication routes with our application.

It all starts with the `Auth::routes()` call in `routes/web.php`:

```php
//from routes/web.php
Illuminate\Support\Facades\Auth::routes();
```

That call is a call to the `Auth` facade class `Auth::routes` method.
The `Auth::routes` method then calls the `Illuminate\Routing\Router` service class instance `auth()` method which in ultimately creates the Authentication routs.

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

Since the `Illuminate\Routing\Router` class does not define an `auth()` method, the `__call` method of the `Illuminate\Routing\Router` instance is called instead.
This method checks to see if an `auth()` method exists, in its `$macros` array that it inherits from its `Macroable` trait.

It finds that it does exists because it was added to the `macros` array from the `AuthRouteMethods` class from the Laravel UI package by the Laravel UI package provider as described in the previous sections.

Since the `auth()` method exists the `$this->macroCall($method, $parameters)` method, inherited from its `Macroable` trait, is called.

Actually the `macroCall` method is just an alias for the `__call` instance method of the `Macroable` class. The alias is declared inline in the `use Macroable{__call as macroCall;}` trait statement in the `Illuminate\Routing\Router` class shown below:

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

    //when we call a Router instance method that does not exist, this method is called instead of
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

So the `$this->macroCall($method, $parameters)` is actually calling the `__call` method implemented in the `Macroable` trait. The call passes along the `auth` string as method name.

Below I show how the `__call` method of the `Macroable` trait uses the passed in arguments to call the installed `auth()` closure function installed by the Laravel UI package provider.

```php
namespace Illuminate\Support\Traits;

trait Macroable
{

    protected static $macros = [];

    public static function hasMacro($name)
    {
        return isset(static::$macros[$name]);
    }


    public function __call($method, $parameters)
    {
        if (! static::hasMacro($method)) {
            throw new BadMethodCallException(sprintf(
                'Method %s::%s does not exist.', static::class, $method
            ));
        }

        //$method is the method name string `auth`
        //$macro is the auth() closure that was added to the macros array by the Laravel UI package
        $macro = static::$macros[$method];

        if ($macro instanceof Closure) {
            return call_user_func_array($macro->bindTo($this, static::class), $parameters);
        }

        return $macro(...$parameters);
    }
}
```

As we can see the `$macro = static::$macros[$method]` first gets the closure function for the `auth` key and sets it to the `$macro` variable.
Then the `call_user_func_array` function calls the closure by passing it the `$macro` closure and the `$parameters` array of parameters if any. You will notice that before the `$macro` closure is passed to `call_user_func_array` the `bindTo($this, static::class)` method is called on it. This is so that the `$this` pointer used inside any closure is bound to the class instance that the method is called on.
So the method `$macro->bindTo($this, static::class)` is called before calling the `auth` closure so that the `$this` pointer in the closure references the `Illuminate\Routing\Router` class.

Once the `auth` closure is called, its code then calls the other closure functions
`resetPassword`,`confirmPassword`,`emailVerification` (as shown previously in the `AuthRouteMethods::auth` method) that were added into the macros array by the Laravel UI package provider boot method.

Below am showing an abbreviated version of the `AuthRouteMethods` class of the Laravel UI package that provided the closures that were added to the `$macros` array. 
I have annotated it to illustrate how the `$this` pointer binding works.

```php
class AuthRouteMethods
{
    //This is the `auth` string key added to the $macros array
    public function auth()
    {
        //This is the closure added to the $macros array that corresponds to `auth` string key
        return function ($options = []) {

            //using the $this pointer bound to `Illuminate\Routing\Router` instance 
            //to call the `get` method defined in `Illuminate\Routing\Router`
            $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
  
            if ($options['reset'] ?? true) {

                //using the $this pointer bound to `Illuminate\Routing\Router` instance
                //to call resetPassword which is actually a call to the `Illuminate\Routing\Router` class
                //Macroable trait __call method, since the resetPassword method does not exist on `Illuminate\Routing\Router`
                $this->resetPassword();
            }
        };
    }

    //This is the `resetPassword` string key added to the $macros array
    public function resetPassword()
    {
        //This is the closure added to the $macros array that corresponds to `resetPassword` string key
        return function () {
            //using the $this pointer bound to `Illuminate\Routing\Router` instance
            //to call the `get` method defined in `Illuminate\Routing\Router`
            $this->get('password/reset', 'Auth\ForgotPasswordController@showLinkRequestForm')->name('password.request');
        };
    }
}
```

So when the `auth` closure calls the other route registration methods using the `$this` pointer that was bound to the `Illuminate\Routing\Router` class, it re-enters the `__call` of the `Macroable` trait method and finds each of those other closures from the Laravel UI package `AuthRouteMethods` class that were added to the `$macros` array and calls them.

## The full Macroable trait implementation

It is useful in general to understand how the Laravel `Macroable` trait allows application and package developers to add additional callable methods to existing framework classes. This allows us to extend the functionality of the existing classes by using third party packages or using custom application code.

Most core framework classes already include the `Macroable` trait allowing us to easily add methods to these classes.

However we are not limited to framework classes. We can add the capability of extending an existing class with additional methods by adding the `Macroable` trait to that class.

Also, framework classes that do not implement the `Macroable` trait can be extended so that we can include the `Macroable` trait in the extended class.

Below I list the full Laravel 7 `Macroable` trait source:

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

The way the `Macroable` trait works is that when included in a class, it adds a static `$macros` hash array to the class. The `Macroable` trait also adds the `__call` and  `__callStatic` dynamic call methods to the class.

Closure functions can be added to the `$macros` hash array using the function name as the array item key.
These functions are the additional methods we want to add to the class. The functions are usually added to the `$macros` hash array in a Laravel provider boot method by calling the `static function macro($name, $macro)` method also provided by the `Macroable` trait.

When we call an instance method on the class that includes the `Macroable` trait and that method does not exist on the class, then the `__call` method will be called with the method name and arguments. The `__call` method will use the method name to retrieve the closure function from the `$macros` array if it exists. Then it will bind the closure to the class instance that the method was called on. It does this so that the `$this` pointer used within the closure function body references the class instance. 
After binding the closure, it will invoke the closure to execute the method call.

When we call a static method on the class that includes the `Macroable` trait and that method does not exist the `__callStatic` method is called and it also retrieves a closure corresponding to the method name from the `$macros` array, if it exists, then invokes the closure. The difference between `__callStatic` and `__call` methods of the `Macroable` trait is that the `__callStatic` method does not bind to the `$this` pointer since it was triggered by a static method call which has no `$this` pointer.

I explained above how the trait invokes closures from the `$macros` array but how do the closures get into the array in the first place?

Well there are two ways that the trait allows us to add the additional methods that we want to be able to call on the class that includes the trait.

The first way is by directly calling the `macro($name, $macro)` method of the `Macroable` trait.

We can call this method and pass the name of the method we want to add as a string and also passing in the closure that we want to be invoked when this method is called on the class.

Note that there is no way to specify if the closure we are adding should be called as a static or instance method. If the method is called on an instance of the class that includes the `Macroable` trait then the closure will be called as an instance method. If the method is called as a static method on the class then the closure will be called as a static method.

The second way to add methods to the class that implements the trait is to define a separate class that defines one or more methods that each return a closure. The name of the method in this class will be used as the key in the `$macros` array and the closure the method returns will be the corresponding closure that is added to `$macros` array.
The `Macroable` trait has a `mixin($mixin, $replace = true)` method that allows us to mix in all the closures returned by the methods of this separate class into the `$macros` array.

The way the `mixin` method works is that we pass it the string name of the class that has the closures we want to add, as the first argument to the  `mixin` method.

The method then uses reflection to get all the methods in the class,
It then iterates over all the methods, calling the `macro($name, $macro)` method for each method in the iteration. It uses the name of each method as the `$name` argument and calls the method to return the closures that it passes as the `$macro` argument.

The `mixin` method also takes a second boolean argument that indicates whether it should replace any existing closures that have the same key that are already in the `$macros` array.
