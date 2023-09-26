In Nginx, the `server_name` directive is used within a `server` block to specify the hostnames or IP addresses that the server block should respond to. It's a crucial directive for configuring virtual hosts, allowing you to define which server block should handle incoming requests based on the `Host` header in the HTTP request. The `server_name` directive can take various options and patterns to match different types of hostnames. Here are some common ways to use it:

**1. Single Hostname:**

```nginx
server {
    listen 80;
    server_name example.com;
    # ...
}
```

In this example, the server block responds to requests with the `Host` header set to `example.com`.

**2. Multiple Hostnames:**

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    # ...
}
```

This configuration responds to requests with either `example.com` or `www.example.com` in the `Host` header.

**3. Wildcard Subdomains:**

```nginx
server {
    listen 80;
    server_name *.example.com;
    # ...
}
```

The `*` character serves as a wildcard, and this configuration matches any subdomain of `example.com`. For example, it responds to requests for `sub.example.com` and `another.example.com`.

**4. Regular Expressions:**

```nginx
server {
    listen 80;
    server_name ~^(?<subdomain>.+)\.example\.com$;
    
    location / {
        root /var/www/$subdomain;  # Use the captured subdomain in the root directory
        index index.html index.htm;
    }
}

```

In this example:

The server_name directive uses a regular expression pattern to capture the subdomain. The (?<subdomain>.+) part captures the subdomain and assigns it to the variable $subdomain.

Inside the location block, we use the captured $subdomain variable to construct the root directory for serving files. For example, if a request is made to sub.example.com, Nginx will look for files in the /var/www/sub directory.

This configuration allows you to serve content dynamically based on the subdomain of the incoming request. For different subdomains, you can have separate directories or configurations, allowing you to host multiple subdomains with a single Nginx server block.

**5. IP Addresses:**

```nginx
server {
    listen 80;
    server_name 192.168.1.100;
    # ...
}
```

You can specify IP addresses directly as arguments to the `server_name` directive. The server block responds to requests directed to the specified IP address.

**6. Port Numbers:**

```nginx
server {
    listen 8080;
    server_name example.com:8080;
    # ...
}
```

You can include port numbers in the `server_name` directive to match requests to specific ports. In this case, it responds to requests with `example.com` in the `Host` header and the port number `8080`.

**7. Default Server:**

```nginx
server {
    listen 80 default_server;
    server_name _;
    # ...
}
```

The underscore (`_`) is a special value used to define a default server block. This block is used if no other server block's `server_name` matches the request's `Host` header. It's often used as a catch-all server block.

These are some common ways to use the `server_name` directive in Nginx. By combining these options and patterns, you can configure how Nginx routes incoming requests to different server blocks based on the values in the `Host` header, allowing you to host multiple websites or applications on a single Nginx instance.
