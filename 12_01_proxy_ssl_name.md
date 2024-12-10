### What is SNI, and Why is it Important in SSL/TLS Communication?

**Server Name Indication (SNI)** is an extension to the **SSL/TLS** protocol that allows a client to indicate the hostname it is attempting to connect to at the beginning of the SSL/TLS handshake process. This is particularly important for hosting multiple SSL/TLS certificates on a single IP address, which is common in shared hosting environments, or in cases where multiple services share the same web server.

**Importance in SSL/TLS Communication:**
- **Multiple Certificates on a Single IP**: Without SNI, a server can only present one SSL/TLS certificate per IP address. This is a limitation because most websites today need to serve different SSL certificates for different domain names, even though they all share the same IP address.
- **Efficiency and Cost-Effective**: By allowing multiple certificates on a single IP, SNI allows more efficient resource use, making it cheaper and easier for hosting providers and large-scale services to deploy SSL encryption on many sites.

### How Does SNI Work During the SSL Handshake Process?

The **SSL/TLS handshake** involves several steps where the client and server exchange information to establish a secure connection. SNI is used during the **ClientHello** message, which is the first step of the handshake, to inform the server which hostname (domain) the client is trying to connect to.

Here’s a breakdown of the process:
1. **ClientHello**: When a client (e.g., a web browser) initiates an SSL/TLS connection, it sends a **ClientHello** message to the server. This message contains various pieces of information, such as the supported SSL/TLS versions, cipher suites, and in the case of SNI, the **hostname** (e.g., `www.example.com`) that the client is trying to connect to.
   
   - The SNI information is included in the `server_name` field within the ClientHello message. This is the key part that allows the server to know which domain the client is requesting.

2. **ServerHello**: After receiving the ClientHello, the server checks the SNI field and determines which SSL/TLS certificate to present. If the server is configured to handle multiple domains, it can select the appropriate certificate for the domain name provided in the SNI field.

3. **Certificate Exchange**: The server sends the certificate associated with the requested hostname (domain) back to the client in the ServerHello message. If there is no SNI, or if no certificate matches the provided SNI, the server may return a default certificate, or in some cases, fail the handshake if it cannot identify a valid certificate for the hostname.

4. **Key Exchange and Session Setup**: Once the certificate is validated by the client, the server and client complete the remainder of the handshake (key exchange, secure parameters setup) and establish a secure communication channel.

### How Does SNI Function When There Is a Reverse Proxy, Such as NGINX, Involved?

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

Your NGINX configuration might look something like this:

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

### What happens here?
1. A client requests `https://www.example1.com`.
2. The NGINX reverse proxy receives the request and forwards it to `https://backend1.example.com`.
3. The `proxy_ssl_name $host;` directive tells NGINX to set the SNI field in the SSL/TLS handshake to `www.example1.com` (the hostname from the `Host` header).
4. The backend server `backend1.example.com` sees the SNI and presents the correct SSL certificate for `www.example1.com`.

For `www.example2.com`, the process is similar but with `backend2.example.com`.

### Without `proxy_ssl_name $host;`
If you do not use `proxy_ssl_name $host;`, the SSL/TLS handshake would not include the correct SNI value, and depending on how the backend is configured, this could result in the wrong certificate being presented or an SSL handshake failure.

### Conclusion

- **`proxy_ssl_name $host;`** configures NGINX to forward the hostname from the `Host` header in the incoming request as the **SNI** when establishing an SSL/TLS connection with a backend server.
- This is essential when using multiple SSL certificates on the same backend server, allowing NGINX to correctly direct SSL/TLS connections based on the requested domain.
