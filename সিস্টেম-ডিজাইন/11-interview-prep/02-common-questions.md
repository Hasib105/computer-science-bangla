# সাধারণ প্রশ্ন ও উত্তর

## 🎯 জনপ্রিয় সিস্টেম ডিজাইন প্রশ্ন

### ১. URL Shortener (bit.ly)
```
Key Points:
- Base62 encoding for short codes
- Database: URL mappings
- Cache: Hot URLs
- 301 Redirect

Challenges:
- Collision handling
- Analytics tracking
- Expiration
```

### ২. Twitter/Social Feed
```
Key Points:
- Push vs Pull model
- Celebrity problem → Hybrid
- Redis for timeline cache
- Kafka for fanout

Challenges:
- Timeline generation at scale
- Real-time updates
```

### ৩. WhatsApp/Chat System
```
Key Points:
- WebSocket connections
- Message queue
- E2E encryption
- Presence service

Challenges:
- Offline message delivery
- Group chat fanout
- Connection management
```

### ৪. YouTube/Video Streaming
```
Key Points:
- Video encoding pipeline
- CDN for delivery
- Adaptive bitrate streaming
- Recommendation system

Challenges:
- Video processing at scale
- Storage optimization
- Global delivery
```

### ৫. Uber/Ride Sharing
```
Key Points:
- Location tracking
- Matching algorithm
- Real-time updates
- Surge pricing

Challenges:
- Geo-indexing (Quadtree/Geohash)
- ETA calculation
- Driver matching
```

## 📊 Quick Reference Table

```
┌─────────────────┬──────────────────────────────────────────┐
│    System       │           Key Components                 │
├─────────────────┼──────────────────────────────────────────┤
│ URL Shortener   │ Base62, Redis cache, 301 redirect       │
│ Twitter         │ Fanout, Timeline cache, Hybrid model    │
│ WhatsApp        │ WebSocket, Message queue, E2E encryption│
│ YouTube         │ CDN, Transcoding, HLS/DASH              │
│ Uber            │ Geohash, Real-time matching, Kafka      │
│ Instagram       │ Object storage, CDN, News feed          │
│ Dropbox         │ Block storage, Sync, Chunking           │
│ Rate Limiter    │ Token bucket, Redis, Sliding window     │
│ Search Engine   │ Inverted index, Ranking, Crawling       │
│ Notification    │ Priority queue, Multiple channels       │
└─────────────────┴──────────────────────────────────────────┘
```

## 🔧 প্রতিটি সিস্টেমের জন্য মনে রাখুন

### Database Selection
```
SQL:
- Users, Orders, Transactions
- ACID needed

NoSQL:
- Chat messages (Cassandra)
- User sessions (Redis)
- Search data (Elasticsearch)
- Social graphs (Neo4j)
```

### Caching Pattern
```
Read-heavy: Cache-Aside
Write-heavy: Write-Through/Write-Behind
Session: Redis
CDN: Static content
```

### Scaling Strategy
```
Stateless services → Horizontal scaling
Database → Read replicas + Sharding
Hot data → Caching
Static content → CDN
Async tasks → Message queue
```

## 💬 সাধারণ Follow-up প্রশ্ন

### "How would you handle failures?"
```
- Retry with exponential backoff
- Circuit breaker pattern
- Fallback mechanisms
- Graceful degradation
- Monitoring & alerting
```

### "How would you ensure data consistency?"
```
- Database transactions (ACID)
- Eventual consistency with retries
- Saga pattern for distributed transactions
- Idempotent operations
```

### "How would you handle 10x traffic?"
```
- Auto-scaling
- More cache layers
- Database sharding
- Rate limiting
- Queue-based load leveling
```

### "What are the trade-offs?"
```
Always think:
- Consistency vs Availability
- Latency vs Throughput
- Simplicity vs Scalability
- Cost vs Performance
```

## 📚 পরবর্তী টপিক

[চেকলিস্ট →](./03-checklist.md)
