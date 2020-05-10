# Laravel App With Mail Server In Docker

May 11, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Laravel App With Mail Server In Docker](https://aregsar.com/blog/2020/laravel-app-with-mail-server-in-docker)

## Creating files and directories

> Note if you have already a docker-compose.yml then skip to the next section.

```bash
echo '/data' >> .gitignore
mkdir data
touch docker-compose.yml
echo `version: "3.1"' >> docker-compose.yml
echo `services:' >> docker-compose.yml
```

## Adding the mysql service to docker-compose

We can pick one of the following configurations to add.

Mail Hog

```yaml
    mailhog:
      image: mailhog/mailhog:latest
      container_name: myapp-mailhog
      ports:
        - "8003:1025"
        - "8100:8025"
```

Mail Catcher

```yaml
    mailcatcher:
        image: schickling/mailcatcher:latest
        container_name: myapp-mailcatcher
        ports:
            - 8003:1025
            - 8100:1080
```

Or Paper Cut:

```yaml
    papercut:
      image: jijiechen/papercut:latest
      container_name: myapp-papercut
      ports:
        - "8003:25"
        - "8100:37408"
```

You will note that there are two port mappings. The mapping that is exposed on port 8003 on the local host is the mail server IMAP server port that the application dispatches emails to.
THe other port that is exposed on the local host port 8100 is a  email dashboard http port that we can navigate to, to see and manage the emails that were received by the mailserver.

> Note: as configured, the emails will not be persisted across container restarts.

## Adding the environment variables for the mail service

```ini
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=8003
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"
```

## Sending emails in the background using Queues


```php
php artisan mail:job WelcomeEmail
```

## Processing queued  emails using background workers

```bash
php artisan queue:work --retry=3 --timeout=30
```

## Add email verification step to user registration