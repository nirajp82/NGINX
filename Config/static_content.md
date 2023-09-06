```
server {
        listen 80;
        server_name localhost;

        location / {
           proxy_pass http://172.31.38.222;
           proxy_set_header Host $host;
        }

        location ~* \.(css|js|jpe?g|JPG|png) {
           root /var/www/assets;
           try_files $uri $uri/ @backend;
        }

        location @backend {
           proxy_pass http://172.31.38.222;
           proxy_set_header Host $host;
           proxy_set_header X_Real_IP $remote_addr;
        }
}
```
