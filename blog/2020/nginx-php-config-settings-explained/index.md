# nginx php config settings explained

[nginx php config settings explained](https://aregsar.com/blog/2020/nginx-php-config-settings-explained)

In this post I will attempt to demystify the nginx configuration settings for serving html pages and php applications

## The components of an NGINX config file

```nginx
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

http {
    include       /etc/nginx/mime.types;

    default_type  application/octet-stream;

    error_log  /var/log/nginx/error.log;

    access_log /var/log/nginx/access.log;

    keepalive_timeout 65;

    gzip on;

    include /etc/nginx/conf.d/*.conf;

    server{
        listen 80;
        listen [::]:80;
        server_name example.com www.example.com;
        ...
    }

    server{
        ...
    }
}
```

For the purposes of this post I will start with the `server` configuration block that matches the URI of the request we want to respond to.

```nginx
server {
        listen 80;
        listen [::]:80;
        server_name example.com www.example.com;

        root /var/www/example.com/public;

        index index.html index.php;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        }

        error_page 404 /index.php;
}
```

The way the location block work is they match the URI path after the domain name or IP:PORT part of the URI.
They also match starting with more specific to less specific paths.

For example if we have two location block

```nginx
location /{

}

location /api/{

}
```

http://nginx.org/en/docs/varindex.html

Here values of both location blocks end in a trailing slash. This is important because it means that the location matches a directory starting with the directory location specified in the `root` setting.

So if we have the request `https://example.com/api/path` and `https://example.com/another/path`, the first URI will be matched by the `location /api/` because of the more specific `/api/` prefix match and the second one will match with the `location /` path with the `/` prefix match. Note the same matching rules will apply for `https://example.com/api/` and `https://example.com/another/path/` where they have a trailing slash. But if we have `https://example.com/api` without the trailing slash, then it will be matched by the `location /` instead because there is no prefix `/api/` to match the other location block.

There are different types of location matching. The examples above used `prefix` matching to match part or all the route path starting from the root URI `/` slash character.

Another type of match is a regular expression match an has the form of:

```nginx
location ~ \.php$ {

}
```

The `~` tilde character option after the `location` keyword signifies a case sensitive regular expression match type. By default when there is no option specified, the matching defaults to prefix matching.

So in this case we are matching a regular expression that looks for any URI path that ends with `.php`. Examples are `https://example.com/index.php`, `https://example.com/foo.php`, `https://example.com/path/to/index.php` and `https://example.com/path/to/foo.php`. These all will be matched by the above location block.

Once we match a location, we are now ready to apply the rules in the location block. Note that with some of the rules, if the URI that is being matched is modified by these rules, NGINX does a internal redirect and re-matches the modified URI against all the sibling location blocks again. This happens quite frequently when the URI being matched is a directory and gets one of the files specified in the `index` setting appended to the URI being matched, triggering a internal redirect and a re-matching.

So for the `location /` the way the `try_files` rule works is that it first tries to match the URI path `$uri` after location prefix, trying to find a matching file on the same path from the root. If it find the file it will serve the file.

If it does not find a it tries to match a directory `$uri/` after the location prefix, trying to find a matching directory on the same path from the root. If it find the directory it will first try appending the first file parameter `index.html` of the `index` rule and do an internal redirect to try to match the new URI with the appended `index.html`. In this case the `location /` will again match the URL and this time the `try_files` `$uri` will be used to match the `index.html` file. If the file exists in the directory then it will get served. If it doesn't then the `$uri/` will be tries again, this time appending `index.php` and will again redirect internally to find a match for the new URI.

This time however the `location ~` will be matched and will server the `index.php` file from the directory specified with the `root setting`.

> Note that the `index` setting is applied at the level above all the `location` blocks. In that case it will be inherited by all the location blocks. However our `location ~` block will never need to use the `index` setting so we could actually move the `index` setting into the `location /` block that will be using it without impacting the URI matching process.

When `try_files $uri $uri/ =404;` tries to match the remainder of the URI, if does not find a match it will fallback to 404 error status. In our case we will never hit the 404 fallback because there will always be redirected to the `location ~` block based on the `index.php` default page specified in the index setting.

> The `$uri` parameter used in the try_files rule is the current URI that NGINX is working with. The current URI starts with the original request URI but is updated when an `index` setting parameter is appended to it in the case of a directory path URI with a trailing slash. Also query string is removed along with the `?` character, any consecutive `/` characters are replace by a single `/` forward slash and URL encoded characters are decoded.

The `error_page 404 /index.php;` indicates that if a 404 error status is generated by NGINX due to not finding a match in any of the location blocks , then it will try to find a location match for the `/index.php` page, which it will in the `location ~` block and so it will proxy the request to the PHP server. In our case again as stated above this will never happen.

## Alternative configuration with fallback location

A slight variation of this configuration can be found in the wild. I have shown this configuration below where I removed the index.php from the index setting in the `server` block and instead I added a fallback `/index.php?$query_string` parameter to the `location /` block. In this version because we don't have `index.php` default to match in the try_files block and redirect to `location ~ \.php$` location block, we will end up falling back to the `/index.php` in the try_files block which will redirect to the `location ~ \.php$` location block.

Of course it does not hurt to include both the index.php and fallback /index.php in that case the fallback will never be used.

Also note that once we have a location match for the index.php file in the `location ~ \.php$` will look for the `index.php` file in the document root which is specified by the root setting. Even if the path index.php that matched had preceding URL path segments leading to the index.php file at the tail. For instance if the path matched was `/a/b/index.php` the file that will be searched for by the PHP interpreter will be at `/var/www/example.com/public/index.php` and not at `/var/www/example.com/public/a/b/index.php`.

```nginx
server {
        listen 80;
        listen [::]:80;
        server_name example.com www.example.com;

        root /var/www/example.com/public;

        index index.html;

        location / {
                try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }
}
```

## Full configuration with additional settings

```nginx
server {
    listen 80;
    server_name server_domain_or_IP;

    charset utf-8;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    root /var/www/travel_list/public;

    index index.html index.php;

    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    location = /robots.txt  {
        access_log off;
        log_not_found off;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }

    error_page 404 /index.php;
}
```

## Serving static content with an API backend

The following configuration is modified for apps that serve an API within the same domain at a `/api/` URI prefix.

```nginx
server {
        listen 80;
        listen [::]:80;
        server_name example.com www.example.com;

        root /var/www/example.com/public;

        location / {
                index index.html;
                try_files $uri $uri/ /index.php?$query_string;
        }

        location /api/ {
                try_files /index.php?$query_string;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }
}
```

Any request that contain a URI prefix of `/api/` are matched to the `location /api/` block and routed to the PHP API server as long as the URIs end with `index.php`.

All other requests are matched to `location /` and served as static files. So a request to `https://example.com/`, `https://example.com/` or `https://example.com/index.html` will serve the static page to bootstrap a Vue JS app perhaps.
