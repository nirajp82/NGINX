Using the NGINX Lua module allows you to extend NGINX's functionality by embedding Lua scripts directly into your NGINX configuration. This can be useful for tasks like authentication, URL rewriting, or dynamic content generation. Here's a step-by-step guide on how to use the NGINX Lua module:

**Step 1: Install NGINX with Lua Module**

Alternatively, ngx_lua can be manually compiled into Nginx:

LuaJIT can be downloaded from the latest release of OpenResty's LuaJIT fork. The official LuaJIT 2.x releases are also supported, although performance will be significantly lower for reasons elaborated above
Download the latest version of the ngx_devel_kit (NDK) module HERE
Download the latest version of ngx_lua HERE
Download the latest supported version of Nginx HERE (See Nginx Compatibility)
Download the latest version of the lua-resty-core HERE
Download the latest version of the lua-resty-lrucache HERE
Build the source with this module:

```sh
 
 wget 'https://openresty.org/download/nginx-1.19.3.tar.gz'
 tar -xzvf nginx-1.19.3.tar.gz
 cd nginx-1.19.3/

 # tell nginx's build system where to find LuaJIT 2.0:
 export LUAJIT_LIB=/path/to/luajit/lib
 export LUAJIT_INC=/path/to/luajit/include/luajit-2.0

 # tell nginx's build system where to find LuaJIT 2.1:
 export LUAJIT_LIB=/path/to/luajit/lib
 export LUAJIT_INC=/path/to/luajit/include/luajit-2.1

 # Here we assume Nginx is to be installed under /opt/nginx/.
 ./configure --prefix=/opt/nginx \
         --with-ld-opt="-Wl,-rpath,/path/to/luajit/lib" \
         --add-module=/path/to/ngx_devel_kit \
         --add-module=/path/to/lua-nginx-module

 # Note that you may also want to add `./configure` options which are used in your
 # current nginx build.
 # You can get usually those options using command nginx -V

 # you can change the parallelism number 2 below to fit the number of spare CPU cores in your
 # machine.
 make -j2
 make install

 # Note that this version of lug-nginx-module not allow to set `lua_load_resty_core off;` any more.
 # So, you have to install `lua-resty-core` and `lua-resty-lrucache` manually as below.

 cd lua-resty-core
 make install PREFIX=/opt/nginx
 cd lua-resty-lrucache
 make install PREFIX=/opt/nginx

 # add necessary `lua_package_path` directive to `nginx.conf`, in the http context

 lua_package_path "/opt/nginx/lib/lua/?.lua;;";
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

Reference: https://github.com/openresty/lua-nginx-module

This is a basic example of how to use the NGINX Lua module. You can use Lua to perform more complex tasks like authentication, request/response modification, or even proxying requests to other services based on custom logic. Be sure to consult the NGINX Lua module documentation for more advanced use cases and available APIs: https://github.com/openresty/lua-nginx-module
