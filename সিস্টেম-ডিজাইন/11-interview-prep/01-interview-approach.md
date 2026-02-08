# ইন্টারভিউ অ্যাপ্রোচ

## 🎯 সিস্টেম ডিজাইন ইন্টারভিউ Framework

```
মোট সময়: ৪৫-৬০ মিনিট

┌────────────────────────────────────────────────────────────┐
│                 Interview Timeline                         │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌───────────────┐                                         │
│  │ Requirements  │ (৫ মিনিট)                               │
│  │ Clarification │                                         │
│  └───────────────┘                                         │
│         │                                                  │
│         ▼                                                  │
│  ┌───────────────┐                                         │
│  │   Capacity    │ (৫ মিনিট)                               │
│  │  Estimation   │                                         │
│  └───────────────┘                                         │
│         │                                                  │
│         ▼                                                  │
│  ┌───────────────┐                                         │
│  │  High-Level   │ (১০ মিনিট)                              │
│  │    Design     │                                         │
│  └───────────────┘                                         │
│         │                                                  │
│         ▼                                                  │
│  ┌───────────────┐                                         │
│  │   Deep Dive   │ (২০ মিনিট)                              │
│  │  Components   │                                         │
│  └───────────────┘                                         │
│         │                                                  │
│         ▼                                                  │
│  ┌───────────────┐                                         │
│  │ Bottlenecks & │ (১০ মিনিট)                              │
│  │   Scaling     │                                         │
│  └───────────────┘                                         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

## ১. Requirements Clarification (৫ মিনিট)

### Functional Requirements জিজ্ঞেস করুন
```
"Design Twitter" প্রশ্নের জন্য:

✓ Users কি tweet করতে পারবে?
✓ Follow/unfollow আছে?
✓ Timeline দেখা যাবে?
✓ Like, retweet, reply আছে?
✓ Search functionality আছে?
✓ Media support (images, videos)?
✓ Notifications?
```

### Non-Functional Requirements
```
✓ কতজন user?
✓ Read-heavy না write-heavy?
✓ Latency requirement কত?
✓ Availability vs Consistency?
✓ Data retention কতদিন?
```

### Scope নির্ধারণ
```
সময় সীমিত, তাই বলুন:

"Let's focus on core features:
 - Tweet creation
 - Timeline generation
 - Follow system
 
 We can discuss search and notifications 
 if time permits."
```

## ২. Capacity Estimation (৫ মিনিট)

### Template
```
Users:
- Total users: X
- Daily active users (DAU): Y
- Actions per user per day: Z

Traffic:
- Read requests/second = DAU × reads/user / 86400
- Write requests/second = DAU × writes/user / 86400
- Peak = Average × 3

Storage:
- Data per item × items per day × retention days
- Add 20% buffer

Bandwidth:
- Request size × requests/second
```

### উদাহরণ: Twitter
```
Users:
- 500M total users
- 200M DAU
- 10 timeline views/day
- 2 tweets/day

Traffic:
- Read: 200M × 10 / 86400 ≈ 23,000 req/s
- Write: 200M × 2 / 86400 ≈ 4,600 req/s
- Peak: 23,000 × 3 ≈ 70,000 req/s

Storage:
- Tweet: 500 bytes
- 400M tweets/day × 500B = 200 GB/day
- 5 years = 365 TB
```

## ৩. High-Level Design (১০ মিনিট)

### ডায়াগ্রাম আঁকুন
```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│      ┌──────────┐                                       │
│      │  Client  │                                       │
│      └────┬─────┘                                       │
│           │                                             │
│      ┌────┴─────┐                                       │
│      │   Load   │                                       │
│      │ Balancer │                                       │
│      └────┬─────┘                                       │
│           │                                             │
│    ┌──────┴──────┐                                      │
│    │             │                                      │
│  ┌─┴───┐     ┌───┴─┐                                   │
│  │ App │     │ App │     ← Stateless servers           │
│  └─┬───┘     └───┬─┘                                   │
│    │             │                                      │
│    └──────┬──────┘                                      │
│           │                                             │
│      ┌────┴─────┐                                       │
│      │  Cache   │      ← Redis                         │
│      └────┬─────┘                                       │
│           │                                             │
│      ┌────┴─────┐                                       │
│      │ Database │      ← Primary + Replicas            │
│      └──────────┘                                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Components ব্যাখ্যা করুন
```
১. Load Balancer: Traffic distribution
২. App Servers: Business logic
৩. Cache: Reduce DB load
৪. Database: Persistent storage
৫. Message Queue: Async processing
```

## ৪. Deep Dive (২০ মিনিট)

### Core Component বিস্তারিত
```
ইন্টারভিউয়ার যেটা গুরুত্বপূর্ণ মনে করেন সেটা Deep dive করুন।

Timeline Generation:
- Push vs Pull model
- Celebrity problem
- Hybrid approach

Database Schema:
- Table design
- Indexing strategy
- Sharding key

Caching:
- What to cache
- Cache invalidation
- TTL strategy
```

### API Design
```
REST endpoints:

POST /tweets
GET /tweets/{id}
GET /users/{id}/timeline
POST /users/{id}/follow
```

### Database Schema
```sql
-- Example tables
CREATE TABLE tweets (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    created_at TIMESTAMP,
    INDEX (user_id, created_at)
);
```

## ৫. Bottlenecks & Scaling (১০ মিনিট)

### Single Point of Failure চিহ্নিত করুন
```
"এখন দেখি কোথায় সমস্যা হতে পারে..."

১. Database bottleneck
   → Read replicas, Sharding

২. Application server overload
   → Horizontal scaling, Auto-scaling

৩. Cache failure
   → Redis cluster, Fallback to DB

৪. Load balancer failure
   → Multiple LBs, DNS failover
```

### Scaling Solutions
```
Read-heavy:
- Add cache layers
- Read replicas
- CDN for static content

Write-heavy:
- Write to queue first
- Async processing
- Database sharding

Global users:
- Multi-region deployment
- Geographic load balancing
- Data replication
```

## 💡 গুরুত্বপূর্ণ Tips

```
DO:
✓ প্রশ্ন করুন, অনুমান করবেন না
✓ Trade-offs ব্যাখ্যা করুন
✓ Big picture আগে, details পরে
✓ ইন্টারভিউয়ারের feedback শুনুন
✓ Whiteboard/paper ব্যবহার করুন

DON'T:
✗ সরাসরি coding শুরু করবেন না
✗ Silent থাকবেন না
✗ Over-engineer করবেন না
✗ একটা solution-এ আটকে থাকবেন না
✗ Time management ভুলবেন না
```

## 🗣️ Communication Tips

```
প্রতিটি decision এর জন্য বলুন:

"I'm choosing X because..."
"The trade-off here is..."
"If we had more time, we could..."
"This might become a bottleneck at scale..."
```

## 📚 পরবর্তী টপিক

[সাধারণ প্রশ্ন ও উত্তর →](./02-common-questions.md)
