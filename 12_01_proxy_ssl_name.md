### Explanation of `proxy_ssl_name $host;`

In NGINX, when you configure it as a reverse proxy to forward requests to an HTTPS backend server, it may need to communicate with that backend server over SSL/TLS. The `proxy_ssl_name` directive allows you to specify the **hostname** that NGINX should use when establishing the SSL/TLS connection with the backend server.

- **`proxy_ssl_name`**: This directive is used to set the **hostname** NGINX uses when performing the SSL handshake with the upstream (backend) server. It is especially useful when you're using SSL/TLS with the backend and need to ensure that the correct certificate is used during the handshake, based on the hostname of the request.

- **`$host`**: This is a built-in NGINX variable that represents the **hostname** (or domain name) from the incoming request's `Host` header. In an HTTP request, the `Host` header typically contains the domain name the client is trying to reach (e.g., `www.example.com`). The `$host` variable captures this value.

So, when you use:

```nginx
proxy_ssl_name $host;
```

it tells NGINX to take the **hostname** from the client's request (the value of the `Host` header) and use that as the `Server Name` for the SSL handshake with the backend server.

### Why is This Important?

This is important for the following reasons:

1. **SNI (Server Name Indication)**: The `$host` variable allows NGINX to pass the correct hostname to the backend server during the SSL/TLS handshake, which is crucial for **SNI** (Server Name Indication). SNI is an SSL/TLS extension that allows multiple SSL certificates to be hosted on the same IP address. When the backend server hosts multiple SSL certificates, the server needs to know which certificate to present based on the requested hostname. By using `$host`, you ensure that the correct certificate is selected by the backend server.

2. **Multiple Hostnames/Virtual Hosts**: If your backend server hosts multiple virtual hosts and each virtual host has its own SSL certificate, setting `proxy_ssl_name $host;` ensures that the backend server uses the correct certificate for the requested hostname.

3. **Backend SSL Configuration**: In some cases, the backend server may perform certificate validation based on the hostname in the SNI field. If the hostname in the SNI doesn't match the expected hostname for that backend, the server may reject the connection. By forwarding the original `Host` header value (`$host`), you ensure that the SSL/TLS connection with the backend is properly established.

Certainly! Let's explore how you can dynamically set the `proxy_pass` URL **on the fly** based on certain conditions (such as domain-based routing or other variables) and still use the `proxy_ssl_name` directive to ensure that the correct **SNI** (Server Name Indication) is forwarded to the backend server.

This will involve using **NGINX with Lua** to dynamically set the `proxy_pass` value while maintaining the correct SNI configuration using `proxy_ssl_name`. Lua in NGINX can be used to implement logic that conditionally sets which backend server a request should be forwarded to. The `proxy_ssl_name` directive can still be used to pass the **hostname** as the SNI value to ensure the correct SSL/TLS certificate is used by the backend server.

### NGINX Configuration with Lua Example

```nginx
http {
    lua_shared_dict backend_cache 10m;

    server {
        listen 443 ssl;
        server_name www.example1.com www.example2.com;

        location / {
            # Dynamically set the proxy_pass based on the host or other condition
            set $backend_url "";

            # Use Lua to dynamically select the backend server
            content_by_lua_block {
                local host = ngx.var.host
                -- Conditional logic based on host or other factors
                if host == "www.example1.com" then
                    ngx.var.backend_url = "https://backend1.example.com"
                elseif host == "www.example2.com" then
                    ngx.var.backend_url = "https://backend2.example.com"
                else
                    ngx.var.backend_url = "https://default-backend.example.com"
                end
            }

            # Proxy request to the dynamically chosen backend server
            proxy_pass $backend_url;

            # Use the Host header for the SNI during the SSL handshake
            proxy_ssl_name $host;   # Ensure the correct SSL certificate is selected based on the domain
        }
    }
}
```

### **Detailed Explanation**

This configuration demonstrates how you can conditionally select which backend server to proxy the request to based on the `Host` header (or any other variable). At the same time, it ensures that the correct **SNI** is used when making the SSL/TLS connection to the backend server. 

Let’s break it down in detail:

---

### **1. Lua Configuration Block (`content_by_lua_block`)**

```nginx
content_by_lua_block {
    local host = ngx.var.host
    -- Conditional logic based on host or other factors
    if host == "www.example1.com" then
        ngx.var.backend_url = "https://backend1.example.com"
    elseif host == "www.example2.com" then
        ngx.var.backend_url = "https://backend2.example.com"
    else
        ngx.var.backend_url = "https://default-backend.example.com"
    end
}
```

- **Lua Scripting**: This block uses **Lua** to define custom logic that runs when a request is received. The Lua code checks the value of the `$host` variable (which contains the hostname in the HTTP request's `Host` header).
- **Conditionally Set `proxy_pass`**: Based on the hostname (or any other custom condition), we set the `$backend_url` variable dynamically. This can be as complex as needed — for example, based on specific request headers, geographical location of the client, or other logic you might want to implement.
  
  - If the request is for `www.example1.com`, it forwards to `https://backend1.example.com`.
  - If the request is for `www.example2.com`, it forwards to `https://backend2.example.com`.
  - If the host does not match any of the conditions, it defaults to `https://default-backend.example.com`.

---

### **2. `proxy_pass` Directive**

```nginx
proxy_pass $backend_url;
```

- **Dynamically Set `proxy_pass`**: The `$backend_url` variable, which is set dynamically by the Lua block, is passed to the `proxy_pass` directive. This means that NGINX will forward the request to the backend server defined in the Lua logic.
- The URL can change based on the condition set in the Lua block. This provides a flexible way to route requests dynamically to different backend servers based on the request details.

---

### **3. Using `proxy_ssl_name` to Pass the SNI Value**

```nginx
proxy_ssl_name $host;   # Ensure the correct SSL certificate is selected based on the domain
```

- **Passing the Host as SNI**: The `proxy_ssl_name` directive is crucial when making an SSL/TLS connection to the backend server. The value of `$host` is passed as the **SNI** to the backend server. This allows the backend to choose the appropriate SSL certificate for the domain requested by the client.
- Even though the backend server (`backend1.example.com`, `backend2.example.com`, etc.) may host multiple domains, using `proxy_ssl_name $host;` ensures that the **correct certificate** is presented to the client during the handshake.
- The SNI mechanism is vital when multiple SSL certificates are hosted on the same IP, as it tells the backend server which domain to use for the SSL handshake and which certificate to present.

---

### **How It Works Together:**

1. **Incoming Request**:
   - A client makes a request to `https://www.example1.com`.

2. **NGINX Receives the Request**:
   - NGINX matches the request to the `server_name` directive (`www.example1.com` or `www.example2.com`).
   - The `content_by_lua_block` is executed, and it sets the `$backend_url` variable dynamically.
     - For `www.example1.com`, the Lua script sets `$backend_url` to `https://backend1.example.com`.

3. **Dynamic `proxy_pass`**:
   - The `proxy_pass` directive then forwards the request to the backend URL stored in `$backend_url`, i.e., `https://backend1.example.com`.
  
4. **SNI Handling**:
   - The `proxy_ssl_name` directive uses the `$host` variable, which will be `www.example1.com` in this case, to pass the correct **SNI value** to `backend1.example.com` during the SSL handshake.
   - This ensures that even though multiple domains might be hosted on the same backend server (using SNI), the correct SSL certificate for `www.example1.com` will be selected and presented.

5. **SSL/TLS Handshake**:
   - The backend server (`backend1.example.com`) receives the connection request and identifies the correct SSL certificate to use based on the SNI (`www.example1.com`).
  
6. **Response**:
   - The backend server processes the request, and the response is sent back through NGINX to the client.

---

### **Practical Use Cases**

This configuration is highly useful in scenarios where:

1. **Multiple Domains with Different Backends**: 
   - If you have multiple domains hosted on different backend servers but want to forward requests based on the domain name dynamically, this setup allows you to handle such routing efficiently.
   
2. **Complex Routing Logic**:
   - You may want to route traffic not just based on the domain name but based on custom conditions, such as headers, cookies, or even the request's geographical origin. Lua can handle complex logic for routing the traffic to different backends.

3. **Multiple SSL Certificates for the Same Backend**:
   - If you’re using **SNI** on your backend servers (i.e., multiple SSL certificates on the same IP), this configuration ensures that the backend server uses the correct SSL certificate based on the domain requested by the client, even though the backend could be serving multiple domains.
   
---

### **Conclusion**

This configuration shows how to combine **Lua scripting** in NGINX with **SNI** to dynamically set the `proxy_pass` directive and ensure the correct **SSL certificate** is used based on the requested domain. By setting the backend server dynamically using Lua (`$backend_url`), and ensuring that `proxy_ssl_name $host;` is used, NGINX can forward traffic securely to the appropriate backend with the correct SNI for SSL/TLS certificate selection.

