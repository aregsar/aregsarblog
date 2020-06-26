# How To Queue Laravel User Verification Email

May 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article I will show you how to change the out of the box Laravel configuration to queue new user verification emails.

For queuing password reset emails see:

[How To Queue Laravel Password Reset Email](https://aregsar.com/blog/2020/how-to-queue-laravel-password-reset-email)

## Enabling Email verification

In this section will quickly repeat the steps required to setup the new user verification email notification.

I will first describe how to make the user registration feature flow send a non queued user verification email.

In order to do that we need to first make the `App\User` class implement the `Illuminate\Contracts\Auth\MustVerifyEmail` contract.

```php
class User extends Authenticatable implements MustVerifyEmail
```

Note that the `use Illuminate\Contracts\Auth\MustVerifyEmail;` statement is already included in the `User.php` file. So all we had to do is add the `implements MustVerifyEmail` to the class declaration.

The last thing we need to do is to enable the sending of the verification email by the framework.

To do that we need to make the `Auth::routes` method take a `['verify' => true]` input argument.

```php
Auth::routes(['verify' => true]);
```

By default Laravel triggers a notification when a user completes the registration step to send a verification email when the `Auth::routes` method has this feature enabled via the  `['verify' => true]` parameter.

## Queuing the verification email

The email verification email is not queued by default and is sent as part of the registration request flow when enabled.

The framework triggers a notification to send the email verification email after the user is added to the database.

 The notification sends the email by calling the `sendEmailVerificationNotification()` method on the `App\User` class. The `sendEmailVerificationNotification` method is included in the `App\User` class through the `Illuminate\Foundation\Auth\User` class (aliased as `Authenticatable`) that is extended by `App\User`:

```php
namespace App;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

//This App\User class extends the Illuminate\Foundation\Auth\User class that is aliased to Authenticatable above
class User extends Authenticatable implements MustVerifyEmail
{
 use Notifiable;
}
```

The `Illuminate\Foundation\Auth\User` class implements the `Illuminate\Auth\MustVerifyEmail` trait (Not to be confused with the `Illuminate\Contracts\Auth\MustVerifyEmail` contract that `App\User` needs to implement) that in turn contains the default implementation of the `sendEmailVerificationNotification` method::

```php
namespace Illuminate\Foundation\Auth;
class User extends Model implements
    AuthenticatableContract,
    AuthorizableContract,
    CanResetPasswordContract
{
    use Authenticatable, Authorizable, CanResetPassword, MustVerifyEmail;
}
```

Here is the default `sendEmailVerificationNotification` implementation in the `Illuminate\Auth\MustVerifyEmail` trait:

```php
namespace Illuminate\Auth;

use Illuminate\Auth\Notifications\VerifyEmail;

trait MustVerifyEmail
{
    public function sendEmailVerificationNotification()
    {
        $this->notify(new VerifyEmail);
    }
}
```

Because `sendEmailVerificationNotification` method is included as a trait in the extended `Illuminate\Foundation\Auth\User` class, we can override its default implementation by simply implementing a `sendEmailVerificationNotification` method in the `App\User` class with our overridden implementation that will queue the delivery of user verification notification email.

We can take one of two approaches for the implementation within the overridden `sendEmailVerificationNotification` method:

Approach 1 - We can queue the notification

Approach 2 - We can dispatch a queued job that sends the notification

These approaches are shown in the next two sections.

## Approach 1 - Queuing the notification approach

With this approach we will extend the `Illuminate\Auth\Notifications\VerifyEmail` notification, that the `Illuminate\Auth\MustVerifyEmail` trait sends, into a queue-able notification and queue this extended notification in the overridden `sendEmailVerificationNotification` method that we will add to the `App\User` class.

So first we create, in the `\App\Notifications\Auth` directory, a new `QueuedVerifyEmail` notification that extends the `Illuminate\Auth\Notifications\VerifyEmail` notification:

```bash
artisan make:notification Auth/QueuedVerifyEmail
```

This command creates the class `\App\Notifications\Auth\QueuedVerifyEmail`

Next we make this class extend the `Illuminate\Auth\Notifications\VerifyEmail` class
and implement the `Illuminate\Contracts\Queue\ShouldQueue` contract

Finally we add the `Illuminate\Bus\Queueable` trait to the body of the class.

That is all we need to do to make a new `QueuedVerifyEmail` notification that is queue-able.

Below is the our final `QueuedVerifyEmail` notification class:

```php
namespace App\Notifications\Auth;

use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Bus\Queueable;

class QueuedVerifyEmail extends VerifyEmail implements ShouldQueue
{
    use Queueable;
}
```

Now we need to add the `sendEmailVerificationNotification()` method to the `App\User` class which will override the default method that it gets from the `MustVerifyEmail` trait of its parent class. Then we can notify the user using the `QueuedVerifyEmail` notification to substitute our new queued notification instead of the original notification.

```php
class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    //Overrideen sendEmailVerificationNotification implementation
    public function sendEmailVerificationNotification()
    {
        $this->notify(new \App\Notifications\Auth\QueuedVerifyEmail);
    }
}
```

Optionally we can add a constructor to the `QueuedVerifyEmail` class to queue the notifications to a different queue and/or different connection from the default connection and queue.

```php
class QueuedVerifyEmail extends VerifyEmail implements ShouldQueue
{
    use Queueable;

    public function __construct()
    {
        //uncomment to override the queue
        //$this->queue = 'verify';

        //uncomment to override the connection
        //$this->connection = 'verify';
    }
}
```

## Approach 2 - Queueing a job that sends the notification approach

In this approach we create a queued job that we dispatch in the overridden `sendEmailVerificationNotification` method of the `App\User` class.

When the job is processed, it will send the original `Illuminate\Auth\Notifications\VerifyEmail` notification that was being sent directly from the overridden `sendEmailVerificationNotification` method.

So we first need to create a job that will be queued.

```bash
artisan make:job QueuedVerifyEmailJob
```

Next in the `handle()` method of the job we need to copy a slightly modified implementation from the original `sendEmailVerificationNotification()` method as shown below:

```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Auth\Notifications\VerifyEmail;
use App\User;

class QueuedVerifyEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $user;

    public function __construct(User $user)
    {
        //the user property passed to the constructor through the job dispatch method
        $this->user = $user;
    }

    public function handle()
    {
        //This queued job sends
        //Illuminate\Auth\Notifications\VerifyEmail verification
        //to the user by triggering the notification
        $this->user->notify(new VerifyEmail);
    }
}
```

Finally we need to override the default `sendEmailVerificationNotification()` method implementation by adding a `sendEmailVerificationNotification()` method directly to the `App\User` class where we will dispatch the `QueuedVerifyEmailJob` to the queue.

```php
class User
{
    public function sendEmailVerificationNotification()
    {
        //dispactches the job to the queue passing it this User object
         QueuedVerifyEmailJob::dispatch($this);
    }
}
```

Looking back at the `QueuedVerifyEmailJob` class we can see that the implementation we added to its  `handle()` method is slightly different than the implementation of the original `sendEmailVerificationNotification` method.

The `handle()` method of the job class calls `$this->user->notify` where the original `sendEmailVerificationNotification` method in the `MustVerifyEmail` trait calls `$this->notify`.

This is because in the original implementation, the `$this` pointer refers to the `App\User` class that includes the trait. But in the Job class the `$this` pointer refers to the job class so we need to reference the `user` property of the job class then call the `notify` method on the user.

## Protecting routes that require verified users

In order to protect routes from allowing access to users that have not verified their email we can use the `verified` middleware.

The example below shows that the middleware is added to the constructor of a controller. We could also apply the middleware to the routes directly instead of to the controller.

```php
public function __construct()
{
    $this->middleware('auth','verified');
}
```

It should be noted that the `verified` middleware checks and authenticated user if they have verified their email by checking the database record for the `App\User`. So if users that are not authenticated automatically are not verified.
