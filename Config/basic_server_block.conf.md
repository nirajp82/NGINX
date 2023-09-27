```nginx
server {
        listen 80;
        server_name localhost;

        location / {
           proxy_pass http://172.31.38.222;
           proxy_set_header Host $host;
        }

         location /hello-world {
             add_header Content-Type text/plain;
             return 200 "Hello, world, IPAddr: $server_addr, Requested URL: $request_uri\n";
         }      
}
```
