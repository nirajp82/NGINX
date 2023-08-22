The `include` directive in NGINX is used to include external configuration files within the main NGINX configuration file. This feature helps in modularizing and organizing your configuration, making it easier to manage complex setups and reuse configuration snippets across multiple server blocks or contexts.

The syntax of the include directive is straightforward:

`include path/to/your/config/file;`

Here's how you might use the `include` directive in an NGINX configuration:

Suppose you have a main configuration file named `nginx.conf`, and you want to split your configuration into separate files for better organization. Create a directory named `conf.d` (or any other name you prefer)
within the same directory as `nginx.conf`. Inside this directory, you can create separate configuration files for different purposes, like `app1.conf`, `app2.conf`, and so on.

Now, in your `nginx.conf` file, you can use the include directive to include these files:

![image](https://github.com/nirajp82/NGINX/assets/61636643/8d4a7e59-f5ed-427e-8c80-47050553587e)

In this example, the include /path/to/nginx/conf.d/*.conf; line includes all .conf files within the conf.d directory into the http context of the main configuration.

Using the include directive offers several benefits:

* Modularity: It allows you to split your configuration into logical parts, making it easier to manage and maintain.
* Reusability: You can reuse configuration snippets across multiple server blocks or contexts.
* Separation of Concerns: Different configuration files can focus on specific aspects of your setup (e.g., SSL settings, proxy configurations).
* Ease of Updates: You can update specific parts of your configuration without touching the main configuration file.

