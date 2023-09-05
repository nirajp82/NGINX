```nginx
server {
        listen 80;
        server_name localhost;

        location /authserver {
                # proxy_pass http://123.45.67.890;
                proxy_pass http://authserver.internal;
                proxy_set_header X-Client-IP $remote_addr;
                proxy_set_header Host $host;
        }

        location /appserver {
                # proxy_pass http://123.45.67.901;
                proxy_pass http://appserver.internal;
                proxy_set_header X-Client-IP $remote_addr;
                proxy_set_header Host $host;
        }

        location /version {
                if ($arg_version = "version1") {
                   proxy_pass http://authserver.internal;
                }
                proxy_pass http://appserver.internal;
        }
}
```
