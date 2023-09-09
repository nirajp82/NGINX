Using the NGINX Lua module allows you to extend NGINX's functionality by embedding Lua scripts directly into your NGINX configuration. This can be useful for tasks like authentication, URL rewriting, or dynamic content generation. Here's a step-by-step guide on how to use the NGINX Lua module:

```nginx
# Change to the root directory
cd /

# Create a directory for NGINX setup
mkdir x_np_nginx
cd x_np_nginx

# Download NGINX source code
wget 'https://openresty.org/download/nginx-1.19.3.tar.gz'
tar -xzvf nginx-1.19.3.tar.gz
cd nginx-1.19.3/

# Create directories for NGINX modules
cd /x_np_nginx
mkdir x_np_modules
cd x_np_modules

# Download and install LuaJIT 2.1
wget https://github.com/openresty/luajit2/archive/refs/tags/v2.1-20230410.tar.gz -O luajit2.tar.gz
tar -xzvf luajit2.tar.gz
cd luajit2-2.1-20230410/
make
sudo make install

# Set environment variables for LuaJIT
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.1

# Back to NGINX source directory
cd /x_np_nginx/nginx-1.19.3/

# Configure and install NGINX with required modules
./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --user=nginx --group=nginx --with-http_mp4_module
make
sudo make install

# Download and extract ngx_devel_kit and lua-nginx-module
wget https://github.com/vision5/ngx_devel_kit/archive/refs/tags/v0.3.2.tar.gz -O ngx_devel_kit.tar.gz
tar -xzvf ngx_devel_kit.tar.gz

wget https://github.com/openresty/lua-nginx-module/archive/refs/tags/v0.10.25.tar.gz -O lua-nginx-module.tar.gz
tar -xzvf lua-nginx-module.tar.gz

# Configure and install NGINX with ngx_devel_kit and lua-nginx-module
./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --user=nginx --group=nginx --with-http_mp4_module \
--with-ld-opt="-Wl,-rpath,/usr/local/lib" \
--add-module=/x_np_nginx/nginx-1.19.3/x_np_modules/ngx_devel_kit-0.3.2 \
--add-module=/x_np_nginx/nginx-1.19.3/x_np_modules/lua-nginx-module-0.10.25

make
sudo make install

# Install lua-resty-core and lua-resty-lrucache
cd /x_np_nginx/nginx-1.19.3/x_np_modules
wget https://github.com/openresty/lua-resty-core/archive/refs/tags/v0.1.27.tar.gz -O lua-resty-core.tar.gz
tar -xzvf lua-resty-core.tar.gz
cd lua-resty-core-0.1.27/
make install PREFIX=/usr/share/nginx

cd /x_np_nginx/nginx-1.19.3/x_np_modules
wget https://github.com/openresty/lua-resty-lrucache/archive/refs/tags/v0.13.tar.gz -O lua-resty-lrucache.tar.gz
tar -xzvf lua-resty-lrucache.tar.gz
cd lua-resty-lrucache-0.13
make install PREFIX=/usr/share/nginx

# Add necessary lua_package_path directive to nginx.conf, in the http context
lua_package_path "/usr/share/nginx/lib/lua/?.lua;;";

```
