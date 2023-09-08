Building dynamic modules in Nginx involves several steps. I'll provide you with a general outline of the process and include an example of building a simple Nginx dynamic module.

```sh
  #Remove NGINX if exist for test only. 
   systemctl stop nginx
   rm -rf ~/nginx-1.24.0
   rm -rf /etc/nginx
   rm -f /user/sbin/nginx
   rm /lib/systemd/system/nginx.service
   systemctl disable nginx
 
```

1. Fetch the NGINX Source (Same as NGINX Production Version)
   ```sh
      yum -y install gcc make zlib-devel pcre-devel openssl-devel wget nano
      cd ~
      sudo su
      wget http://nginx.org/download/nginx-1.24.0.tar.gz
      tar -xzvf nginx-1.24.0.tar.gz
      useradd nginx # (Optional)
      mkdir -p /var/lib/nginx/tmp/  # (Optional)
      chown -R nginx.nginx /var/lib/nginx/tmp/ # (Optional)
      cd nginx-1.24.0/
   ```
   
2. Fetch the module source
   - `yum -y install git`
   - Go to nginx dir. `cd nginx-1.24.0/`
   - `git clone https://github.com/perusio/nginx-hello-world-module.git`
  
3. Build static Module
   - ```sh
     ./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules
     --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log
     --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx
     --user=nginx --group=nginx --with-http_mp4_module
     --add-module=../nginx-hello-world-module
     ```
     make sure configure command finish successfully without any issue, else you will receive an error when running `make` command such as  `make: *** No rule to make target 'build', needed by 'default'.  Stop.`
     
   - `make`
   - `make install`
  
   - if you run the `nginx -V` command you will see the hello world moudule is compiled. (--add-module=../nginx-hello-world-module)
     `
     nginx version: nginx/1.24.0
built by gcc 11.3.1 20221121 (Red Hat 11.3.1-4) (GCC)
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --user=nginx --group=nginx --add-module=../nginx-hello-world-module
     `
      
4. Reference module path within NGINX configuration.
      ```sh
         # Open the nginx config file
         cd /etc/nginx/
         vim nginx.conf
      ```
      ```nginx
          # ...                       
            events {
                worker_connections  1024;
            }
                        
            http {
                include       mime.types;
                default_type  application/octet-stream;
            	# ...
                server {
                        listen 8080;
                      location / {
                            hello_world;
                      }
                }
            	# ...
            
            # ...
      ```

      Test the Nginx configuration for syntax errors and validity
      `nginx -t`
      `curl http://127.0.0.1:8080`


