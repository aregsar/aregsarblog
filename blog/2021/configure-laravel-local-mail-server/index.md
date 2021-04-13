# Configure Laravel Local Mail Server

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

This post will detail how to setup a mailhog email server docker service for local email delivery testing in your Laravel project.

The mailhog server will also expose an http endpoint that shows a dashboard where you can view and manage emails sent to the mailhog server

## Steps to configure Laravel to use a local mail server

To start, go to the root of your Laravel project then follow steps below:

### Step 1 - Setup a Redis sever docker service

Lets first setup a Redis server using a Docker compose service, for local development that our application can connect to:

Create a docker-compose.yml file containing the redis docker service.

```bash
echo docker-compose.yml << EOL
version: "3.1"
services:
    mail:
      image: mailhog/mailhog:latest
      container_name: mail
      ports:
        - "${MAIL_PORT}:1025"
        - "${MAIL_DASHBOARD}:8025"
EOL
```

If you already have a `docker-compose.yml` file in the project root then just add the mailhog service portion of the above yml code under the services section.

### Step 2 - Set the connection settings for the docker Redis service

Open the project `.env` file and set the connection settings for the docker mailhog service.

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=8003
MAIL_DASHBOARD=8100
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"
```

You will need to add the MAIL_DASHBOARD env var as it does not come with the out of the box installation.

The MAIL_FROM_NAME uses the APP_NAME env var configured in the same .env file to the name of your project.

The docker-compose.yml file we setup before will use some of the settings from the .env file.

### Step 3 - Configure the email

The email configuration in the config/mail.php file should look like below:

```php
'mailers' => [
      'smtp' => [
          'transport' => 'smtp',
          'host' => env('MAIL_HOST', 'localhost'),
          //using the same value here for the port number fallback value as in the .env file
          'port' => env('MAIL_PORT', 8003),
          'encryption' => env('MAIL_ENCRYPTION', 'tls'),
          'username' => env('MAIL_USERNAME'),
          'password' => env('MAIL_PASSWORD'),
          'timeout' => null,
      ],
],
  'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'admin@myapp.com'),
        'name' => env('MAIL_FROM_NAME', 'Admin'),
    ],
```

### Step 4 - Using the mail service

First startup the service:

```bash
docker-compose up -d
```

Now you are ready to test sending emails from your app and view the received emails in the dashboard.

Use the Laravel email API to send an email then view the email in the dashboard.

The Mailhog service exposes a dashboard for viewing and managing delivered emails at the MAIL_DASHBOARD port value that we set to 8100.

Open the dashboard in the browser:

```bash
open -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome http://localhost:8025
```

When done you can shutdown the docker service:

```bash
docker-compose down
```

## Using Mailcatcher instead of Mailhog

If you prefer to use mailcatcher instead of mailhog you can change the mail docker service to below:

```yaml
mail:
  image: schickling/mailcatcher:latest
  container_name: mail
  ports:
    - "${MAIL_PORT}:1025"
    - "${MAIL_DASHBOARD}:1080"
```

As you can see only the image name and the dashboard port number are different.
