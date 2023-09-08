Building dynamic modules in Nginx involves several steps. I'll provide you with a general outline of the process and include an example of building a simple Nginx dynamic module.

1. Fetch the NGINX Source (Same as NGINX Production Version)
   
2. Fetch the module source
   - `yum -y install git`
   - Go to nginx dir. `cd nginx-1.24.0/`
   - `git clone https://github.com/perusio/nginx-hello-world-module.git`
  
3. Build Dynamic Module
   - `./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --user=nginx --group=nginx --with-http_mp4_module --add-dynamic-module=nginx-hello-world-module`
     make sure configure command finish successfully without any issue, else you will receive an error when running `make modules` command such as  `make: *** No rule to make target 'modules'.  Stop.`
     
   - `make modules`
  
   - View installed module in objs folder `cd objs/`
  
   - copy the installed modules to nginx dir.
     ```sh
        mkdir /etc/nginx/modules/
        cp ~/nginx-1.24.0/objs/ngx_http_hello_world_module.so /etc/nginx/modules/
     
     ```
   - 
5. Reference module path within NGINX configuration. 

**Step 1: Prerequisites**

Before you begin, make sure you have the following prerequisites:

- Nginx installed on your system.
- Development tools (compiler, make, etc.) installed.
- Nginx source code (matching the version you have installed) downloaded.

**Step 2: Prepare the Nginx Source Code**

1. Download the Nginx source code if you don't have it already:

   ```bash
   wget http://nginx.org/download/nginx-<version>.tar.gz
   tar -xzvf nginx-<version>.tar.gz
   cd nginx-<version>
   ```

2. Create a directory for your module inside the Nginx source directory. For example:

   ```bash
   mkdir -p nginx-<version>/modules/my_module
   ```

3. Create your module's source code files within this directory.

**Step 3: Create a Configuration File**

Create a configuration file for your module named `config`. This file specifies the module's build parameters. Here's a simple example for the module we created:

```nginx
# my_module/config

ngx_addon_name=ngx_http_my_module
HTTP_AUX_FILTER_MODULES="$HTTP_AUX_FILTER_MODULES ngx_http_my_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_my_module.c"
```

In this example, replace `ngx_http_my_module` with your module's name and provide the appropriate source file(s) under `NGX_ADDON_SRCS`.

**Step 4: Configure Nginx with the Module**

Run the `configure` script with the `--add-dynamic-module` option to include your module:

```bash
./configure --add-dynamic-module=modules/my_module
```

This will configure Nginx to build your module as a dynamic module during the compilation process.

**Step 5: Compile Nginx with the Module**

Compile Nginx with your dynamic module:

```bash
make
```

**Step 6: Load the Module**

In your Nginx configuration file (`nginx.conf`), load the dynamic module using the `load_module` directive. For example:

```nginx
# nginx.conf

load_module modules/my_module.so;
```

**Step 7: Test and Verify**

- Check your Nginx configuration for any errors: `nginx -t`
- Reload Nginx to apply the configuration: `systemctl reload nginx` (or equivalent)
- Test your module's functionality to ensure it works as expected.

**Step 8: Cleanup (Optional)**

You can remove the temporary build files:

```bash
make clean
```

That's it! You've successfully built and loaded a dynamic module into Nginx. This is a simplified example, and actual module development may involve more complex code and configurations. Be sure to refer to the official Nginx documentation and module development guides for further details and advanced usage.
