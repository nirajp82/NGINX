## Overview: Rate Limiting in NGINX

Rate limiting is a technique used to control the amount of incoming traffic to a server or service within a specified time window. It helps protect backend systems from being overwhelmed by too many requests, prevents abuse, and improves overall reliability and performance.

### How Rate Limiting Works in NGINX

NGINX implements rate limiting using the **token bucket algorithm**, which controls how many requests are allowed per key (such as a client IP or hostname) within a given rate (e.g., requests per second or minute).

* A **token bucket** is created per key (like `$ssl_server_name` or `$remote_addr`).
* Tokens accumulate in the bucket at a configured rate (e.g., 2000 requests per minute → about 33 tokens per second).
* Each request consumes one token.
* If tokens are available, requests pass through immediately.
* If the bucket is empty, NGINX can reject or delay requests, depending on configuration.
* The **burst** parameter allows temporary spikes by letting some requests exceed the rate without immediate rejection, acting as a cushion.

### Example Configuration

```nginx
# Define rate limit zone per hostname with 2000 requests per minute
limit_req_zone $ssl_server_name zone=rl_by_host_srv_name:10m rate=2000r/m;

# Apply the limit with a burst of 200 requests allowed instantly
limit_req zone=rl_by_host_srv_name burst=200 nodelay;
```

This setup means each hostname is allowed up to 2000 requests per minute, with short bursts up to 200 extra requests allowed immediately without delay.
