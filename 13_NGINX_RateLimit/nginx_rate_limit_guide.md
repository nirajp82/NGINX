# NGINX Rate Limiting per Hostname (2000 requests per minute)

This guide explains how to configure and understand NGINX rate limiting using `limit_req` for **each hostname (tenant)** with a limit of **2000 requests per minute**, including the burst configuration, internal working of the token bucket algorithm, and example scenarios.

---

## 🔧 Objective

> Rate limit each `$ssl_server_name` (i.e., hostname) to **2000 requests per minute**, allowing brief spikes with a `burst`, and provide step-by-step behavior analysis.

---

## 🧩 Configuration

```nginx
limit_req_zone $ssl_server_name zone=rl_by_host_srv_name:10m rate=2000r/m sync;

limit_req zone=rl_by_host_srv_name burst=200 nodelay;
limit_req_dry_run off;
```

### 🔍 Explanation:

* `$ssl_server_name`: Key used for per-tenant/hostname limiting.
* `zone=rl_by_host_srv_name:10m`: Stores rate limiting data in a 10MB shared memory zone.
* `rate=2000r/m`: 2000 requests per **minute**, or **33.33 requests per second**.
* `burst=200`: Allows short bursts beyond the normal rate.
* `nodelay`: Burst requests are not delayed — they are allowed instantly.
* `limit_req_dry_run off`: Enforces the rate limiting (set to `on` for testing without rejecting requests).

---

## ⏱️ Internal Mechanics: Token Bucket Algorithm

NGINX uses a **leaky token bucket algorithm**:

### 🔁 How It Works:

1. A virtual "bucket" holds tokens.
2. Each token allows **1 request**.
3. The bucket fills at a steady rate: **33.33 tokens per second**.
4. Maximum tokens the bucket can hold = **burst + 1** (i.e., 201 tokens).
5. If requests arrive:

   * If tokens are available → ✅ request allowed
   * If no tokens left:

     * If within burst range → ✅ request allowed
     * If burst limit exceeded → ❌ request rejected
6. `nodelay` means even burst requests are not queued or slowed — they are passed immediately.

### 💡 Important Note on Burst Behavior

* **Burst is not replenished each second.**
* Only the main rate bucket (33.33 tokens/sec) is refilled.
* The `burst` acts as a one-time cushion for excess traffic.
* Once consumed, it stays depleted unless traffic slows down to allow tokens to accumulate again.

---

## 📈 Timeline Example: Burst in Action

### Config:

```nginx
rate=2000r/m   → 33.33 tokens/sec
burst=200
```

### Scenario: Spike of 2500 requests per second for 5 seconds

#### Total Incoming Requests: 2500 \* 5 = 12,500 requests

#### Refill Rate: 33.33 tokens/sec

#### Total Refilled Tokens: 5 \* 33.33 ≈ 167 tokens over 5 seconds

#### Total Capacity with Burst: 201 tokens at time zero

Let’s simulate step by step:

### 🕐 Second 1:

* 2500 requests arrive
* Bucket starts with 201 tokens
* ✅ 201 requests allowed
* ❌ 2299 rejected
* Bucket is empty

### 🕑 Second 2:

* 33 new tokens added
* 2500 requests arrive
* ✅ 33 requests allowed
* ❌ 2467 rejected

### 🕒 Second 3:

* 33 more tokens
* 2500 requests arrive
* ✅ 33 requests allowed
* ❌ 2467 rejected

### 🕓 Second 4:

* 33 more tokens
* 2500 requests arrive
* ✅ 33 requests allowed
* ❌ 2467 rejected

### 🕔 Second 5:

* 33 more tokens
* 2500 requests arrive
* ✅ 33 requests allowed
* ❌ 2467 rejected

> 🔎 **Why doesn’t burst help after second 1?**
> Burst is not a refillable pool. It represents the overflow limit at time zero. Once you consume it, only the base rate (33.33/s) is available.

---

## 📊 Summary of Spike:

| Second    | Incoming  | Allowed | Rejected  |
| --------- | --------- | ------- | --------- |
| 1         | 2500      | 201     | 2299      |
| 2         | 2500      | 33      | 2467      |
| 3         | 2500      | 33      | 2467      |
| 4         | 2500      | 33      | 2467      |
| 5         | 2500      | 33      | 2467      |
| **Total** | **12500** | **333** | **12167** |

---

## ❌ Common Misconceptions

| Statement                                                         | True or False |
| ----------------------------------------------------------------- | ------------- |
| "2000r/m means I can send all 2000 in one second"                 | ❌ False       |
| "NGINX spreads 2000r/m across each second (\~33.33/s internally)" | ✅ True        |
| "Unused capacity from idle seconds carries over to next second"   | ❌ False       |
| "Burst allows a one-time spike briefly above the rate"            | ✅ True        |
| "Burst gets refilled every second"                                | ❌ False       |
| "Once burst is used, only refill rate applies"                    | ✅ True        |

---

## ✅ Best Practices

* Set `burst` based on acceptable spike tolerance.
* Always convert `r/m` to `r/s` internally for understanding actual flow.
* Use `$ssl_server_name` or `$host` for tenant/domain-level rate limiting.
* Start with `limit_req_dry_run on` to test effect.
* If you need sustained high traffic: increase `rate`, not `burst`.

---

## 📌 Testing Tools

To simulate and observe behavior:

```bash
ab -n 10000 -c 500 https://yourdomain.com/api/test
wrk -t4 -c500 -d5s https://yourdomain.com/api/test
```

Look for HTTP 503s to detect rate limit rejection.

---

## 📈 Example: Burst Refill Scenario

### Scenario 1: Idle period followed by burst again

* `rate=2000r/m` → 33.33 tokens/sec
* `burst=200`

🕐 **Seconds 1–6:**

* Only \~5 requests per second
* Each second, \~28.33 tokens remain unused → tokens accumulate (up to max 201)
* By second 6, bucket is full again (burst + 1 tokens)

🕖 **Second 7:**

* Suddenly, 500 requests arrive
* ✅ 201 requests allowed (full bucket tokens)
* ❌ 299 requests rejected
* Bucket empties again

> 🔄 **When is burst available again?**
> After low traffic periods allow tokens to accumulate, the bucket refills to max capacity, enabling new bursts.

---

### Scenario 2: Burst tokens used up, then refill during slowdown, then another burst

| Second | Incoming Requests | Tokens Before Requests | Allowed Requests | Rejected Requests | Tokens After Requests |
| ------ | ----------------- | ---------------------- | ---------------- | ----------------- | --------------------- |
| 1      | 2500              | 201                    | 201              | 2299              | 0                     |
| 2      | 33                | 33                     | 33               | 0                 | 0                     |
| 3      | 0                 | 33                     | 0                | 0                 | 33                    |
| 4      | 0                 | 66                     | 0                | 0                 | 66                    |
| 5      | 0                 | 99                     | 0                | 0                 | 99                    |
| 6      | 0                 | 132                    | 0                | 0                 | 132                   |
| 7      | 0                 | 165                    | 0                | 0                 | 165                   |
| 8      | 0                 | 198                    | 0                | 0                 | 198                   |
| 9      | 0                 | 201                    | 0                | 0                 | 201                   |
| 10     | 500               | 201                    | 201              | 299               | 0                     |

* **Seconds 1-2:** Heavy traffic uses all burst tokens and refill tokens
* **Seconds 3-9:** No traffic, tokens refill steadily at 33/sec to max 201
* **Second 10:** New burst arrives, allowed up to full bucket again

> 💡 **Key Takeaway:**
> Burst tokens refill only when incoming requests slow or stop, allowing tokens to accumulate up to `burst + 1` capacity. This enables handling another burst later.

---

# Diagrams of Token Refill and Burst Behavior in NGINX Rate Limiting

### Token Bucket Model Overview

```
+----------------------------+
|      Token Bucket           |
|                            |
|  +---------------------+   |
|  | Max Capacity = burst+1|  |
|  | (tokens)             |   |
|  +---------------------+   |
|                            |
+----------------------------+
```

*
Tokens accumulate over time at the configured rate (e.g., 33.33 tokens/sec for 2000r/m).\*

---

### Token Refill Over Time

```
Time (seconds) → 
Tokens
 201 |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
     |                                                                  
 150 |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■                              
     |                                                                  
 100 |■■■■■■■■■■■■■■■■■■■■■■                                           
     |                                                                  
  50 |■■■■■■■■■■■                                                   
     |                                                                  
   0 +--------------------------------------------------------------->
       0   1    2    3    4    5    6    7    8    9   10  seconds
```

* At 0 seconds bucket is full (201 tokens).
* Tokens consumed by requests.
* Tokens refill at constant rate (\~33 tokens/s).

---

### Request Handling Flow

```
Incoming Requests
        |
        v
+------------------------+
| Are tokens available?   |--No--> Reject request (HTTP 503)
+------------------------+
        |
       Yes
        |
        v
Consume token from bucket
        |
        v
Allow request
```

---

### Burst Behavior

```
Burst Tokens (extra capacity above base refill rate)
+-------------------+
| burst + 1 tokens   | (initial tokens at t=0)
+-------------------+

After burst tokens are used up:
Only refill rate tokens (e.g., 33/sec) are available.

Tokens accumulate again only when requests slow down,
refilling the bucket up to max capacity (burst+1).
```

![image](https://github.com/user-attachments/assets/6d9ee476-f7b5-4913-bd98-4d38127a51b9)

