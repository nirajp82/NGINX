Here's a well-structured and complete **README** file combining all the details you've provided about **Rate Limiting in NGINX**, organized clearly for practical understanding and implementation:

---

# 🧱 NGINX Rate Limiting Guide

## 📘 Overview: Rate Limiting in NGINX

Rate limiting is a technique used to **control incoming traffic** to a server or service within a defined time window. It helps:

* Prevent abuse (e.g., brute-force attacks)
* Reduce backend overload
* Improve stability and performance
* Enforce fair usage across users

NGINX provides a built-in way to apply rate limits to traffic using the **token bucket algorithm**.

---

## ⚙️ How Rate Limiting Works in NGINX

NGINX tracks request rates using a **token bucket algorithm** (not leaky bucket, as previously misunderstood):

* Each key (e.g., client IP or hostname) has an associated **bucket**.
* Tokens are added to the bucket at a steady rate (e.g., 2000r/m = \~33r/s).
* Every request **consumes one token**.
* If tokens are available, the request is **allowed immediately**.
* If the bucket is empty:

  * Requests can be **delayed or rejected**, based on config.
  * A **`burst`** setting allows short-term spikes by queuing extra requests.

---

## 🔧 Core Directives

### ### `limit_req_zone`

Defines rate limiting parameters and a shared memory zone for tracking.

**Syntax:**

```nginx
limit_req_zone $key zone=<zone_name>:<size> rate=<rate>;
```

**Parameters:**

* **Key**: Usually `$remote_addr` (client IP) or `$ssl_server_name` (hostname).
* **Zone**: A named memory zone (`zone=...`) used to track state across workers.

  * Example: `zone=rl_zone:10m` uses 10MB of shared memory.
  * \~1MB stores data for \~16,000 unique keys.
* **Rate**: Request limit (e.g., `10r/s`, `2000r/m`).

> When memory is full, NGINX evicts old entries. If there's no space even after eviction, NGINX returns **503 Service Temporarily Unavailable**.

---

### `limit_req`

Applies the rate limiting to specific locations or servers.

**Syntax:**

```nginx
limit_req zone=<zone_name> [burst=<N>] [nodelay] [dry_run];
```

**Parameters:**

* **`zone`**: The name defined in `limit_req_zone`.
* **`burst`** *(optional)*: Number of requests allowed to exceed the rate temporarily.
* **`nodelay`** *(optional)*: If set, burst requests are processed immediately (not queued).
* **`dry_run`** *(optional)*: Enables monitoring mode without enforcing limits. Helps tune settings without impacting live traffic.

---

## 📌 Example 1: Basic IP-based Rate Limiting

```nginx
http {
    limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;

    server {
        location /login/ {
            limit_req zone=ip_limit burst=5 nodelay;
        }
    }
}
```

* Limits each IP to **10 requests per second**.
* Allows up to **5 extra requests** during a burst.
* `nodelay` ensures those 5 extra are not delayed, but served instantly.
* Good for login endpoints or APIs where abuse prevention is critical.

---

## 📌 Example 2: Hostname-based Rate Limiting

```nginx
http {
    limit_req_zone $ssl_server_name zone=rl_by_host_srv_name:10m rate=2000r/m;

    server {
        location / {
            limit_req zone=rl_by_host_srv_name burst=200 nodelay;
        }
    }
}
```

* Limits each **hostname** to **2000 requests per minute** (\~33r/s).
* Allows **200 extra requests** instantly (`burst=200`).
* Useful in multi-tenant environments (e.g., SaaS platforms using SNI).

---

## 🧠 Best Practices

* **Tune the burst size** to match real-world usage patterns.
* **Use `$binary_remote_addr`** instead of `$remote_addr` to reduce memory footprint.
* **Start with `dry_run` mode** to analyze traffic and refine your limits safely.
* Monitor **status codes (e.g., 503)** to detect when limits are being hit.

---

## 📎 References

* [NGINX Official Docs: Rate Limiting](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)
* [Token Bucket Algorithm Explained](https://en.wikipedia.org/wiki/Token_bucket)

---

Let me know if you want this in Markdown file format or need a diagram added!
