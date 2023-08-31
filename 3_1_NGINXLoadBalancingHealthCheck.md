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

### Random:

**Description:**  Each request will be passed to a randomly selected server. If the two parameter is specified, first, NGINX randomly selects two servers taking into account server weights, and then chooses one of these servers using the specified method:

	- least_conn – The least number of active connections
	- least_time=header (NGINX Plus) – The least average time to receive the response header from the server ($upstream_header_time)
	- least_time=last_byte (NGINX Plus) – The least average time to receive the full response from the server ($upstream_response_time)
	
The Random load balancing method should be used for distributed environments where multiple load balancers are passing requests to the same set of backends. For environments where the load balancer has a full view of all requests, use other load balancing methods, such as round robin, least connections and least time.

```nginx
upstream backend {
    random two least_time=last_byte;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
    server backend4.example.com;
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

- **weight:** Assigns a weight to each server, influencing the distribution of requests. By default, NGINX distributes requests among the servers in the group according to their weights using the Round Robin method. The weight parameter to the server directive sets the weight of a server; the default is 1:

```nginx
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```
In the example, backend1.example.com has weight 5; the other two servers have the default weight (1), but the one with IP address 192.0.0.1 is marked as a backup server and does not receive requests unless both of the other servers are unavailable. With this configuration of weights, out of every 6 requests, 5 are sent to backend1.example.com and 1 to backend2.example.com.

-  **max_conns:**  With NGINX Plus, it is possible to limit the number of active connections to an upstream server by specifying the maximum number with the max_conns parameter. If the max_conns limit has been reached, the request is placed in a queue for further processing, provided that the queue directive is also included to set the maximum number of requests that can be simultaneously in the queue:

```nginx
upstream backend {
    server backend1.example.com max_conns=3;
    server backend2.example.com;
    queue 100 timeout=70;
}
```

If the queue is filled up with requests or the upstream server cannot be selected during the timeout specified by the optional timeout parameter, the client receives an error.

Note that the max_conns limit is ignored if there are idle keepalive connections opened in other worker processes. 

### HTTP Health Checks
- **slow_start:** The server slow‑start feature prevents a recently recovered server from being overwhelmed by connections, which may time out and cause the server to be marked as failed again.

  In NGINX Plus, slow‑start allows an upstream server to gradually recover its weight from 0 to its nominal value after it has been recovered or became available. This can be done with the slow_start parameter to the server directive:

```nginx
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

The time value (here, 30 seconds) sets the time during which NGINX Plus ramps up the number of connections to the server to the full value.

Note that if there is only a single server in a group, the max_fails, fail_timeout, and slow_start parameters to the server directive are ignored and the server is never considered unavailable.

-  **max_fails:** Sets the maximum number of consecutive failures allowed before considering a server as unhealthy.
-  **fail_timeout:** Defines the time during which failures are counted. It sets the period during which that number of errors or timeouts must occur, as well as how long NGINX waits to try the server again after marking it unhealthy.  (default is 10 seconds).

In the following example, if NGINX fails to send a request to a server or does not receive a response from it 3 times in 30 seconds, it marks the server as unavailable for 30 seconds:

  ```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
}
  ```
Note that if there is only a single server in a group, the fail_timeout and max_fails parameters are ignored and the server is never marked unavailable.
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

References:

https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/ 
