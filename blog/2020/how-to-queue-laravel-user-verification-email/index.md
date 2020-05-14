# How To Queue Laravel User Verification Email

May 14, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[How To Queue Laravel User Verification Email](https://aregsar.com/blog/2020/how-to-queue-laravel-user-verification-email)

## Sending User Verification Email

I have instructions here that sets up a mail server in a docker container and tests it by configuring the Laravel user verification email here:

[Laravel App With Mail Server In Docker](https://aregsar.com/blog/2020/laravel-app-with-mail-server-in-docker)

In this article I will show you how to change that configuration to queue the user verification email.

## Enabling Email verification

I will first quickly repeat the steps required to setup a verification email.

make the User class implement `MustVerifyEmail`

`class User extends Authenticatable implements MustVerifyEmail`

make the Auth::routes method take a `['verify' => true]` input argument
`Auth::routes(['verify' => true]);`

protect any routes that use the `auth` middleware that we want to check the authenticated user has verified their email with the `verified` middleware by adding it to the controller constructor or to the route. 

Below is an example of adding it to a controller constructor. 

```php
public function __construct()
{
    $this->middleware('auth','verified');
}
```

## Queuing the verification email

By default Laravel triggers a notification when a user completed the registration step to send a verification email when the `Auth::routes` method has this feature enabled via the  `['verify' => true]` parameter.

This notification directly sends the email as part of the web request.

The default implementation of the of the sendEmailVerificationNotification() method is the code sends the verify email notification. This method is included in the User class through the xxx trait:

```php
public function sendEmailVerificationNotification()
{
    $this->notify(new Notifications\VerifyEmail);
}
```

In order to queue the delivery of this email we can take one of two approaches:

1 We can queue the notification
2 We can dispatch a queued job that sends the notification

These approaches are shown in the next two sections.

## Queuing the notification approach

Create a new notification that extends the verify email notification:

```bash
artisan make:notification Auth/QueuedVerifyEmail
```

Make this class extend ` Illuminate\Auth\Notifications\VerifyEmail`
and implement `Illuminate\Contracts\Queue\ShouldQueue`

Also add the `Illuminate\Bus\Queueable` trait to the body of the class.

That is all we need to do to make a new notification based on the frameworks notification that is queue-able.

```php
namespace App\Auth\Notifications;

use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Bus\Queueable;

class QueuedVerifyEmail extends VerifyEmail implements ShouldQueue
{
    use Queueable;
}
```

The last thing we need to do is to add the `sendEmailVerificationNotification()` method to the User class which will override the default method that the `User` class gets from the trait to substitute our queued notification instead of the original notification.

```php
class User
{
    public function sendEmailVerificationNotification()
    {
        $this->notify(new \App\Notifications\Auth\QueuedVerifyEmail);
    }
}
```

Optionally add a constructor to the QueuedVerifyEmail class to queue verify the notifications to a different queue and/or different connection from the default connection and queue

```php
class QueuedVerifyEmail extends VerifyEmail implements ShouldQueue
{
    use Queueable;

    public function __construct()
    {
        $this->queue = 'verify';
        //$this->connection = 'verify';
    }
}
```

## Queueing a job that sends the notification approach

First we need to create a job that will be queued.
This job will be responsible for sending the notification via the User object that is passed to it when the job is processed.

```bash
artisan make:job QueuedVerifyEmailJob
```

Next in the handle() method we need to copy the notification implementation that is in the original `sendEmailVerificationNotification()` method from the xxx trait included in the User class.

```php
namespace App\Jobs;

use App\User;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Auth\Notifications\VerifyEmail;

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
        // This queued job sends verification to the user by triggering the notification
        $this->user->notify(new Notifications\VerifyEmail);
    }
}
```

The original implementation was slightly different as shown below:

```php
public function sendEmailVerificationNotification()
{
    //Notifications\VerifyEmail relative to App\Auth
    $this->notify(new Notifications\VerifyEmail);
}
```

This is because the orginal implemenation, the $this pointer was the User class in which we added the trait that has the sendEmailVerificationNotification implemenation.
That is why in the `handle` method of the job class we call `$this->user->notify` instead of `$this->notify`.


Next we need to override the default `sendEmailVerificationNotification()` method implementation by adding a sendEmailVerificationNotification() method directly in User class.
In this method we dispatch the QueuedVerifyEmailJob to the queue.

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
