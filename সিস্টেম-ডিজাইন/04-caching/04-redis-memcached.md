# Redis ও Memcached

## 🎯 Redis কি?

**Redis** (Remote Dictionary Server) হলো একটি ইন-মেমরি ডাটা স্ট্রাকচার স্টোর যা ক্যাশ, ডাটাবেস, এবং মেসেজ ব্রোকার হিসেবে ব্যবহার হয়।

```
Redis বৈশিষ্ট্য:
├── In-memory (অত্যন্ত দ্রুত)
├── Persistent (ডাটা সেভ করা যায়)
├── Rich Data Structures
├── Pub/Sub
├── Clustering
└── Lua Scripting
```

## 📊 Redis Data Structures

### ১. String
```redis
# Basic SET/GET
SET user:1:name "আহমেদ"
GET user:1:name  → "আহমেদ"

# With TTL
SETEX session:abc 3600 "user_data"  # ১ ঘন্টা

# Atomic increment
SET counter 0
INCR counter  → 1
INCR counter  → 2

# JSON স্টোর
SET user:1 '{"name": "আহমেদ", "age": 25}'
```

### ২. Hash
```redis
# Object-like storage
HSET user:1 name "আহমেদ"
HSET user:1 age 25
HSET user:1 city "ঢাকা"

# Get single field
HGET user:1 name  → "আহমেদ"

# Get all fields
HGETALL user:1
→ {"name": "আহমেদ", "age": "25", "city": "ঢাকা"}

# Multiple set
HMSET user:2 name "রহিম" age 30 city "চট্টগ্রাম"
```

### ৩. List
```redis
# Queue (FIFO)
LPUSH queue:tasks "task1"
LPUSH queue:tasks "task2"
RPOP queue:tasks  → "task1"

# Stack (LIFO)
LPUSH stack:items "item1"
LPUSH stack:items "item2"
LPOP stack:items  → "item2"

# Recent items
LPUSH recent:users "user3"
LTRIM recent:users 0 9  # শুধু শেষ ১০টা রাখো
```

### ৪. Set
```redis
# Unique items
SADD tags:post:1 "tech" "programming" "python"
SMEMBERS tags:post:1  → ["tech", "programming", "python"]

# Check membership
SISMEMBER tags:post:1 "tech"  → 1 (true)

# Set operations
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"
SINTER set1 set2  → ["b", "c"]  # Intersection
SUNION set1 set2  → ["a", "b", "c", "d"]  # Union
```

### ৫. Sorted Set
```redis
# Leaderboard
ZADD leaderboard 100 "player1"
ZADD leaderboard 200 "player2"
ZADD leaderboard 150 "player3"

# Top 3
ZREVRANGE leaderboard 0 2 WITHSCORES
→ [("player2", 200), ("player3", 150), ("player1", 100)]

# Rank
ZRANK leaderboard "player1"  → 0 (সবার নিচে)
```

### ৬. Pub/Sub
```redis
# Subscriber
SUBSCRIBE channel:notifications

# Publisher
PUBLISH channel:notifications "New message!"

# Pattern subscribe
PSUBSCRIBE channel:*
```

## 🔧 Redis Python ব্যবহার

```python
import redis
import json

# Connection
r = redis.Redis(host='localhost', port=6379, db=0)

# String operations
r.set('name', 'আহমেদ')
r.get('name')  # b'আহমেদ'

# With TTL
r.setex('session:123', 3600, 'session_data')

# Hash operations
r.hset('user:1', mapping={
    'name': 'আহমেদ',
    'age': 25
})
r.hgetall('user:1')

# JSON caching
def get_user(user_id):
    cache_key = f'user:{user_id}'
    
    # Check cache
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Get from DB
    user = db.get_user(user_id)
    
    # Cache it
    r.setex(cache_key, 3600, json.dumps(user))
    
    return user
```

## 🎯 Memcached কি?

**Memcached** হলো একটি হাই-পারফরম্যান্স, ডিস্ট্রিবিউটেড মেমরি ক্যাশিং সিস্টেম।

```
Memcached বৈশিষ্ট্য:
├── সিম্পল key-value store
├── Multi-threaded
├── No persistence
├── শুধু String ডাটা টাইপ
└── LRU eviction
```

## 🔧 Memcached Python ব্যবহার

```python
from pymemcache.client import base

client = base.Client(('localhost', 11211))

# Set
client.set('key', 'value')

# Get
client.get('key')  # b'value'

# With expiration
client.set('key', 'value', expire=3600)

# Delete
client.delete('key')

# Increment
client.set('counter', '0')
client.incr('counter', 1)
```

## 🆚 Redis vs Memcached

```
┌─────────────────────┬─────────────────┬─────────────────┐
│       Feature       │      Redis      │   Memcached     │
├─────────────────────┼─────────────────┼─────────────────┤
│ Data Types          │ Rich (6+)       │ String only     │
│ Persistence         │ ✓               │ ✗               │
│ Replication         │ ✓               │ ✗               │
│ Clustering          │ ✓               │ ✓ (client)      │
│ Pub/Sub             │ ✓               │ ✗               │
│ Lua Scripting       │ ✓               │ ✗               │
│ Threading           │ Single-threaded │ Multi-threaded  │
│ Memory Efficiency   │ Lower           │ Higher          │
│ Use Case            │ Complex caching │ Simple caching  │
└─────────────────────┴─────────────────┴─────────────────┘
```

## 🔀 কখন কোনটি ব্যবহার করবেন?

### Redis বেছে নিন
```
✓ Complex data structures দরকার (List, Set, Hash)
✓ Persistence দরকার
✓ Pub/Sub messaging দরকার
✓ Leaderboards, Counting, Rate limiting
✓ Session storage
✓ Real-time analytics
```

### Memcached বেছে নিন
```
✓ সিম্পল key-value ক্যাশ
✓ মাল্টি-থ্রেডেড পারফরম্যান্স চাই
✓ মেমরি efficiency গুরুত্বপূর্ণ
✓ অনেক ছোট ছোট ভ্যালু
✓ লিগ্যাসি সিস্টেমে ইতিমধ্যে আছে
```

## 🏗️ Redis Cluster

```
┌───────────────────────────────────────────────────────────────┐
│                      Redis Cluster                            │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│   │   Master 1  │    │   Master 2  │    │   Master 3  │      │
│   │ Slots 0-5460│    │ Slots 5461- │    │Slots 10923- │      │
│   │             │    │    10922    │    │    16383    │      │
│   └──────┬──────┘    └──────┬──────┘    └──────┬──────┘      │
│          │                  │                  │              │
│   ┌──────┴──────┐    ┌──────┴──────┐    ┌──────┴──────┐      │
│   │  Replica 1  │    │  Replica 2  │    │  Replica 3  │      │
│   └─────────────┘    └─────────────┘    └─────────────┘      │
│                                                               │
│   Hash Slots: 16384 টি স্লট ভাগ করা                           │
│   Key → hash(key) % 16384 → Slot → Master                   │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

## 💡 Redis Best Practices

```
১. সঠিক Data Structure ব্যবহার করুন
   - User object → Hash
   - Queue → List
   - Unique items → Set
   - Rankings → Sorted Set

২. Key Naming Convention
   - object-type:id:field
   - user:123:profile
   - session:abc123

৩. TTL সবসময় সেট করুন
   - মেমরি ফুল হওয়া রোধ

৪. Pipeline ব্যবহার করুন
   - Multiple commands একসাথে

৫. Connection Pooling
   - প্রতি রিকোয়েস্টে নতুন কানেকশন নয়
```

## 📚 পরবর্তী টপিক

[CDN →](./05-cdn.md)
