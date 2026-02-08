# Rate Limiting

## 🎯 Rate Limiting কি?

**Rate Limiting** হলো API-তে request সংখ্যা সীমিত করা।

```
Why Rate Limiting?
├── DDoS protection
├── Cost control
├── Fair usage
├── Server stability
└── API abuse prevention
```

## 📊 Rate Limiting Algorithms

### ১. Token Bucket
```
┌──────────────────────────────────────────┐
│              Token Bucket                │
├──────────────────────────────────────────┤
│                                          │
│  Bucket capacity: 10 tokens              │
│  Refill rate: 1 token/second             │
│                                          │
│  ┌────────────────────────┐              │
│  │ [●][●][●][●][●][●][ ][ ][ ][ ]       │
│  │     6 tokens available                │
│  └────────────────────────┘              │
│                                          │
│  Request → Token available? → Process    │
│                            → Reject      │
│                                          │
└──────────────────────────────────────────┘

Request 1: Use 1 token ✓ (5 remaining)
Request 2: Use 1 token ✓ (4 remaining)
...
Request 7: No tokens! ✗ (rejected)
Wait 1 sec: 1 token added
```

### ২. Sliding Window
```
┌──────────────────────────────────────────┐
│           Sliding Window Log             │
├──────────────────────────────────────────┤
│                                          │
│  Window: Last 1 minute                   │
│  Limit: 100 requests                     │
│                                          │
│  Time ─────────────────────────────→    │
│                                          │
│  [req][req][req]...[req] | Current      │
│  ├─── Window ────────────┤              │
│                                          │
│  Requests in window: 95                  │
│  Status: 5 more allowed                  │
│                                          │
└──────────────────────────────────────────┘
```

### ৩. Fixed Window
```
┌──────────────────────────────────────────┐
│            Fixed Window                  │
├──────────────────────────────────────────┤
│                                          │
│  Window: 0:00 - 0:59 (1 minute)         │
│  Limit: 100 requests                     │
│                                          │
│  0:00 ──────────── 0:59 | 1:00 ─────    │
│  [Window 1: 100 max]    | [Window 2]     │
│                                          │
│  Problem: Burst at window edges          │
│  0:59 → 100 requests                     │
│  1:00 → 100 requests                     │
│  = 200 requests in 2 seconds!            │
│                                          │
└──────────────────────────────────────────┘
```

## 🔧 Implementation

### Redis Implementation
```python
import redis
import time

r = redis.Redis()

def is_rate_limited(user_id, limit=100, window=60):
    key = f"rate_limit:{user_id}"
    current = r.get(key)
    
    if current is None:
        r.setex(key, window, 1)
        return False
    
    if int(current) >= limit:
        return True
    
    r.incr(key)
    return False

# Usage
if is_rate_limited("user_123"):
    return {"error": "Rate limit exceeded"}, 429
```

## 📨 Response Headers

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000

HTTP/1.1 429 Too Many Requests
Retry-After: 60
```

## 💡 Best Practices

```
✓ Different limits per tier
✓ Clear error messages
✓ Retry-After header
✓ Graceful degradation
✓ Monitor and alert
```

---

🎉 API ডিজাইন সেকশন সম্পূর্ণ!

[সিকিউরিটি →](../09-security/README.md)
