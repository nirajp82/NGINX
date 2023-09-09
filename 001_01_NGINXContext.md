# Nginx Configuration Contexts

Nginx is a high performance web server that is responsible for handling the load of some of the largest sites on the internet. It is especially good at handling many concurrent connections and excels at forwarding or serving static content. 

 The location of Nginx configuration file will depend on how Nginx was installed. On many Linux distributions, the file will be located at `/etc/nginx/nginx.conf`. If it does not exist there, it may also be at `/usr/local/nginx/conf/nginx.conf` or `/usr/local/etc/nginx/nginx.conf`.

 One of the first things that you should notice when looking at the main configuration file is that it is organized in a tree-like structure, marked by sets of brackets ({ and }). In Nginx documentation, the areas that these brackets define are called “contexts” because they contain configuration details that are separated according to their area of concern. These divisions provide an organizational structure along with some conditional logic to decide whether to apply the configurations within.

Because contexts can be layered within one another, Nginx allows configurations to be inherited. As a general rule, if a directive is valid in multiple nested scopes, a declaration in a broader context will be passed on to any child contexts as default values. The child contexts can override these values. It is important to note that an override to any array-type directives will replace the previous value, not add to it.

Directives can only be used in the contexts that they were designed for. Nginx will throw an error when reading a configuration file with directives that are declared in the wrong context. 

# The Core Contexts
In NGINX, the term "context" refers to different configuration blocks or sections within the NGINX configuration file where you define various directives to configure the behavior of the server. These contexts help organize and separate different types of configuration settings, making it easier to manage and customize the server's behavior for different scenarios.


## The Main Context
The most general context is the “main” or “global” context. It is the only context that is not contained within the typical context blocks that look like this:

![image](https://github.com/nirajp82/NGINX/assets/61636643/a50fa75f-2a0b-4a86-ad9a-8fdc0c6697f8)

Any directive that exists entirely outside of these blocks belongs to the “main” context. Keep in mind that if your Nginx configuration is set up in a modular fashion – i.e., with configuration options in multiple files – some files will contain instructions that appear to exist outside of a bracketed context, but will be included within a context when the configuration is loaded together.

The main context represents the broadest environment for Nginx configuration. It is used to configure details that affect the entire application. While the directives in this section affect the lower contexts, many of these cannot be overridden in lower levels.

Some common details that are configured in the main context are the system user and group to run the worker processes as, the number of workers, and the file to save the main Nginx process’s ID. The default error file for the entire application can be set at this level (this can be overridden in more specific contexts).

The main context (outermost block) and the HTTP context (within the http block) are executed at the start of NGINX when it is initially loaded or restarted. These contexts contain global and server-wide settings that apply to the entire server. 

## The Events Context
The “events” context is contained within the “main” context. It is used to set global options that affect how Nginx handles connections at a general level. There can only be a single events context defined within the Nginx configuration.

Nginx uses an event-based connection processing model, so the directives defined within this context determine how worker processes should handle connections. The event-driven model enables NGINX to efficiently manage multiple connections without relying on a separate thread or process for each connection.

This context will look like this in the configuration file, outside of any other bracketed contexts:

![image](https://github.com/nirajp82/NGINX/assets/61636643/41aad2a8-9056-4034-8e65-0fd673e0bddc)

In this example, the worker_connections directive is set to 1024, which determines the maximum number of simultaneous connections that each worker process can handle. This directive impacts how NGINX allocates resources and handles incoming connections.

## The HTTP Context
Defining an HTTP context is probably the most common use of Nginx. When configuring Nginx as a web server or reverse proxy, the “http” context will hold the majority of the configuration. This context will contain all of the directives and other contexts necessary to define how the program will handle HTTP or HTTPS connections.

The http context is a sibling of the events context, so they should be listed side-by-side, rather than nested. They both are children of the main context:

![image](https://github.com/nirajp82/NGINX/assets/61636643/a00fee05-eb87-4921-aaae-60410bdab2a6)

In this example:
* The `http` context encapsulates all the HTTP-related configurations.
* The `include` directive is used to include the `mime.types` file, which defines mappings between file extensions and MIME types.
* The `default_type` directive sets the default MIME type for files that do not match any defined types.
* The `server_tokens` directive disables the inclusion of NGINX version details in response headers.
* Inside the `server` block, you can configure how NGINX handles requests for the domain `example.com`.
  
Within the http context, you can use various directives to configure aspects of your HTTP server, such as:
* Defining SSL certificates and enabling HTTPS.
* Setting up proxying to backend servers.
* Configuring SSL/TLS settings.
* Enabling compression for responses.
* Configuring error pages.
* Setting up virtual server blocks (server contexts) for different domain names.
* Defining request and response headers.

This context includes directives that affect how the server handles HTTP requests, such as defining server blocks, setting up proxying, configuring SSL, enabling compression, and more.

The HTTP context (within the http block) (Also main context - outermost block) are executed at the start of NGINX when it is initially loaded or restarted. These contexts contain global and server-wide settings that apply to the entire server.

## The Server Context
Within the http context, you can have multiple server blocks. Each server block represents a virtual server that handles requests for a specific domain or IP address. Inside the server context, you can define directives that are specific to that virtual server, such as listen ports, server_name, SSL settings, and request handling rules.

The server context is where you configure the behavior of your NGINX server for a specific website or application. It includes directives that define how requests should be processed, what content should be served, and how the server should respond to different scenarios.

![image](https://github.com/nirajp82/NGINX/assets/61636643/c16d99a4-f65f-4dd4-9a30-0923dec8cc63)


In this example:

* There are two `server` contexts, each representing a different website (`example.com` and `another-domain.com`).
* The first `server` context handles requests for `example.co`m and `www.example.com`.
  * The `location /` block specifies that requests should be served from the `/var/www/example` directory and use `index.html` as the default `index` file.
  * The `location /images/` block uses the `alias` directive to serve image files from the `/var/www/example/images/` directory.
* The second server context handles requests for another-domain.com.
  * The `location /` block specifies that requests should be served from the `/var/www/another` directory and use `index.php` or `index.html` as the default index files.
  * The `location ~ \.php$` block defines how to process PHP files using PHP-FPM for dynamic content.


Within the server context, you can use directives to configure various aspects of your virtual server, such as:

* Setting up request handling rules.
* Defining URL paths and how they map to file paths on the server.
* Configuring SSL/TLS settings for HTTPS.
* Enabling or disabling specific features like compression or caching.
* Redirecting or rewriting URLs.
* Configuring proxying to backend servers.

  The server context (within the http block) is executed when the server receives a request that matches the server_name directive in that context. If multiple virtual hosts are defined, NGINX selects the appropriate server block based on the request's hostname.

## Location Context: 
The location context allows you to specify configuration settings for specific URL paths within a server block. You can use location blocks to define how NGINX should handle requests to different URLs, including proxying requests to backend servers, serving static files, and applying access controls.

```nginx
location /static/ {
    alias /var/www/static/;
}
```

The location context (within a server block) is executed for each request that matches the location pattern defined in that context. These context-specific settings apply to specific URL paths or patterns and are executed independently for each matching request.


## If Context: 
The if context allows you to add conditional statements to your configuration. It's important to note that the use of if directives should be minimized because they can be complex and potentially lead to unexpected behavior.

Example:

```nginx
if ($request_uri ~* ^/admin) {
    # Configuration for requests matching the condition
}
```

The server context allows you to customize the behavior of your NGINX server for different domains or IP addresses, making it possible to host multiple websites or applications on the same server with distinct configurations.



```nginx
# Sample NGINX configuration file with comments

# Set the user and group that NGINX should run as (best practice is to create a dedicated nginx user)
user nginx;

# Automatically detect the number of CPU cores for worker processes
worker_processes auto;

# Error log location
error_log /var/log/nginx/error.log;

# Access log location
access_log /var/log/nginx/access.log;

# PID file for NGINX process
pid /var/run/nginx/nginx.pid;

# Include additional configuration files from the /etc/nginx/conf.d directory
include /etc/nginx/conf.d/*.conf;

# Events block defines event-driven settings
events {
    # Number of worker connections per worker process
    worker_connections 1024;

    # Enable multiple connections per worker process for performance
    multi_accept on;
}

# HTTP block contains server-wide HTTP settings
http {
    # Include common MIME types
    include mime.types;

    # Default MIME type
    default_type application/octet-stream;

    # Enable GZIP compression for supported content types
    gzip on;
    gzip_comp_level 6;
    gzip_min_length 1000;
    gzip_types text/plain text/css application/json application/javascript application/xml;

    # Security headers
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-XSS-Protection "1; mode=block";

    # Server block defines server-specific settings
    server {
        # Listen on port 80 for HTTP traffic
        listen 80;

        # Server name (replace with your domain name)
        server_name example.com;

        # Define location blocks for handling requests
        location / {
            # Root directory for serving static content
            root /var/www/html;

            # Default index files
            index index.html index.htm;
        }

        # Redirect HTTP to HTTPS (if using SSL)
        # return 301 https://$host$request_uri;

        # SSL configuration (if using HTTPS)
        # listen 443 ssl;
        # ssl_certificate /etc/nginx/ssl/server.crt;
        # ssl_certificate_key /etc/nginx/ssl/server.key;
    }

    # Additional server blocks for virtual hosts can be added here

}
```



References:
* https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts#understanding-nginx-configuration-contexts
