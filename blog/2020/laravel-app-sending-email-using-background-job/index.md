# Laravel App Sending Email Using Background Job

May 13, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article I will show you how we can create a Laravel background job that sends an email using the Mailable class.

I will also show how to queue that job using a Redis backed queue and to process the queued job using a Laravel queue worker.

Before we can queue jobs we need a mail server to test sending emails to. I have covered configuring a local email server running in a docker container here:

[Laravel App With Mail Server In Docker](https://aregsar.com/blog/2020/laravel-app-with-mail-server-in-docker).

Also we will need to configure our Laravel project to use redis in a docker container. I have covered that here:

[Laravel App With Redis In Docker](https://aregsar.com/blog/2020/laravel-app-with-redis-in-docker)

This article assumes we are using the configurations in the above referenced links to send and view the emails.

## Creating the html text and plain text email blade templates

We will create two email templates to deliver email in both Html and plain text formats.

First we will create the html template by creating the following blade template file:

```bash
touch resources/views/mails/welcomeemail.blade.php
```

This will create the `resources/views/mails/welcomeemail.blade.php` template file.

Next we will put the following html markup in `resources/views/mails/welcomeemail.blade.php` file.

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome</title>
</head>
<body>
Welcome to our app
</body>
</html>
```

Finally we will create the plane text template by creating the following blade template file with raw text inside:

```bash
touch resources/views/mails/welcomeemail-text.blade.php
echo 'Welcome to our app' > resources/views/mails/welcomeemail-text.blade.php
```

This will create the `resources/views/mails/welcomeemail-text.blade.php` plain text template file.

## Creating the Mailable

Now that we have the email templates we will create the Mailable class file for our email:

```ini
php artisan make:mail WelcomeEmail
```

This will create the `App/Mail/welcomeemail.php` plain text template file.

The artisan command will create the `App/Mail/WelcomeEmail.php` file.

We will need to replace the initial content of the `build()` method after we created the file, with the content of the `build()` method show below:

```php
<?php
  
namespace App\Mail;
  
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;
  
class WelcomeEmail extends Mailable
{
    use Queueable, SerializesModels;
  
    public function __construct()
    {
    }
  
    public function build()
    {
        //The mails.welcomeemail is a html blade template
        //the mails.welcomeemail-text is a text blade template as a fallback
        //either one can be removed if we want to only send html or text email
        return $this->from('admin@myapp.com', 'Admin')
                    ->subject('Welcome')
                    ->view('mails.welcomeemail')
                    ->text('mails.welcomeemail-text');
    }
}
```

## Creating the Email Job

Now that we have created the email templates and the Mailable class that builds the email, we will create the job that will send the Mailable.

```ini
php artisan make:job WelcomeEmailJob
```

This will create the `App/Jobs/WelcomeEmailJob.php` job class.

Next we need to replace the implementation in the initial file with the implementation below:

```php
<?php
  
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use App\Mail\WelcomeEmail;
use Mail;

class WelcomeEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
  
    protected $to;
  
    public function __construct($email)
    {
        $this->to = $email;
    }

    /**
     * Execute the job.
     */
    public function handle()
    {
        Mail::to($this->to)->send(new WelcomeEmail);
    }
}
```

## Dispatching the Email Job

Now that we have our job created, we will need to dispatch the job to the Redis queue from within a controller action.

In order to do that we must first add the following route to the `routes/web.php` file:

```php
Route::get('welcome', 'WelcomeEmailController@send');
```

Next we can create a `WelcomeEmailController` controller class:

```ini
php artisan make:controller WelcomeEmail
```

This command will create the `WelcomeEmailController.php` file.

Then we can add the following implementation in the controller file:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Controllers\Controller;

use App\Jobs\WelcomeEmailJob;

class WelcomeEmailController extends Controller
{
    public function send(Request $request)
    {
        WelcomeEmailJob::dispatch('recipient@example.com');

        //Alternative way of dispaching Job
        //dispatch(new WelcomeEmailJob('recipient@example.com'));

        //override the default connection
        //override the default queue of the connection
        //you can override both or one or the other
        //WelcomeEmailJob::dispatch('recipient@example.com')->onConnection('sqs')->onQueue('processing');
    }
}
```

When we navigate to the url `http://localhost/welcome`,
the `WelcomeEmailController:: send` action will send the `WelcomeEmailJob` to our redis queue running in the docker container.

## Processing the queued emails

At this point we have dispatched the job to our redis queue but have not sent the email since the job has not been processed yet.

To remedy that, we can run the following artisan command to launch and run a single worker process to process jobs in the queue:

```ini
php artisan queue:work --tries=3 --timeout=30
```

The worker process will pull the `WelcomeEmailJob` job from the queue and process it by calling its `handle()` method. The `handle` method implementation will then send the `WelcomeEmail` email to the mailhog mail server running in the docker container.

If we navigate to http://localhost:8100 we can see the email that was sent in the mailhog server dashboard.

## Further customizations

For further customizations of the Laravel email features see `https://blog.mailtrap.io/laravel-mail-queue/`.
