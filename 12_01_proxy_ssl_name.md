Certainly! Let's go through the configuration example step by step, explaining each component in detail with an example and how everything works together. I'll rewrite everything to provide clarity.

---

### NGINX Configuration Explained: Proxy with SNI and Dynamic Backend Selection

---

# Table of Contents

1. [Introduction](#introduction)
2. [What is SNI (Server Name Indication)?](#what-is-sni-server-name-indication)
3. [What is `proxy_ssl_name`?](#what-is-proxy_ssl_name)
4. [What is `proxy_set_header Host`?](#what-is-proxy_set_header-host)
5. [NGINX Configuration Example](#nginx-configuration-example)
6. [Detailed Walkthrough of the Configuration](#detailed-walkthrough-of-the-configuration)
7. [How Components Work Together](#how-components-work-together)
8. [Conclusion](#conclusion)

---

### 1. Introduction

In this document, we’ll explain an NGINX configuration that dynamically selects backend servers based on the requested hostname (domain), manages SSL certificates using **SNI (Server Name Indication)**, and ensures the correct headers are passed to the backend server. We will also break down each part of the configuration and demonstrate how it functions in a real-world example.

---

### 2. What is SNI (Server Name Indication)?

**SNI (Server Name Indication)** is an extension to the SSL/TLS protocol that allows clients to specify which domain they are trying to reach during the SSL handshake. This is critical when hosting multiple SSL certificates on the same IP address and port, which is common for virtual hosting.

For example, if a client connects to `www.example1.com`, the SNI tells the server to present the SSL certificate for `www.example1.com`. Without SNI, the server could only present one certificate for all domains hosted on that IP address.

**Why is it needed?**
- It allows the server to select the correct SSL certificate based on the requested domain.
- It enables virtual hosting of HTTPS sites using different certificates without requiring separate IPs for each domain.

---

### 3. What is `proxy_ssl_name`?

The `proxy_ssl_name` directive in NGINX is used when NGINX is acting as a **reverse proxy** to forward requests to an upstream server over HTTPS. This directive allows you to specify the **SNI hostname** that will be sent to the backend server during the SSL/TLS handshake.

**Purpose**: It ensures that the SSL handshake between NGINX and the backend server uses the correct domain name (hostname) so the server can select the right SSL certificate.

Example:
- When a client requests `www.example1.com`, NGINX forwards the request to the backend, and the backend server uses the `www.example1.com` domain for the SSL handshake.

---

### 4. What is `proxy_set_header Host`?

The `proxy_set_header` directive in NGINX is used to modify HTTP request headers before they are sent to the backend server. Specifically, `proxy_set_header Host $host;` ensures that the backend receives the correct `Host` header, which represents the original domain requested by the client.

**Why is it important?**
- Many backend applications (e.g., web servers or APIs) rely on the `Host` header to route the request or apply domain-specific logic.
- It ensures that even though the request is proxied through NGINX, the backend still knows which domain was originally requested.

---

### 5. NGINX Configuration Example

Here’s a complete NGINX configuration example that demonstrates dynamic backend selection, SSL/TLS handling via SNI, and proper header forwarding:

```nginx
http {
    lua_shared_dict backend_cache 10m;

    server {
        listen 443 ssl;
        server_name www.example1.com www.example2.com;

        location / {
            # Dynamically set the proxy_pass URL based on the requested domain (host)
            set $backend_url "";

            # Use Lua to dynamically choose the backend server URL
            content_by_lua_block {
                local host = ngx.var.host
                -- Conditional logic based on the requested host
                if host == "www.example1.com" then
                    ngx.var.backend_url = "https://backend1.example.com"
                elseif host == "www.example2.com" then
                    ngx.var.backend_url = "https://backend2.example.com"
                else
                    ngx.var.backend_url = "https://default-backend.example.com"
                end
            }

            # Forward the original Host header to the backend server
            proxy_set_header Host $host;

            # Proxy the request to the selected backend server
            proxy_pass $backend_url;

            # Use the Host header for the SNI during the SSL handshake with the backend
            proxy_ssl_name $host;  # Ensures the correct SSL certificate is used by the backend server
        }
    }
}
```

---

### 6. Detailed Walkthrough of the Configuration

Let’s break down this configuration line by line.

#### 1. **Global Configuration**:

```nginx
lua_shared_dict backend_cache 10m;
```
This defines a shared memory zone for Lua scripts to cache data. Here, it’s allocated 10 MB for caching purposes.

#### 2. **Server Block**:

```nginx
server {
    listen 443 ssl;
    server_name www.example1.com www.example2.com;
```
- The server listens on port 443 for HTTPS traffic (with SSL enabled).
- The server block is configured for two domain names: `www.example1.com` and `www.example2.com`.

#### 3. **Location Block**:

```nginx
location / {
    set $backend_url "";
```
- This location block handles all incoming requests (`/` means all paths).
- The `$backend_url` variable is initialized to an empty string. This variable will later hold the backend server URL.

#### 4. **Lua Script for Dynamic Backend Selection**:

```nginx
content_by_lua_block {
    local host = ngx.var.host
    if host == "www.example1.com" then
        ngx.var.backend_url = "https://backend1.example.com"
    elseif host == "www.example2.com" then
        ngx.var.backend_url = "https://backend2.example.com"
    else
        ngx.var.backend_url = "https://default-backend.example.com"
    end
}
```
- The `content_by_lua_block` is used to execute a Lua script dynamically.
- The script checks the hostname (`$host`) of the incoming request:
  - If the hostname is `www.example1.com`, it sets the backend URL to `https://backend1.example.com`.
  - If the hostname is `www.example2.com`, it sets the backend URL to `https://backend2.example.com`.
  - Otherwise, it defaults to a fallback backend URL: `https://default-backend.example.com`.

#### 5. **Proxying Request to Backend**:

```nginx
proxy_set_header Host $host;
proxy_pass $backend_url;
```
- **`proxy_set_header Host $host;`**: This sets the `Host` header in the request forwarded to the backend server. The `$host` variable holds the original domain requested by the client.
- **`proxy_pass $backend_url;`**: This forwards the request to the dynamically selected backend URL, which was set by the Lua script.

#### 6. **SNI Handling with `proxy_ssl_name`**:

```nginx
proxy_ssl_name $host;
```
- **`proxy_ssl_name`** tells NGINX to use the `$host` (original domain) value for the SNI field when establishing the SSL connection to the backend.
- This ensures that the correct SSL certificate is selected based on the requested domain (e.g., `www.example1.com`).

---

### 7. How Components Work Together

Here’s how all the pieces work together in action:

1. **Client Request**: The client makes a request to `https://www.example1.com`.
2. **SNI and SSL Handshake**: NGINX establishes an SSL connection with the backend. The `proxy_ssl_name $host;` directive ensures that the `SNI` field in the SSL handshake contains `www.example1.com`, so the backend server can present the correct SSL certificate.
3. **Dynamic Backend Selection**: Based on the requested domain (`$host`), the Lua script dynamically selects the backend server URL:
   - For `www.example1.com`, it selects `https://backend1.example.com`.
   - For `www.example2.com`, it selects `https://backend2.example.com`.
4. **Proxying the Request**: The request is then forwarded to the appropriate backend server, with the `Host` header set correctly via `proxy_set_header Host $host;`.
5. **Backend Processing**: The backend server receives the correct `Host` header and processes the request accordingly.

---

### 8. Conclusion

This NGINX configuration enables you to:

- **Dynamically route requests** to different backend servers based on the domain name (`$host`).
- Ensure the **correct SSL certificate** is used for the SSL/TLS handshake using **SNI**.
- Forward the correct **`Host` header** to the backend server, ensuring that domain-specific logic is applied.

This setup is perfect for hosting multiple domains on the same server with different SSL certificates, and it allows you to dynamically manage backend servers based on the incoming request.
