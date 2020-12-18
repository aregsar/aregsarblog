# nginx php config settings explained

[nginx php config settings explained](https://aregsar.com/blog/2020/nginx-php-config-settings-explained)

In this post I will attempt to demystify the nginx configuration settings for serving html pages and php applications

## The components of an NGINX config file

The main NGINX config file is located at `/etc/nginx/nginx.config`.

The content is listed below:

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
}
```

At the top level we have general settings for the NGINX processes.

For HTTP requests the the `http{}` block inside which are the HTTP related settings.

Within the HTTP blog there can be one or more `server{}` blocks. Each one of these blocks is responsible for matching a request coming in through an IP and PORT as shown below.

```nginx
http {
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

The HTTP block also includes an `include` directive that includes one or more configuration files that include the `.conf` extension from the `/etc/nginx/conf.d/` directory. These files can also include server blocks that will be included in the HTTP block.

General best practice is to not modify the main config file to add server blocks. If we are only running a single server then it is best practice to add the server block in a `/etc/nginx/conf.d/server.conf` file.

If we are running more than one website on a single server, there is a `/etc/nginx/sites_active/` directory that we can add configuration files to for each server. Each of these files will contain a single server block for a specific website. Any files in this directory will be included by default in the HTTP block.

Instead of directly adding configuration files in `/etc/nginx/sites_active/` it is best practice to instead add the file to the `/etc/nginx/sites_available/` directory and symlink it in the `/etc/nginx/sites_active/` directory. This way we can keep the configuration file and simply unlink it in `/etc/nginx/sites_active/` if we want to disable the site, instead of having to remove the actual file from `/etc/nginx/sites_active/`.

## Configuring Server blocks

As mentioned before the `server` configuration block that matches the URI of the request we want to respond to.

Below a server block for responding to requests using a PHP server is shown:

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

In the following sections I will describe each of the main directives and blocks within the server block. There can be additional directives that will be shown in a more fully configured server block.

### The listen directive

The listen directives at the top match the PORT that the request is coming in. `listen 80` is for IPv4 and the `listen [::]:80` is for IPv6 port matching. You could also specify a specific IP along with the PORT such as `listen 123.22.22.33:80` to match the right interface if there are multiple IP interfaces on the server.

Once listen directive has a match, that server block is chosen to handle the request.

Sometimes multiple server blocks may use the same listen directive which will result in multiple matching server blocks. In that case and only in that case the `server_name` directive is used to match the requests for the server blocks that has a match using the `listen` directive.

Note that if the listen directive does not match a server block then the server_name directive is not even considered and the server block will be eliminated from consideration.

### The server_name directive

The server_name directive matches the host header sent as part of the request to select a single server block if the listen directive had multiple matches. The hots header can either contain a domain name or an IP address depending on what address was used to send the HTTP request.

In the server block above the `server_name example.com www.example.com;` directive will match if the host header is either `example.com` of `www.example.com`.

Other examples are `server_name localhost` or `server_name 127.0.0.1`.

Again it is important to emphasize that the server name directive only comes into play if the listen directive did not find a unique match between server blocks.

## The root directive

Within a server block there will be directives such as the `try_files` directive that will try to match the URI path of the request. The URI path is everything after the host and port portion of the URI not including the query string.

The root directive maps the root of the URI path to the directory location specified as its parameter.

So for example, in the server block above the root URI `/` path is mapped to `/var/www/example.com/public/` directory by the `root` directive.

Everything after the root path will be matched to the directory structure in the root directory.

So for example, the URI path `/foo/bar.html` will be matched matched to `/var/www/example.com/public/foo/bar.html` if that path exists.

## The index directive

The `index` directive is useful when matching URI paths that are directories (i.e. they end in trailing slash) using the `try_files{}` directive.

When a URI directory path is matched to an existing file directory path under the root directory then the `index` directive comes into play.

The files listed as the parameters of the `index` directive are appended to the URI path one at a time resulting a URI file path.

This URI file path is then internally redirected and goes through the matching rules of the `location{}` blocks within the server block.

Location blocks are described next.

### The location blocks

Location blocks specify URI paths to match within the URI path of the incoming request. This match happens starting with the most specific location path and ends with the least specific location path. The location block that matches first will specify the further matching directives that apply to the request and how the is routed.

Some directives within the location block will cause the request to be redirected internally and go through the entire location matching process within the server block again. This redirection usually happens when the URI path is modified based on one of the directives that apply to the location block. One example of this is the `index` directive.

> Directives in a server block are inherited by all child blocks within that server block. So the index directive in the server block applies to all location blocks in that server block.

For example if we have two location blocks below:

```nginx
location /{

}

location /api/{

}
```

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

    root /var/www/example.com/public;

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
                try_files $uri $uri/ =404;
        }

        location /api {
                #index index.php;
                #try_files $uri/ /index.php?$query_string;

                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }

        location /api/ {
                #index index.php;
                #try_files $uri/ /index.php?$query_string;

                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }

        <!-- location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        } -->
}
```

Any request that contain a URI prefix of `/api/` are matched to the `location /api/` block and routed to the PHP API server as long as the URIs end with `index.php`.

All other requests are matched to `location /` and served as static files. So a request to `https://example.com/`, `https://example.com/` or `https://example.com/index.html` will serve the static page to bootstrap a Vue JS app perhaps.

## Docker and NGINX configuration

When deploying PHP applications using Docker we typically have separate containers for NGINX and the PHP server PHP-FPM.

In this scenario both containers will have the root directory in the file system.
The difference is that the NGINX container root directory will only have the static html and other static files that it serves directly but not the index.php file.

Note that even though the NGINX configuration uses `index.php` string in the URI path to match which location block that handles the request, it does not need the actual file to exist on the server since it does not serve the index.php file. The location block that matches the `index.php` string in the URI path forwards the request to the PHP server to be served by PHP-FPM.

The PHP-FPM server on the other hand needs the index.php file to exist in the root directory so it can serve it. It needs the index.php file but it does not need all the other static files that NGINX serves, since PHP is not serving those files.

It just so happens that when we are using a single server to run both NGINX and PHP then both the static files and the index.php file need to exist in the root directory so each server can serve the file requested of it.

> Note: The root directory is specified in the NGINX config so it has to be passed to PHP-FPM as a CGI parameter along with the request so PHP-FPM will know the location of the root directory.

Even thought NGINX and PHP-FPM only need the files that they serve to exist in the root directory specified by the NGINX configuration, all the files can be copied to root directory location on both containers as long as space is not a constraint.

The other aspect that is different about hosting NGINX and PHP-FPM in separate docker containers is that the way that the request is passed from NGINX to PHP-FPM in the `location * ./php.index` block will be slightly different.

In a multi container solution using docker compose each service will have it own name. So if the PHP-FPM service is named `app` then the request is passed from the NGINX container to the `app` service as `proxy_pass http://app:80`. The internal networking DNS for docker will figure out how to route the request to the `app` container. Internally in the PHP_FPM container port 80 will be mapped to port 9000 which is the port that PHP-FPM runs on by default.

The updated NGINX server block that includes this change is shown below:

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
                proxy_pass http://app:80;
        }
}
```

## Resources

http://nginx.org/en/docs/varindex.html
