```nginx
server {
        listen 80;
        server_name localhost;

        location /admin {
                auth_basic " Basic Authentication ";
                auth_basic_user_file "/etc/nginx/.htpasswd";
        }
}
```

* To create password file:
```sh
  htpasswd -c /etc/nginx/.htpasswd admin
```
