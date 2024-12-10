## Table of Contents

1. [What is SNI, and why is it important in SSL/TLS communication?](#what-is-sni-and-why-is-it-important-in-ssltls-communication)
2. [How does SNI work during the SSL handshake process?](#how-does-sni-work-during-the-ssl-handshake-process)
3. [How does SNI function when there is an NGINX reverse proxy with a configuration like `proxy_ssl_name $host;`?](#how-does-sni-function-when-there-is-an-nginx-reverse-proxy-with-a-configuration-like-proxy_ssl_name-host)
4. [Example NGINX Configuration with Multiple Server Blocks](#example-nginx-configuration-with-multiple-server-blocks)
5. [Scenario with Example](#scenario-with-example)
6. [Why is this Configuration Useful?](#why-is-this-configuration-useful)
7. [Conclusion](#conclusion)

### 1. **What is SNI, and why is it important in SSL/TLS communication?**

**Server Name Indication (SNI)** is an extension to the SSL/TLS protocol that allows a client (such as a browser) to indicate the hostname it is attempting to connect to at the start of the SSL/TLS handshake. This allows the server to select the appropriate SSL/TLS certificate based on the hostname provided by the client, even if multiple domains are hosted on the same IP address and server.

#### Importance of SNI:
- **Multiple SSL/TLS certificates on a single IP**: Without SNI, a server could only use one SSL/TLS certificate per IP address, limiting the hosting of multiple secure websites. SNI allows hosting multiple domains with different certificates on the same IP address, making it essential for modern shared hosting environments, CDNs, and cloud infrastructure.
  
- **Optimized resource usage**: In the past, each SSL/TLS-enabled site would require its own dedicated IP address, but with SNI, multiple sites can share the same IP while still maintaining secure HTTPS connections with distinct certificates.

- **Enabling encryption in virtual hosting environments**: It allows web servers to differentiate between domains and serve the correct SSL certificate, even when different domains are hosted on the same server.

---

### 2. **How does SNI work during the SSL handshake process?**

In an SSL/TLS handshake, the process typically begins when a client sends a **ClientHello** message to the server. The **ClientHello** message contains several pieces of information, including the supported cryptographic algorithms, the session ID, and importantly, the **SNI** extension, which includes the hostname the client is trying to connect to.

#### SSL/TLS Handshake with SNI:
1. **ClientHello (with SNI extension)**: When the client initiates the handshake, it sends a **ClientHello** message, which includes the SNI extension. The SNI extension contains the **hostname** of the server the client wants to communicate with (e.g., `www.example.com`). The SNI is transmitted as plain text in the early stages of the handshake, before the encryption starts, allowing the server to choose the correct SSL/TLS certificate.
   
   Example:
   ```
   ClientHello
   ----------------
   SNI: www.example.com
   ```
   
2. **ServerHello (choosing the correct certificate)**: Upon receiving the **ClientHello**, the server uses the hostname specified in the SNI extension to determine which SSL/TLS certificate to present. This is crucial when a server hosts multiple websites (virtual hosts) on the same IP. If the server recognizes the hostname, it sends the appropriate **ServerHello** message with the selected certificate for that domain.

3. **Certificate Exchange**: The server sends the appropriate SSL/TLS certificate corresponding to the domain requested by the client. The certificate includes the server’s public key, which is used for encryption.

4. **Key Exchange & Handshake Completion**: Following this, the client and server proceed with the rest of the SSL/TLS handshake (key exchange, encryption setup, etc.) to establish a secure connection.

#### Without SNI:
- If the SNI extension is not used, the server would not know which certificate to present until after the handshake begins. This results in problems when multiple sites are hosted on the same server, as it can only serve one certificate, often leading to certificate mismatches or errors.

---

### 3. **How does SNI function when there is an NGINX reverse proxy with a configuration like `proxy_ssl_name $host;`?**

In an NGINX reverse proxy setup, SNI plays a crucial role in ensuring that the proxy forwards the correct domain name to the backend server, especially when handling multiple SSL/TLS certificates on the backend.

#### NGINX and SNI:
- In a typical scenario, NGINX is acting as a reverse proxy for several backend servers. If those backend servers are using SSL/TLS (i.e., HTTPS), NGINX needs to forward the correct hostname to the backend server for it to present the right SSL/TLS certificate. This is where the **SNI extension** is useful.

- The directive `proxy_ssl_name $host;` in the NGINX configuration ensures that the **$host** variable (which corresponds to the hostname in the HTTP request) is passed as the SNI to the upstream server during the SSL handshake.

#### Detailed Process:
1. **Client makes request to NGINX**: A client sends an HTTPS request to NGINX with a specific hostname, such as `https://api.example.com`.

2. **NGINX forwards request to backend**: NGINX forwards the request to the upstream backend server. At this point, it includes the **SNI extension** in the SSL/TLS handshake with the backend.

3. **`proxy_ssl_name $host;` directive**: The `$host` variable, which represents the value of the `Host` header in the incoming HTTP request, is passed as the **SNI** value to the upstream server. This allows the backend to choose the correct SSL/TLS certificate for the specific domain.

   For example, if the client connects to `https://api.example.com`, the `proxy_ssl_name $host;` directive ensures that the backend sees `api.example.com` as the SNI value during the SSL handshake. The backend server can then present the correct SSL certificate for `api.example.com`.

4. **SSL handshake with the backend**: When NGINX forwards the request to the backend, it performs its own SSL/TLS handshake with the backend. Thanks to the SNI extension, the backend server knows which certificate to use, even if it hosts multiple domains with different SSL certificates on the same IP.

   Example of the NGINX configuration:
   ```nginx
   location / {
       proxy_pass https://backend_server;
       proxy_ssl_name $host;  # Pass the hostname as the SNI to the backend
       proxy_ssl_certificate /etc/nginx/cert.pem;  # NGINX's certificate for outgoing SSL
       proxy_ssl_certificate_key /etc/nginx/key.pem;
   }
   ```

#### Why is this important?
- **SSL Termination at NGINX**: If NGINX is set up to handle SSL termination (decrypting the HTTPS request and passing it on to the backend via HTTP), the SNI is still needed to ensure that when NGINX makes a new SSL/TLS connection to the backend, it selects the correct certificate.
  
- **Backend SSL Negotiation**: By passing the `$host` variable as the SNI, NGINX ensures that the backend server is aware of which domain is being requested and can present the correct certificate for that domain. This is essential in environments where multiple SSL certificates are in use for different domains, but they share the same IP address.

---

### NGINX Configuration Example:

```nginx
server {
    listen 80;
    server_name www.example1.com;

    location / {
        proxy_pass https://backend1.example.com;
        proxy_ssl_name $host;   # Use the Host header as the SNI value
    }
}

server {
    listen 80;
    server_name www.example2.com;

    location / {
        proxy_pass https://backend2.example.com;
        proxy_ssl_name $host;   # Use the Host header as the SNI value
    }
}
```

### Explanation of the Configuration:

#### 1. **Server Block for `www.example1.com`**:
- **`listen 80;`**: This tells NGINX to listen on port 80 (the default HTTP port). This means NGINX will accept incoming requests over HTTP for this server block.
  
- **`server_name www.example1.com;`**: This directive specifies that this server block will handle requests for the domain `www.example1.com`.

- **Location block (`/`)**: The location block defines how requests to `www.example1.com` should be handled. Here, it passes the requests to a backend server:
  
  - **`proxy_pass https://backend1.example.com;`**: The request is forwarded to the backend server `backend1.example.com` over HTTPS. NGINX will proxy the request to the backend using HTTPS as the protocol.

  - **`proxy_ssl_name $host;`**: This directive tells NGINX to forward the value of the `Host` header from the client request as the **SNI** value when establishing the SSL/TLS connection with the backend. The `$host` variable is dynamically replaced with the hostname that the client is accessing — in this case, `www.example1.com`.

  - **What happens here**: When a client sends a request to `http://www.example1.com`, NGINX forwards the request to `https://backend1.example.com`. The `$host` variable will contain `www.example1.com`, which NGINX uses as the **SNI** when establishing the SSL/TLS handshake with `backend1.example.com`. This ensures that if `backend1.example.com` hosts multiple domains, it will present the correct SSL certificate for `www.example1.com`.

#### 2. **Server Block for `www.example2.com`**:
- **`listen 80;`**: Similarly, this tells NGINX to listen for requests on port 80 (HTTP) for this server block.
  
- **`server_name www.example2.com;`**: This server block handles requests for `www.example2.com`.

- **Location block (`/`)**: Just like the first server block, the location block here handles requests by forwarding them to another backend server:

  - **`proxy_pass https://backend2.example.com;`**: The request is forwarded to the backend server `backend2.example.com` over HTTPS.

  - **`proxy_ssl_name $host;`**: The `$host` variable, which corresponds to `www.example2.com` in the client's request, is passed as the **SNI** to the backend server (`backend2.example.com`).

  - **What happens here**: When a client requests `http://www.example2.com`, NGINX proxies the request to `https://backend2.example.com`. The SNI sent during the SSL handshake will be `www.example2.com`. This ensures that `backend2.example.com` can serve the correct SSL certificate for `www.example2.com`, even if multiple certificates are hosted on the same backend.

### Key Concepts:

1. **Proxying to Different Backends**:
   - Both server blocks are set up to forward traffic to different backend servers (`backend1.example.com` and `backend2.example.com`). NGINX acts as a reverse proxy, forwarding requests to the appropriate backend based on the incoming domain (`www.example1.com` vs. `www.example2.com`).

2. **Using `$host` as SNI**:
   - The crucial part of this configuration is the use of `proxy_ssl_name $host;`. The `$host` variable contains the domain name that the client is trying to access (i.e., the domain in the `Host` header of the HTTP request). This variable is used as the **SNI** value during the SSL/TLS handshake with the backend.
   
   - This allows NGINX to forward the correct **SNI** value to the backend, ensuring that the backend knows which certificate to present, especially when multiple certificates are hosted on the same server.

### Scenario with Example:

Let’s walk through a practical example of how this configuration works:

1. **Client Requests `www.example1.com`**:
   - The client sends an HTTP request to `http://www.example1.com`. 
   
   - NGINX matches the request to the first server block (because of the `server_name www.example1.com` directive).

   - NGINX forwards the request to `https://backend1.example.com` using the `proxy_pass` directive. The important thing here is that NGINX passes the `Host` header (`www.example1.com`) as the SNI during the SSL/TLS handshake with `backend1.example.com`. This tells the backend to present the SSL certificate for `www.example1.com`.

2. **Client Requests `www.example2.com`**:
   - The client sends an HTTP request to `http://www.example2.com`. 
   
   - NGINX matches the request to the second server block (because of the `server_name www.example2.com` directive).

   - NGINX forwards the request to `https://backend2.example.com`. During the SSL/TLS handshake, NGINX sends `www.example2.com` as the SNI value, so `backend2.example.com` knows to present the certificate for `www.example2.com`.

### Why is this Configuration Useful?

This configuration is useful in situations where:

- **Multiple domains are hosted on different backends**, and each backend requires different SSL certificates (even if they share the same IP address).
  
- **SSL Termination at NGINX**: If SSL termination is happening at NGINX (meaning NGINX decrypts the incoming SSL traffic), NGINX still needs to handle secure communication with the backend servers over HTTPS. By using `proxy_ssl_name $host;`, NGINX ensures that each backend server receives the correct SNI, allowing it to select the proper certificate for the corresponding domain.

- **Shared IP Address with Different Certificates**: If `backend1.example.com` and `backend2.example.com` host multiple domains on the same server (with SNI), each server will present the correct certificate for the domain requested by the client, without the need for multiple IP addresses.

### Conclusion:

In this setup, NGINX is acting as a reverse proxy, forwarding requests to different backend servers while using the **SNI** extension to ensure the correct SSL certificate is used for each domain. The `proxy_ssl_name $host;` directive dynamically passes the client’s hostname (from the `Host` header) to the backend, allowing the backend to select the right certificate during the SSL/TLS handshake. This setup is essential for handling multiple domains with different SSL certificates on a shared infrastructure.

### Summary:
- **SNI** is a vital extension to SSL/TLS that allows multiple SSL certificates to be used on a single IP address, enabling secure hosting of multiple domains.
- During the **SSL/TLS handshake**, the client sends the SNI to the server, which uses it to select the correct certificate.
- In an **NGINX reverse proxy** setup, the `proxy_ssl_name $host;` directive ensures that the correct hostname (and hence the correct certificate) is passed along to the backend server during the SSL handshake, allowing for proper SSL/TLS certificate selection.


