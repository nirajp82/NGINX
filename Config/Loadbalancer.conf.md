```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    upstream backend {
        server 123.45.77.123 max_fails=1 fail_timeout=10s;
        server 123.45.67.222 max_fails=1 fail_timeout=10s;
    }	
	
	server {                                              	
			 listen 80;                                    	
			 server_name localhost;                        	
														  
			 location / {                                  
					proxy_pass http://backend;            
			 }                                             
	}
}
```
