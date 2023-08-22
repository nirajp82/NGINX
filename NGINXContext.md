# Nginx Configuration Contexts

Nginx is a high performance web server that is responsible for handling the load of some of the largest sites on the internet. It is especially good at handling many concurrent connections and excels at forwarding or serving static content. 

 The location of Nginx configuration file will depend on how Nginx was installed. On many Linux distributions, the file will be located at `/etc/nginx/nginx.conf`. If it does not exist there, it may also be at `/usr/local/nginx/conf/nginx.conf` or `/usr/local/etc/nginx/nginx.conf`.

 One of the first things that you should notice when looking at the main configuration file is that it is organized in a tree-like structure, marked by sets of brackets ({ and }). In Nginx documentation, the areas that these brackets define are called “contexts” because they contain configuration details that are separated according to their area of concern. These divisions provide an organizational structure along with some conditional logic to decide whether to apply the configurations within.

Because contexts can be layered within one another, Nginx allows configurations to be inherited. As a general rule, if a directive is valid in multiple nested scopes, a declaration in a broader context will be passed on to any child contexts as default values. The child contexts can override these values. It is important to note that an override to any array-type directives will replace the previous value, not add to it.

Directives can only be used in the contexts that they were designed for. Nginx will throw an error when reading a configuration file with directives that are declared in the wrong context. 

## The Core Contexts
In NGINX, the term "context" refers to different configuration blocks or sections within the NGINX configuration file where you define various directives to configure the behavior of the server. These contexts help organize and separate different types of configuration settings, making it easier to manage and customize the server's behavior for different scenarios.


### The Main Context
The most general context is the “main” or “global” context. It is the only context that is not contained within the typical context blocks that look like this:

![image](https://github.com/nirajp82/NGINX/assets/61636643/a50fa75f-2a0b-4a86-ad9a-8fdc0c6697f8)

Any directive that exists entirely outside of these blocks belongs to the “main” context. Keep in mind that if your Nginx configuration is set up in a modular fashion – i.e., with configuration options in multiple files – some files will contain instructions that appear to exist outside of a bracketed context, but will be included within a context when the configuration is loaded together.

The main context represents the broadest environment for Nginx configuration. It is used to configure details that affect the entire application. While the directives in this section affect the lower contexts, many of these cannot be overridden in lower levels.

Some common details that are configured in the main context are the system user and group to run the worker processes as, the number of workers, and the file to save the main Nginx process’s ID. The default error file for the entire application can be set at this level (this can be overridden in more specific contexts).

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



References:
* https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts#understanding-nginx-configuration-contexts
