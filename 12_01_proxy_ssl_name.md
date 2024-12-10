### Table of Contents

1. [Introduction to SNI in SSL/TLS Communication](#what-is-sni-and-why-is-it-important-in-ssltls-communication)
   - 1.1 [What is SNI?](#what-is-sni)
   - 1.2 [Why is SNI Important in SSL/TLS Communication?](#why-is-sni-important-in-ssltls-communication)
     - 1.2.1 [Multiple Certificates on a Single IP](#multiple-certificates-on-a-single-ip)
     - 1.2.2 [Efficiency and Cost-Effective Use of SSL](#efficiency-and-cost-effective-use-of-ssl)

2. [How SNI Works During the SSL/TLS Handshake](#how-does-sni-work-during-the-ssltls-handshake-process)
   - 2.1 [ClientHello and the SNI Field](#clienthello-and-the-sni-field)
   - 2.2 [ServerHello and Certificate Exchange](#serverhello-and-certificate-exchange)
   - 2.3 [Key Exchange and Session Setup](#key-exchange-and-session-setup)

3. [SNI with Reverse Proxy (NGINX)](#how-does-sni-function-when-there-is-a-reverse-proxy-such-as-nginx-involved)
   - 3.1 [Using `proxy_ssl_name` Directive](#using-proxy_ssl_name-directive)
   - 3.2 [The Role of the `$host` Variable](#the-role-of-the-host-variable)
   - 3.3 [How SNI Works with NGINX as a Reverse Proxy](#how-sni-works-with-nginx-as-a-reverse-proxy)

4. [Why `proxy_ssl_name $host;` is Important](#why-is-this-important)
   - 4.1 [SNI (Server Name Indication)](#sni-server-name-indication)
   - 4.2 [Multiple Hostnames/Virtual Hosts](#multiple-hostnamesvirtual-hosts)
   - 4.3 [Backend SSL Configuration](#backend-ssl-configuration)

5. [Example Scenario with NGINX Reverse Proxy](#example-scenario)
   - 5.1 [NGINX Configuration with Multiple Backends](#nginx-configuration-with-multiple-backends)
   - 5.2 [Process Flow for `example1.com` and `example2.com`](#process-flow-for-example1com-and-example2com)

6. [What Happens Without `proxy_ssl_name $host;`](#without-proxy_ssl_name-host)
   
7. [Conclusion](#conclusion)

---

### 1. Introduction to SNI in SSL/TLS Communication

#### 1.1 What is SNI?

**Server Name Indication (SNI)** is an extension to the **SSL/TLS** protocol that allows a client to indicate the hostname it is attempting to connect to at the beginning of the SSL/TLS handshake process.

#### 1.2 Why is SNI Important in SSL/TLS Communication?

##### 1.2.1 Multiple Certificates on a Single IP

Without SNI, a server can only present one SSL/TLS certificate per IP address, limiting the ability to serve multiple domain names securely from the same server.

##### 1.2.2 Efficiency and Cost-Effective Use of SSL

SNI allows for the efficient use of resources, enabling multiple certificates to be hosted on a single IP, which reduces costs and complexity, especially in shared hosting and large-scale cloud environments.

---

### 2. How SNI Works During the SSL/TLS Handshake

#### 2.1 ClientHello and the SNI Field

In the initial **ClientHello** message, the client includes the **SNI** field, which indicates the hostname that the client wants to connect to.

#### 2.2 ServerHello and Certificate Exchange

Upon receiving the **ClientHello** message, the server looks up the SNI value and selects the appropriate certificate to return in the **ServerHello** message.

#### 2.3 Key Exchange and Session Setup

After the certificate exchange, the client and server proceed with key exchange and finalizing the secure connection.

---

### 3. SNI with Reverse Proxy (NGINX)

#### 3.1 Using `proxy_ssl_name` Directive

The `proxy_ssl_name` directive is used in NGINX when proxying requests to an HTTPS backend. It ensures that the correct hostname is used in the SSL handshake with the backend server.

#### 3.2 The Role of the `$host` Variable

The `$host` variable in NGINX represents the **hostname** from the `Host` header of the incoming HTTP request. This value is used to specify the **SNI** when NGINX forwards the request to the backend server.

#### 3.3 How SNI Works with NGINX as a Reverse Proxy

NGINX uses the `$host` variable in the `proxy_ssl_name` directive to set the **SNI** during the SSL/TLS handshake with the backend server.

---

### 4. Why `proxy_ssl_name $host;` is Important

#### 4.1 SNI (Server Name Indication)

By forwarding the correct hostname using the `$host` variable, NGINX ensures that the correct certificate is selected by the backend server, which is critical for multiple SSL certificates hosted on the same IP.

#### 4.2 Multiple Hostnames/Virtual Hosts

When a backend server hosts multiple virtual hosts with different SSL certificates, using `$host` ensures that each hostname receives the correct certificate, avoiding certificate mismatches.

#### 4.3 Backend SSL Configuration

Some backend servers validate the hostname during the SSL handshake. By passing the correct SNI value (`$host`), NGINX helps ensure that the SSL handshake is successful and the connection is established securely.

---

### 5. Example Scenario with NGINX Reverse Proxy

#### 5.1 NGINX Configuration with Multiple Backends

The following NGINX configuration sets up reverse proxying with SSL to two backend servers, each handling a different domain.

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

#### 5.2 Process Flow for `example1.com` and `example2.com`

1. Client requests `https://www.example1.com`.
2. NGINX forwards the request to `https://backend1.example.com` with the SNI set to `example1.com`.
3. Backend1 responds with the appropriate certificate for `example1.com`.
4. A similar process occurs for `www.example2.com` and `backend2.example.com`.

---

### 6. What Happens Without `proxy_ssl_name $host;`

Without the `proxy_ssl_name $host;` directive, NGINX would not include the correct SNI value during the SSL handshake. This can lead to:

- SSL/TLS handshake failures
- The wrong SSL certificate being presented by the backend server

---

### 7. Conclusion

The `proxy_ssl_name $host;` directive in NGINX plays a critical role in ensuring that the correct SSL certificate is used when proxying requests to an HTTPS backend. By forwarding the hostname from the request's `Host` header, it allows NGINX to properly use **SNI** during the SSL/TLS handshake, ensuring that multi-domain certificates are handled correctly and securely.

