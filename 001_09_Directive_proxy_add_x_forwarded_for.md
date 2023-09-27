The `proxy_add_x_forwarded_for` directive in Nginx is used to control whether or not the client's IP address should be added to the `X-Forwarded-For` header when proxying requests to another server. The `X-Forwarded-For` header is commonly used to track the original client IP address when requests pass through one or more proxy servers.

Here's how the `proxy_add_x_forwarded_for` directive works:

- When `proxy_add_x_forwarded_for` is set to `on` (the default), Nginx appends the client's IP address to the `X-Forwarded-For` header of the request. If the request has already passed through one or more proxy servers, the existing values in the `X-Forwarded-For` header are preserved, and the client's IP address is added as an additional entry. The header typically takes the form of a comma-separated list of IP addresses, with the client's IP address appearing first.

- When `proxy_add_x_forwarded_for` is set to `off`, Nginx does not modify the `X-Forwarded-For` header. This can be useful when you don't want the client's IP address to be exposed to the upstream server, or if you want to control the header manually in your Nginx configuration.

Here's an example of how to use the `proxy_add_x_forwarded_for` directive:

```nginx
location / {
    proxy_pass http://backend;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

In this example:

- We're using Nginx as a reverse proxy to forward requests to a backend server defined by the `proxy_pass` directive.

- We set the `X-Real-IP` header to `$remote_addr`, which represents the client's IP address as seen by Nginx.

- We use the `proxy_add_x_forwarded_for` variable as the value for the `X-Forwarded-For` header. This will append the client's IP address to the existing `X-Forwarded-For` header, preserving any existing values.

With this configuration, as requests pass through each proxy server in the chain, the X-Forwarded-For header will contain a list of IP addresses representing the client's path through the proxies. For example, if the request goes through two proxy servers, the X-Forwarded-For header might look like this:

```X-Forwarded-For: client_ip, proxy1_ip, proxy2_ip```

This allows the final backend server to see the client's original IP address (client_ip) along with the IP addresses of the intermediate proxy servers (proxy1_ip and proxy2_ip).

This is a common configuration when you want to pass along the client's IP address through a chain of proxy servers while using Nginx as the front-end proxy.

Remember that the behavior of `proxy_add_x_forwarded_for` depends on its value and the existing `X-Forwarded-For` header in the incoming request. You can set it to `off` if you need more control over the header or if you don't want to expose the client's IP address to the upstream server.
