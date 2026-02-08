# Rate Limiter Design

## 🎯 Requirements

```
Functional:
- Limit requests per user/IP
- Different limits for different endpoints
- Return proper error codes (429)
- Configurable limits

Non-Functional:
- Low latency (< 1ms overhead)
- Distributed (multiple servers)
- Accurate counting
- Minimal memory usage
```

## 📊 Rate Limiting Algorithms

### ১. Token Bucket

```
┌─────────────────────────────────────────────────────────────────┐
│                    Token Bucket                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────┐                       │
│  │           Token Bucket              │                       │
│  │                                     │                       │
│  │   Capacity: 10 tokens               │                       │
│  │   Refill Rate: 2 tokens/second      │                       │
│  │                                     │                       │
│  │   ┌───┬───┬───┬───┬───┬───┬───┐   │                       │
│  │   │ ● │ ● │ ● │ ● │ ● │   │   │   │ Tokens = 5            │
│  │   └───┴───┴───┴───┴───┴───┴───┘   │                       │
│  │                                     │                       │
│  │   Request arrives:                  │                       │
│  │   - If tokens >= 1: Allow, remove 1 │                       │
│  │   - If tokens = 0: Reject           │                       │
│  │                                     │                       │
│  └─────────────────────────────────────┘                       │
│                                                                 │
│  Pros: Allows bursts up to bucket size                        │
│  Cons: Memory for each bucket                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ২. Sliding Window Log

```
┌─────────────────────────────────────────────────────────────────┐
│                  Sliding Window Log                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Store timestamp of each request                               │
│  Limit: 10 requests per minute                                 │
│                                                                 │
│  Window: [now - 60s, now]                                      │
│                                                                 │
│  Log for user_123:                                             │
│  [10:00:05, 10:00:15, 10:00:25, 10:00:35, 10:00:45]           │
│                                                                 │
│  New request at 10:01:00:                                      │
│  1. Remove timestamps < 10:00:00                               │
│  2. Count remaining = 5                                        │
│  3. If count < 10: Allow, add 10:01:00                        │
│  4. If count >= 10: Reject                                     │
│                                                                 │
│  Pros: Very accurate                                           │
│  Cons: High memory usage (store all timestamps)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ৩. Sliding Window Counter

```
┌─────────────────────────────────────────────────────────────────┐
│                 Sliding Window Counter                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Combine fixed window counters with weighted average           │
│                                                                 │
│  Time: 10:01:15 (15 seconds into current window)              │
│  Previous window (10:00): 8 requests                           │
│  Current window (10:01): 5 requests                            │
│                                                                 │
│  Weighted count = prev × (1 - overlap%) + curr × overlap%     │
│                 = 8 × (45/60) + 5 × (15/60)                   │
│                 = 6 + 1.25                                     │
│                 = 7.25 requests                                │
│                                                                 │
│  Limit: 10                                                     │
│  7.25 < 10 → Allow                                            │
│                                                                 │
│  Pros: Memory efficient, reasonably accurate                   │
│  Cons: Approximate (not exact)                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 🏗️ Distributed Rate Limiter

```
┌─────────────────────────────────────────────────────────────────┐
│              Distributed Rate Limiter                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Challenge: Multiple servers need shared state                 │
│                                                                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │                                                     │       │
│  │   Server 1        Server 2        Server 3         │       │
│  │      │               │               │             │       │
│  │      └───────────────┴───────────────┘             │       │
│  │                      │                              │       │
│  │                      ↓                              │       │
│  │         ┌────────────────────────┐                 │       │
│  │         │        Redis           │                 │       │
│  │         │   (Centralized Store)  │                 │       │
│  │         │                        │                 │       │
│  │         │  user:123:count = 8    │                 │       │
│  │         │  user:123:window = 1706│                 │       │
│  │         │                        │                 │       │
│  │         └────────────────────────┘                 │       │
│  │                                                     │       │
│  └─────────────────────────────────────────────────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 🔧 Redis Implementation

```python
import redis
import time

class RateLimiter:
    def __init__(self, redis_client, limit, window_seconds):
        self.redis = redis_client
        self.limit = limit
        self.window = window_seconds
    
    def is_allowed(self, user_id):
        key = f"rate:{user_id}"
        current_time = int(time.time())
        window_start = current_time - self.window
        
        # Use Redis pipeline for atomic operations
        pipe = self.redis.pipeline()
        
        # Remove old entries
        pipe.zremrangebyscore(key, 0, window_start)
        
        # Count current window
        pipe.zcard(key)
        
        # Add current request
        pipe.zadd(key, {str(current_time): current_time})
        
        # Set expiry
        pipe.expire(key, self.window)
        
        results = pipe.execute()
        request_count = results[1]
        
        if request_count < self.limit:
            return True
        else:
            return False

# Usage
limiter = RateLimiter(redis.Redis(), limit=100, window_seconds=60)
if limiter.is_allowed("user_123"):
    # Process request
    pass
else:
    # Return 429 Too Many Requests
    pass
```

## 📡 Response Headers

```
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1706295600
Retry-After: 30

{
  "error": "rate_limit_exceeded",
  "message": "Too many requests. Please try again in 30 seconds."
}
```

## 💡 Design Considerations

```
✓ Different limits per tier (free/premium)
✓ Separate limits per endpoint
✓ Graceful degradation if Redis fails
✓ Local caching to reduce Redis calls
✓ Rate limit by: IP, User, API Key
```

## 📚 পরবর্তী টপিক

[Notification System →](./08-notification-system.md)
