# How To Queue Laravel Password Reset Email

May 14, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[How To Queue Laravel Password Reset Email](https://aregsar.com/blog/2020/how-to-queue-laravel-password-reset-email)

## Sending Password Reset Email

In this article I will show you how to change that configuration to queue the user verification email.

## Enabling Password Reset

I will first quickly repeat the steps required to setup a verification email.

make the User class implement `Illuminate\Contracts\Auth\CanResetPassword`
The `use Illuminate\Contracts\Auth\CanResetPassword;` statement is already included in the User.php file.
So all we have to do is add the `implements CanResetPassword` to the class declaration.

```php
class User extends Authenticatable implements CanResetPassword
```

## Queuing the Password Reset email

Normally the included password reset notification directly sends the email as part of the web request by calling the sendPasswordResetNotification() method on the User class. The sendPasswordResetNotification method is included in the User class through the extended `Illuminate\Foundation\Auth\User` class:

```php
namespace App;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

//we are extending the Illuminate\Foundation\Auth\User class that is aliased to Authenticatable above
class User extends Authenticatable
{
    use Notifiable;
}
```

The `Illuminate\Foundation\Auth\User` class implements the `CanResetPassword` trait that in turn contains the default implementation of the sendPasswordResetNotification method

```php
class User extends Model implements
    AuthenticatableContract,
    AuthorizableContract,
    CanResetPasswordContract
{
    use Authenticatable, Authorizable, CanResetPassword, MustVerifyEmail;
}
```

Here is the default  sendPasswordResetNotification implementation in the MustVerifyEmail trait:

```php
namespace Illuminate\Auth\Passwords;

use Illuminate\Auth\Notifications\ResetPassword as ResetPasswordNotification;

trait CanResetPassword
{
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }
}
```

Because sendPasswordResetNotification method is included as a trait in the extended User class, we can override its default implementation by simply adding a sendPasswordResetNotification method in the User class with our overridden implementation that will queue the delivery of this email.

Inside the overridden sendPasswordResetNotification method we can take one of two approaches for the implementation within the sendPasswordResetNotification method:

Approach 1-We can queue the notification
Approach 2-We can dispatch a queued job that sends the notification

These approaches are shown in the next two sections.

## Approach 1 - Queuing the notification approach

With this approach we will extend the Illuminate\Auth\Notifications\VerifyEmail notification that the MustVerifyEmail trait sends into a queue-able notification and queue this extended notification in the overridden sendPasswordResetNotification of the User class.

So first we create a new notification that extends the verify email notification:

```bash
artisan make:notification Auth/QueuedResetPassword
```

This command creates the class `\App\Notifications\Auth\QueuedResetPassword`

Make this class extend ` Illuminate\Auth\Notifications\ResetPassword`
and implement `Illuminate\Contracts\Queue\ShouldQueue`

Also add the `Illuminate\Bus\Queueable` trait to the body of the class.

That is all we need to do to make a new notification based on the frameworks notification that is queue-able.

```php
namespace App\Auth\Notifications;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Bus\Queueable;
use Illuminate\Auth\Notifications\ResetPassword;

class QueuedResetPassword extends ResetPassword implements ShouldQueue
{
    use Queueable;
}
```

The last thing we need to do is to add the `sendPasswordResetNotification()` method to the User class which will override the default method that the `User` class gets from the trait to substitute our queued notification instead of the original notification.

```php
class User extends Authenticatable
{
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new \App\Notifications\Auth\QueuedResetPassword($token));
    }
}
```

Optionally add a constructor to the QueuedResetPassword class to queue verify the notifications to a different queue and/or different connection from the default connection and queue

```php
class QueuedResetPassword extends ResetPassword implements ShouldQueue
{
    use Queueable;

    public function __construct()
    {
        $this->queue = 'reset';
        //$this->connection = 'reset';
    }
}
```

## Approach 2 - Queueing a job that sends the notification approach

In this approach we create a queued job that we dispatch in the in the overridden sendPasswordResetNotification method of the User class. When the job is processed, it will send the original Illuminate\Auth\Notifications\ResetPassword notification that was being sent directly from the sendPasswordResetNotification method before.

So we first need to create a job that will be queued.

```bash
artisan make:job QueuedPasswordResetJob
```

Next in the handle() method we need to copy the notification implementation that is in the original `sendPasswordResetNotification()` method.

```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Auth\Notifications\ResetPassword;
use App\User;

class QueuedPasswordResetJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $user;
    protected $token;

    public function __construct(User $user, $token)
    {
        //the user property passed to the constructor through the job dispatch method
        $this->user = $user;
        $this->token = $token;
    }

    public function handle()
    {
        //This queued job sends
        //Illuminate\Auth\Notifications\ResetPassword notification
        //to the user by triggering the notification
        $this->user->notify(new ResetPassword($this->token));
    }
}
```

Finally we need to override the default `sendPasswordResetNotification()` method implementation by adding a sendPasswordResetNotification() method directly in User class.
In this method we dispatch the QueuedVerifyEmailJob to the queue.

```php
class User extends Authenticatable
{
    public function sendPasswordResetNotification($token)
    {
        //dispactches the job to the queue passing it this User object
         QueuedPasswordResetJob::dispatch($this,$token);
    }
}
```

Note that the original implementation of sendPasswordResetNotification was slightly different then the implementation in the handle method of the  QueuedVerifyEmailJob class.

In the `handle` method of the job class we call `$this->user->notify` where as the original sendPasswordResetNotification method in the `CanResetPassword` trait calls `$this->notify`.
This is because in the `CanResetPassword` traits implementation of the sendPasswordResetNotification method the $this pointer references the User class that includes the trait.
So in the job class we we reference the User and call notify on it.

==================================

After a password is reset, the user will automatically be logged into the application and redirected to /home. You can customize the post password reset redirect location by defining a redirectTo property on the ResetPasswordController:

protected $redirectTo = '/dashboard';
