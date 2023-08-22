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

![image](https://github.com/nirajp82/NGINX/assets/61636643/5d83f7f5-55dd-4820-8a3c-f96d002953be)


References:
* https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts#understanding-nginx-configuration-contexts
