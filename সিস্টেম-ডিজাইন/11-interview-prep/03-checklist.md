# সিস্টেম ডিজাইন চেকলিস্ট

## ✅ ইন্টারভিউয়ের আগে

### Concepts জানা আছে?
```
□ CAP Theorem
□ Load Balancing (L4 vs L7)
□ Caching strategies
□ Database sharding
□ Replication patterns
□ Message queues
□ Microservices
□ REST vs GraphQL vs gRPC
□ WebSocket
□ CDN
```

### Numbers মনে আছে?
```
□ 1 second = 1,000 milliseconds
□ 1 day = 86,400 seconds
□ 1 million requests/day = ~12 req/s
□ 1 billion requests/day = ~12,000 req/s

Storage:
□ 1 KB = 1,000 bytes
□ 1 MB = 1,000 KB
□ 1 GB = 1,000 MB
□ 1 TB = 1,000 GB
```

### Common Latencies
```
□ Memory access: ~100 ns
□ SSD read: ~100 μs
□ Network round trip: ~500 μs
□ HDD seek: ~10 ms
□ Cross-region: ~100 ms
```

## ✅ ইন্টারভিউয়ের সময়

### Step 1: Requirements
```
□ Functional requirements জিজ্ঞেস করেছি
□ Non-functional requirements জিজ্ঞেস করেছি
□ Scale জেনেছি
□ Scope limit করেছি
```

### Step 2: Estimation
```
□ DAU/MAU ক্যালকুলেট করেছি
□ QPS বের করেছি
□ Storage estimate করেছি
□ Bandwidth estimate করেছি
```

### Step 3: High-Level Design
```
□ Diagram এঁকেছি
□ Main components চিহ্নিত করেছি
□ Data flow দেখিয়েছি
□ API endpoints বলেছি
```

### Step 4: Deep Dive
```
□ Database schema design করেছি
□ Caching strategy বলেছি
□ Critical path explain করেছি
□ Trade-offs discuss করেছি
```

### Step 5: Scaling
```
□ Bottlenecks চিহ্নিত করেছি
□ Solutions propose করেছি
□ Failure scenarios cover করেছি
```

## ✅ সিস্টেম ডিজাইনের সময় যা মনে রাখবেন

### Must-Have Components
```
□ Load Balancer
□ Application Servers (Stateless)
□ Cache Layer
□ Database (Primary + Replicas)
□ CDN (for static content)
□ Message Queue (for async)
```

### Database Considerations
```
□ SQL vs NoSQL decision
□ Primary key selection
□ Indexing strategy
□ Sharding key
□ Replication factor
```

### Caching Considerations
```
□ What to cache
□ Cache invalidation strategy
□ TTL values
□ Cache eviction policy
□ Cache consistency
```

### API Design
```
□ RESTful conventions
□ Pagination
□ Rate limiting
□ Authentication
□ Versioning
```

## 🎯 Final Tips

```
করবেন:
✓ প্রশ্ন করুন
✓ Trade-offs explain করুন
✓ Simple থেকে শুরু করুন
✓ Communicate clearly
✓ Time manage করুন

করবেন না:
✗ Silent থাকবেন না
✗ Over-engineer করবেন না
✗ Details-এ হারিয়ে যাবেন না
✗ Panic করবেন না
```

## 📖 Resources

```
Books:
- Designing Data-Intensive Applications
- System Design Interview (Alex Xu)

Online:
- highscalability.com
- GitHub system-design-primer
- ByteByteGo YouTube
```

---

🎉 **অভিনন্দন!** আপনি সিস্টেম ডিজাইন ডকুমেন্টেশন সম্পূর্ণ করেছেন!

শুভকামনা রইল আপনার ইন্টারভিউয়ের জন্য! 🚀
