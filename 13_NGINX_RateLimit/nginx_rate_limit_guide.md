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


