# Twitter/X ডিজাইন

## 🎯 প্রবলেম স্টেটমেন্ট

```
Twitter-এর মতো একটি সোশ্যাল মিডিয়া প্ল্যাটফর্ম ডিজাইন করুন।
```

## 📊 Requirements

### Functional Requirements
```
১. Tweet করা (text, image, video)
২. Follow/Unfollow users
৩. Home Timeline দেখা
৪. User Timeline দেখা
৫. Search (tweets, users)
৬. Like, Retweet, Reply
৭. Notifications
```

### Non-Functional Requirements
```
১. High availability
২. Low latency timeline (< 200ms)
৩. Eventual consistency acceptable
৪. Read-heavy system
```

### Scale Estimation
```
Assumptions:
- 500M monthly active users
- 200M daily active users
- Average 2 tweets/day/user = 400M tweets/day
- 100 follows on average
- Read:Write = 1000:1

Timeline:
- Each user checks timeline 10 times/day
- 200M × 10 = 2B timeline requests/day
- 2B / 86400 = 23,000 requests/second

Storage:
- Tweet: ~280 chars = 500 bytes
- 400M tweets × 500 bytes = 200 GB/day
- Per year: 73 TB
```

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Twitter Architecture                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│      ┌──────────────────────────────────────────────────┐      │
│      │                 Load Balancer                     │      │
│      └─────────────────────────┬────────────────────────┘      │
│                                │                                │
│         ┌──────────────────────┼──────────────────────┐        │
│         │                      │                      │        │
│    ┌────┴────┐           ┌────┴────┐           ┌────┴────┐   │
│    │  Tweet  │           │Timeline │           │ User    │   │
│    │ Service │           │ Service │           │ Service │   │
│    └────┬────┘           └────┬────┘           └────┬────┘   │
│         │                     │                     │         │
│         │               ┌─────┴─────┐              │         │
│         │               │   Cache   │              │         │
│         │               │  (Redis)  │              │         │
│         │               └─────┬─────┘              │         │
│         │                     │                     │         │
│    ┌────┴────────────────────┴─────────────────────┴────┐   │
│    │                    Message Queue                    │   │
│    │                      (Kafka)                        │   │
│    └────────────────────────┬────────────────────────────┘   │
│                             │                                 │
│    ┌────────────────────────┼────────────────────────────┐   │
│    │                   Data Stores                        │   │
│    │  ┌──────────┐   ┌──────────┐   ┌──────────┐        │   │
│    │  │  Tweet   │   │ Timeline │   │  User    │        │   │
│    │  │   DB     │   │  Cache   │   │   DB     │        │   │
│    │  └──────────┘   └──────────┘   └──────────┘        │   │
│    └─────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 📰 Timeline Generation

### Push Model (Fan-out on Write)
```
User A (100K followers) tweets:
          │
          ▼
    ┌──────────┐
    │  Tweet   │ ── Save Tweet
    │ Service  │
    └────┬─────┘
         │
         ▼
    ┌──────────┐
    │  Fanout  │ ── Push to 100K followers' timelines
    │  Worker  │
    └────┬─────┘
         │
         ├── User B's Timeline Cache
         ├── User C's Timeline Cache
         ├── User D's Timeline Cache
         └── ... (100K updates)

Pros: Fast read (timeline ready)
Cons: Slow write for celebrities, wasted storage
```

### Pull Model (Fan-out on Read)
```
User reads timeline:
          │
          ▼
    ┌──────────┐
    │ Timeline │
    │ Service  │
    └────┬─────┘
         │
         ├── Fetch User A's tweets
         ├── Fetch User B's tweets
         ├── Fetch User C's tweets
         └── ... (100 follows)
         │
         ▼
    Merge & Sort by time
         │
         ▼
    Return Timeline

Pros: Write করা সহজ
Cons: Read slow (many fetches)
```

### Hybrid Model (Twitter's Approach)
```
সাধারণ ইউজার (< 10K followers):
   └── Push Model (Fan-out on Write)

সেলিব্রিটি (> 10K followers):
   └── Pull Model (Fan-out on Read)

Timeline Generation:
   └── Pre-computed timeline + Celebrity tweets merge

┌──────────────────────────────────────────────────────────┐
│                  Hybrid Timeline                         │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Timeline Request                                        │
│        │                                                 │
│        ├── Get pre-computed timeline (normal users)     │
│        │           +                                     │
│        ├── Fetch celebrity tweets on-the-fly           │
│        │                                                 │
│        ▼                                                 │
│    Merge & Return                                        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## 📦 Data Models

```sql
-- Users
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100),
    created_at TIMESTAMP,
    is_celebrity BOOLEAN DEFAULT FALSE
);

-- Tweets
CREATE TABLE tweets (
    tweet_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    media_urls JSON,
    created_at TIMESTAMP,
    like_count INT DEFAULT 0,
    retweet_count INT DEFAULT 0,
    
    INDEX idx_user_created (user_id, created_at DESC)
);

-- Follows
CREATE TABLE follows (
    follower_id BIGINT,
    followee_id BIGINT,
    created_at TIMESTAMP,
    
    PRIMARY KEY (follower_id, followee_id),
    INDEX idx_followee (followee_id)
);

-- Timeline Cache (Redis)
user:123:timeline = [tweet_id_1, tweet_id_2, ...]
```

## ⚡ Key Design Decisions

### Tweet Storage
```
Primary: MySQL/PostgreSQL (write)
Cache: Redis (timeline)
Search: Elasticsearch

Sharding by user_id:
- user_id % num_shards
```

### Media Storage
```
Images/Videos → S3/CDN

Tweet → {
    "id": 123,
    "text": "Hello",
    "media_urls": ["https://cdn.example.com/img1.jpg"]
}
```

### Caching Strategy
```
Timeline Cache (Redis):
- Key: user:{id}:timeline
- Value: List of tweet_ids
- TTL: 24 hours

Tweet Cache:
- Key: tweet:{id}
- Value: Tweet object
- TTL: 1 hour

Hot content → Higher cache priority
```

## ✅ Summary

```
Read Path (Timeline):
User → Load Balancer → Timeline Service → Redis Cache → Return

Write Path (Tweet):
User → Load Balancer → Tweet Service → DB → Kafka → Fanout Workers

Key Components:
1. Tweet Service: CRUD operations
2. Timeline Service: Generate timelines
3. Fanout Service: Push to followers
4. Search Service: Tweet/user search
5. Notification Service: Real-time alerts
```

## 📚 পরবর্তী টপিক

[WhatsApp/Chat System →](./03-chat-system.md)
