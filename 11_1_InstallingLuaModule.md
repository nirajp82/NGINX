Using the NGINX Lua module allows you to extend NGINX's functionality by embedding Lua scripts directly into your NGINX configuration. This can be useful for tasks like authentication, URL rewriting, or dynamic content generation. Here's a step-by-step guide on how to use the NGINX Lua module:

**Step 1: Install NGINX with Lua Module**

First, you need to make sure you have NGINX installed with the Lua module. If you're using a package manager like apt (Ubuntu/Debian) or yum (CentOS/RHEL), you can typically install NGINX with the Lua module as follows:

For Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install nginx-extras
```

For CentOS/RHEL:

```bash
sudo yum install nginx
```

Ensure that the Lua module is included in your NGINX installation. You can check this by running `nginx -V`, and it should include `--with-http_lua_module`.

**Step 2: Create a Lua Script**

Write the Lua script that you want to execute within your NGINX configuration. Save this script to a location on your server. For example, create a file named `my_lua_script.lua`:

```lua
-- my_lua_script.lua
ngx.say("Hello from NGINX Lua!")
```

This simple script just prints "Hello from NGINX Lua!".

**Step 3: Modify NGINX Configuration**

Next, modify your NGINX configuration to include the Lua script. Typically, the configuration file is located at `/etc/nginx/nginx.conf`, but you may have separate configuration files for your specific sites in the `/etc/nginx/sites-available/` directory.

Here's an example configuration to include the Lua script:

```nginx
http {
    # ... other configuration ...

    server {
        listen 80;
        server_name yourdomain.com;

        location /lua {
            default_type 'text/plain';
            content_by_lua_file /path/to/your/my_lua_script.lua;
        }

        # ... other location blocks ...

    }
}
```

In this configuration:

- `location /lua` defines a location where the Lua script will be executed.
- `default_type 'text/plain'` sets the default content type for the response.
- `content_by_lua_file` specifies the path to your Lua script.

Replace `yourdomain.com` with your actual domain or server name, and update the path to your Lua script.

**Step 4: Reload NGINX**

After making changes to your NGINX configuration, you'll need to reload NGINX to apply the changes:

```bash
sudo nginx -s reload
```

**Step 5: Test the Lua Script**

You can now test the Lua script by accessing the appropriate URL in your web browser or using a tool like `curl`:

```bash
curl http://yourdomain.com/lua
```

You should see the response from your Lua script, which in this case is "Hello from NGINX Lua!"

This is a basic example of how to use the NGINX Lua module. You can use Lua to perform more complex tasks like authentication, request/response modification, or even proxying requests to other services based on custom logic. Be sure to consult the NGINX Lua module documentation for more advanced use cases and available APIs: https://github.com/openresty/lua-nginx-module
