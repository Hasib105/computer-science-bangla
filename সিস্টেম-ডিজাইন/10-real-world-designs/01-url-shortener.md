# URL Shortener ডিজাইন (bit.ly)

## 🎯 প্রবলেম স্টেটমেন্ট

```
Long URL কে Short URL এ রূপান্তর করা।

Input:  https://www.example.com/very/long/url/path?param=value
Output: https://short.url/abc123
```

## 📊 Requirements

### Functional Requirements
```
১. URL Shortening: লং URL → শর্ট URL
২. URL Redirection: শর্ট URL → লং URL-এ রিডাইরেক্ট
৩. Custom alias (optional)
৪. Link expiration
৫. Analytics
```

### Non-Functional Requirements
```
১. High availability
২. Low latency redirection (< 100ms)
৩. 301 redirect (permanent)
৪. Shortened URLs should not be predictable
```

### Capacity Estimation
```
Assumptions:
- 100M URLs created per month
- Read:Write = 100:1
- URL retention: 5 years

Calculations:
- Write: 100M / month = 40 URL/second
- Read: 40 × 100 = 4000 redirects/second

Storage:
- 100M × 12 months × 5 years = 6B URLs
- Each URL: ~500 bytes
- Total: 6B × 500 = 3 TB
```

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    URL Shortener Architecture                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                      Load Balancer                        │  │
│  └─────────────────────────┬────────────────────────────────┘  │
│                            │                                    │
│           ┌────────────────┼────────────────┐                  │
│           ↓                ↓                ↓                  │
│     ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│     │ App      │    │ App      │    │ App      │              │
│     │ Server 1 │    │ Server 2 │    │ Server 3 │              │
│     └────┬─────┘    └────┬─────┘    └────┬─────┘              │
│          │               │               │                     │
│          └───────────────┼───────────────┘                     │
│                          ↓                                     │
│     ┌────────────────────────────────────────────────────┐    │
│     │                    Cache (Redis)                    │    │
│     │              (Hot URLs for fast redirect)           │    │
│     └────────────────────────┬───────────────────────────┘    │
│                              │                                 │
│                              ↓                                 │
│     ┌────────────────────────────────────────────────────┐    │
│     │               Database (Primary)                    │    │
│     │              (URL mappings store)                   │    │
│     └────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 🔑 Short URL Generation

### Approach 1: Base62 Encoding
```
Characters: a-z, A-Z, 0-9 (62 chars)

6 characters = 62^6 = 56.8 billion combinations
7 characters = 62^7 = 3.5 trillion combinations

Example:
ID: 12345 → Base62 → "3d7"
ID: 1000000 → Base62 → "4c92"

Code:
def to_base62(num):
    chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    result = ""
    while num > 0:
        result = chars[num % 62] + result
        num //= 62
    return result
```

### Approach 2: MD5/SHA Hash
```
Input: "https://example.com/long/url"
         ↓
MD5: "e99a18c428cb38d5f260853678922e03"
         ↓
First 7 chars: "e99a18c"

সমস্যা: Collision হতে পারে
সমাধান: Collision check + retry
```

### Approach 3: Counter + Distributed ID
```
┌──────────────┐
│ ID Generator │ → 1, 2, 3, 4, 5...
│  (Redis/ZK)  │
└──────────────┘
       │
       ▼
   Base62 Encode
       │
       ▼
   "0", "1", "2"...
```

## 📦 Database Schema

```sql
-- URLs Table
CREATE TABLE urls (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    original_url TEXT NOT NULL,
    user_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    click_count INT DEFAULT 0,
    
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id)
);

-- Analytics Table
CREATE TABLE clicks (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    url_id BIGINT,
    clicked_at TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    referrer TEXT,
    country VARCHAR(50),
    
    FOREIGN KEY (url_id) REFERENCES urls(id)
);
```

## 🔄 API Design

### Create Short URL
```http
POST /api/v1/shorten
Content-Type: application/json

{
    "url": "https://example.com/very/long/url",
    "custom_alias": "my-link",  // optional
    "expiry_days": 30           // optional
}

Response:
{
    "short_url": "https://short.url/abc123",
    "original_url": "https://example.com/very/long/url",
    "expires_at": "2024-02-25T00:00:00Z"
}
```

### Redirect
```http
GET /abc123

Response: 301 Redirect
Location: https://example.com/very/long/url
```

## ⚡ Optimization

### Caching
```
┌─────────────────────────────────────────────┐
│              Caching Strategy               │
├─────────────────────────────────────────────┤
│                                             │
│  Request: /abc123                           │
│      │                                      │
│      ▼                                      │
│  ┌─────────┐                                │
│  │  Cache  │ → Hit? → Return URL            │
│  │ (Redis) │                                │
│  └────┬────┘                                │
│       │ Miss                                │
│       ▼                                     │
│  ┌──────────┐                               │
│  │ Database │ → Fetch URL                   │
│  └────┬─────┘                               │
│       │                                     │
│       ▼                                     │
│  Store in Cache (with TTL)                  │
│       │                                     │
│       ▼                                     │
│  Return URL                                 │
│                                             │
└─────────────────────────────────────────────┘

Popular URLs: Higher TTL
Less popular: Lower TTL (or LRU eviction)
```

### Database Sharding
```
Shard by short_code first character:

a-j → Shard 1
k-t → Shard 2
u-z, 0-9 → Shard 3

Or consistent hashing:
hash(short_code) % num_shards
```

## ✅ System Design Checklist

```
□ Short code generation strategy
□ Collision handling
□ Cache for hot URLs
□ Database indexing
□ 301 vs 302 redirect choice
□ Rate limiting
□ Analytics tracking
□ URL validation
□ Expiration handling
□ Custom alias support
```

## 📚 পরবর্তী টপিক

[Twitter/X ডিজাইন →](./02-twitter-design.md)
