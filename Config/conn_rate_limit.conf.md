```nginx
http {
      ...

    # Define a shared memory zone that tracks connection counts. It is typically used in conjunction with the limit_conn directive to control the maximum number of simultaneous connections from specific IP addresses to a location or server block.
    limit_conn_zone $binary_remote_addr zone=addr:10m;
	
	server {
        listen 80;
        server_name localhost;

        location /downloads {
                limit_rate 50k; # Limit the download speed to 50 KB/s
                limit_rate_after 10m; # Allow 10 MB of data to be sent before limiting to 50 KB/s
                limit_conn addr 1;  # Allow up to 1 simultaneous connections per IP
        }
	}
 
	...
}	
```
https://github.com/nirajp82/NGINX/blob/main/6_2_DirectiveForConnectionAndRateLimiting.md 
