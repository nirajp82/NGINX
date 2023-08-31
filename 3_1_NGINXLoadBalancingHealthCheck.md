## Nginx Load Balancing and Health Checks Guide

This guide provides an in-depth understanding of Nginx load balancing concepts, methods, configurations, and various parameters, including active and passive health checks.

### Table of Contents

1. **Introduction to Load Balancing**
2. **Load Balancing Methods**
    - Round Robin
    - Least Connections
    - IP Hash
    - Generic Hash
    - Least Time (NGINX Plus only)
    - Random
3. **Configuring Load Balancing in Nginx**
    - Upstream Block
    - Load Balancing Algorithms
4. **Health Checks**
    - Active Health Checks (Nginx Plus)
    - Passive Health Checks
5. **Configuration Examples**

---

## 1. Introduction to Load Balancing

Load balancing is a technique that distributes incoming network traffic across multiple backend servers, ensuring optimal resource utilization, improved performance, and high availability. Nginx, a versatile web server and reverse proxy, also offers robust load balancing capabilities.

## 2. Load Balancing Methods

### Round Robin

**Description:** Requests are sequentially assigned to each backend server, distributing the load evenly. It will consider server weights into consideration. This method is used by default (there is no directive for enabling it):
```nginx
upstream backend {
   # no load balancing method is specified for Round Robin
   server backend1.example.com;
   server backend2.example.com;
}
```

### Least Connections

**Description:** Requests are directed to the server with the fewest active connections, optimizing resource utilization. It will consider server weights into consideration.
```nginx
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

### IP Hash

**Description:** Requests from the same IP address are consistently routed to the same backend server, maintaining session affinity. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available.

```nginx
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
}
```

If one of the servers needs to be temporarily removed from the load‑balancing rotation, it can be marked with the down parameter in order to preserve the current hashing of client IP addresses. Requests that were to be processed by this server are automatically sent to the next server in the group:


### Generic Hash

**Description:** Allows customized hashing based on variables (for ex: user‑defined key which can be a text string, variable, or a combination) enabling fine-grained control over load distribution. For example, the key may be a paired source IP address and port, or a URI as in this example:

```nginx
upstream backend {
    hash $request_uri consistent;
    server backend1.example.com;
    server backend2.example.com;
}
```

### IP Hash

**Description:** Requests from the same IP address are consistently routed to the same backend server, maintaining session affinity. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available. 

### Least Time (NGINX Plus only) 

**Description:**  For each request, NGINX Plus selects the server with the lowest average latency and the lowest number of active connections, where the lowest average latency is calculated based on which of the following parameters to the least_time directive is included:

    - header – Time to receive the first byte from the server
    - last_byte – Time to receive the full response from the server
    - last_byte inflight – Time to receive the full response from the server, taking into account incomplete requests

```nginx
upstream backend {
    least_time header;
    server backend1.example.com;
    server backend2.example.com;
}
```


## 3. Configuring Load Balancing in Nginx

### Upstream Block

The `upstream` block defines a group of backend servers and the load balancing algorithm to be used.

```nginx
http {
    upstream backend {
        # Load balancing method (e.g., round-robin)
        round-robin;

        # Server definitions
        server backend1.example.com weight=5 max_fails=3 fail_timeout=10s;
        server backend2.example.com weight=3 max_fails=3 fail_timeout=10s;
        server backend3.example.com;
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```

#### Load Balancing Algorithms Parameters

- **weight:** Assigns a weight to each server, influencing the distribution of requests.
- **max_fails:** Sets the maximum number of consecutive failures allowed before considering a server as unhealthy.
- **fail_timeout:** Defines the time during which failures are counted.

## 4. Health Checks

### Active Health Checks (Nginx Plus)

_Active health checks are a feature of Nginx Plus, the commercial version._

Nginx periodically sends requests to backend servers to assess their health. Unhealthy servers are temporarily removed based on failure thresholds and timeouts.

### Passive Health Checks

Passive health checks rely on external monitoring systems to report server health. Nginx reacts to these reports, redirecting traffic away from unhealthy servers.

## 5. Configuration Examples

### Active Health Checks (Nginx Plus)

```nginx
http {
    upstream backend {
        least_conn;

        # Active health check settings
        health_check interval=5s fails=2 passes=3 uri=/health;

        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```

### Passive Health Checks

```nginx
http {
    upstream backend {
        least_conn;

        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        location / {
            proxy_pass http://backend;
            error_page 502 = @fallback;
        }

        location @fallback {
            # Handle case when all backend servers are down
        }
    }
}
```

---

