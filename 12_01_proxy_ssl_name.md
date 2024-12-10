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

### Example Scenario

Let’s say NGINX is acting as a reverse proxy, and you have two backend servers, each handling a different domain name over HTTPS:

- `www.example1.com` is handled by `backend1.example.com`
- `www.example2.com` is handled by `backend2.example.com`

Here’s a more detailed and comprehensive explanation of the provided **NGINX configuration** with a focus on each element, especially in the context of using **SNI (Server Name Indication)** to forward HTTPS requests to backend servers.

---

### NGINX Configuration Example Breakdown

```nginx
server {
    listen       443 ssl;
    server_name www.example1.com;

    location / {
        proxy_pass https://backend1.example.com;
        proxy_ssl_name $host;   # Use the Host header as the SNI value
    }
}

server {
    listen       443 ssl;
    server_name www.example2.com;

    location / {
        proxy_pass https://backend2.example.com;
        proxy_ssl_name $host;   # Use the Host header as the SNI value
    }
}
```

### **Explanation of the Configuration**

This **NGINX configuration** contains two server blocks. Each block handles HTTPS requests (port 443) for different domain names (`www.example1.com` and `www.example2.com`) and forwards those requests to different backend servers, maintaining SSL/TLS security through the **SNI** mechanism.

Let’s break down each part of the configuration and how it works.

---

### **General Structure and Purpose**

- **Server Block**: Each `server` block in NGINX corresponds to a different virtual host configuration, determining how NGINX will handle requests for a specific domain. In this case, two domains (`www.example1.com` and `www.example2.com`) are configured to handle incoming HTTPS requests, proxying them to different backend servers.

- **HTTPS (SSL)**: Both server blocks are configured to listen on port `443`, which is the default port for HTTPS traffic. SSL/TLS encryption is enabled by the `ssl` keyword after the `listen` directive.

---

### **Server Block 1: Handling Requests for `www.example1.com`**

```nginx
server {
    listen       443 ssl;
    server_name www.example1.com;

    location / {
        proxy_pass https://backend1.example.com;
        proxy_ssl_name $host;   # Use the Host header as the SNI value
    }
}
```

#### 1. **`listen 443 ssl;`**
- This directive tells NGINX to listen on **port 443**, which is the default port for HTTPS traffic, and to enable SSL/TLS encryption. It means that this server block will handle incoming secure (HTTPS) requests.

#### 2. **`server_name www.example1.com;`**
- This specifies that the server block will handle requests for `www.example1.com`. When a client makes a request to `https://www.example1.com`, NGINX will match the hostname in the request to the `server_name` directive and process the request using this block.

#### 3. **`location / { ... }`**
- This block defines how the server should handle requests that match the root URL path (`/`), meaning requests for the entire site. Here, the requests are being **proxied** to an upstream server.

#### 4. **`proxy_pass https://backend1.example.com;`**
- The `proxy_pass` directive tells NGINX to forward the incoming request to the backend server `https://backend1.example.com`. This means that instead of serving content directly, NGINX acts as a reverse proxy and forwards the HTTPS request to the backend server over a secure connection.

#### 5. **`proxy_ssl_name $host;`**
- **SNI (Server Name Indication)** is used here to forward the correct **hostname** to the backend server during the SSL/TLS handshake. The `$host` variable contains the value of the `Host` header from the client request, which in this case will be `www.example1.com`.
- When NGINX makes the SSL/TLS connection to `backend1.example.com`, it sends `www.example1.com` as the **SNI** value during the handshake. This allows `backend1.example.com` to present the correct SSL certificate for `www.example1.com`, assuming it hosts multiple certificates on the same IP address (a common scenario with SNI).

---

### **Server Block 2: Handling Requests for `www.example2.com`**

```nginx
server {
    listen       443 ssl;
    server_name www.example2.com;

    location / {
        proxy_pass https://backend2.example.com;
        proxy_ssl_name $host;   # Use the Host header as the SNI value
    }
}
```

The second server block is structured similarly to the first but is responsible for handling requests to `www.example2.com`.

#### 1. **`listen 443 ssl;`**
- Again, this tells NGINX to listen on port `443` with SSL/TLS encryption for incoming HTTPS traffic.

#### 2. **`server_name www.example2.com;`**
- This directive specifies that this server block will handle requests for the domain `www.example2.com`. Requests for this domain will be processed by this block, separate from the first.

#### 3. **`location / { ... }`**
- Similar to the first server block, this location block defines how to handle requests to the root path (`/`). These requests will be proxied to another backend server.

#### 4. **`proxy_pass https://backend2.example.com;`**
- Requests to `www.example2.com` are forwarded to `https://backend2.example.com`. The communication between NGINX and the backend server remains secure with HTTPS.

#### 5. **`proxy_ssl_name $host;`**
- The `$host` variable is passed as the **SNI** value to `backend2.example.com` during the SSL handshake. If `backend2.example.com` hosts multiple SSL certificates for different domains (as is common with SNI), this ensures that the backend serves the correct SSL certificate for `www.example2.com`.

---

### **Key Concepts and How SNI Works**

1. **SNI (Server Name Indication)**:
   - SNI is an extension to the SSL/TLS protocol that allows a client (e.g., a browser) to indicate which domain it is trying to reach during the SSL handshake. This is crucial when multiple websites are hosted on the same IP address but require different SSL certificates.
   - In the context of the provided configuration, NGINX is forwarding the value of the `Host` header (i.e., `$host`) to the backend server as the SNI value. This ensures that the backend server knows which certificate to use when responding to the client.

2. **Proxying HTTPS Requests**:
   - The `proxy_pass` directive is used to forward requests to an upstream server. In this case, NGINX is acting as a reverse proxy, meaning it forwards client requests to another server (backend) for processing.
   - The connection between NGINX and the backend server is made over HTTPS, ensuring that traffic is encrypted during the entire communication process.

3. **Why `proxy_ssl_name $host` is Important**:
   - By using the `$host` variable for the `proxy_ssl_name`, NGINX ensures that the correct SNI value is sent to the backend server. If `backend1.example.com` or `backend2.example.com` hosts multiple domains and certificates, this ensures that the server can present the appropriate certificate for the requested domain (`www.example1.com` or `www.example2.com`).
   - Without this configuration, the backend might not be able to determine which certificate to use, leading to SSL errors.

---

### **Practical Scenario**

Let’s walk through an example of how this configuration works:

1. A client requests `https://www.example1.com`.
2. NGINX receives the request and matches it to the first server block (`server_name www.example1.com`).
3. NGINX forwards the request to `https://backend1.example.com` using the `proxy_pass` directive.
4. During the SSL/TLS handshake, NGINX sends the SNI value as `www.example1.com` to `backend1.example.com`, allowing it to present the correct certificate.
5. The backend responds to NGINX with the content, and NGINX forwards it back to the client.

Similarly, for a request to `https://www.example2.com`, NGINX will follow the same steps, but forward the request to `https://backend2.example.com`, ensuring that the correct SSL certificate is used by sending the appropriate SNI value (`www.example2.com`).

---

### **Conclusion**

In summary:
- This NGINX configuration is designed to handle multiple HTTPS domains (`www.example1.com` and `www.example2.com`) by forwarding requests to different backend servers.
- The `proxy_ssl_name $host;` directive ensures that the correct **SNI value** is passed to the backend servers, enabling them to present the appropriate SSL certificates.
- This setup is useful when hosting multiple SSL-secured sites on the same NGINX instance but needing to proxy traffic to backend servers that require different SSL certificates for each domain.
