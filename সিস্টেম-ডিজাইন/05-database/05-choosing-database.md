# ডাটাবেস নির্বাচন গাইড

## 🎯 Decision Framework

```
প্রশ্ন করুন:
1. Data structure কেমন? (Structured/Unstructured)
2. Read-heavy না Write-heavy?
3. ACID দরকার?
4. Scale করতে হবে কিভাবে?
5. Query patterns কেমন?
```

## 📊 Use Case অনুযায়ী Database

### ১. User Data, Orders, Transactions
```
✓ PostgreSQL / MySQL

কারণ:
- Structured data
- ACID compliance
- Complex queries (JOINs)
- Relationships important

Example Schema:
users → orders → order_items → products
```

### ২. Session Storage, Cache
```
✓ Redis / Memcached

কারণ:
- In-memory (super fast)
- Key-value structure
- TTL support
- Simple operations

Example:
session:user123 → {user_data}
cache:product:456 → {product_data}
```

### ৩. Chat Messages, Time-Series
```
✓ Cassandra / ScyllaDB

কারণ:
- Write-heavy
- Time-ordered data
- High availability
- Horizontal scaling

Example:
messages_by_chat: (chat_id, timestamp) → message
```

### ৪. Search, Full-Text
```
✓ Elasticsearch / OpenSearch

কারণ:
- Full-text search
- Fuzzy matching
- Aggregations
- Near real-time indexing

Example:
Search products by name, description, tags
```

### ৫. Social Networks, Recommendations
```
✓ Neo4j / Amazon Neptune

কারণ:
- Graph relationships
- Complex traversals
- Friend-of-friend queries

Example:
(User)-[:FOLLOWS]->(User)
(User)-[:LIKES]->(Product)
```

### ৬. Content, Documents
```
✓ MongoDB

কারণ:
- Flexible schema
- Nested documents
- Quick development
- Horizontal scaling

Example:
{
  "user_id": "123",
  "profile": {...},
  "preferences": {...}
}
```

### ৭. File/Object Storage
```
✓ Amazon S3 / MinIO

কারণ:
- Large files
- Images, videos
- Backups
- Static content

Example:
s3://bucket/users/123/avatar.jpg
```

## 🔧 Quick Reference

```
┌─────────────────────┬─────────────────────────────────────┐
│      Use Case       │           Database                  │
├─────────────────────┼─────────────────────────────────────┤
│ General purpose     │ PostgreSQL                          │
│ E-commerce          │ PostgreSQL + Redis                  │
│ Social media        │ Cassandra + Redis + Elasticsearch   │
│ Chat/Messaging      │ Cassandra + Redis                   │
│ Analytics           │ ClickHouse / BigQuery              │
│ IoT/Time-series     │ TimescaleDB / InfluxDB             │
│ Gaming leaderboard  │ Redis (Sorted Sets)                 │
│ Content management  │ MongoDB                             │
│ Search engine       │ Elasticsearch                       │
│ Recommendations     │ Neo4j + Redis                       │
└─────────────────────┴─────────────────────────────────────┘
```

## 💡 Pro Tips

```
১. একাধিক database ব্যবহার করতে পারেন
   - Write: PostgreSQL
   - Cache: Redis
   - Search: Elasticsearch

২. শুরুতে simple রাখুন
   - PostgreSQL দিয়ে শুরু করুন
   - প্রয়োজনে অন্য যোগ করুন

৩. Managed services consider করুন
   - RDS, Aurora, Atlas, etc.
   - Operations কম, reliability বেশি
```

## ✅ Checklist

```
□ Data structure analyzed
□ Read/Write ratio estimated
□ Consistency requirements defined
□ Scale requirements understood
□ Query patterns identified
□ Cost considered
□ Team expertise evaluated
```

---

🎉 ডাটাবেস সেকশন সম্পূর্ণ!

[মেসেজ কিউ →](../06-message-queues/README.md)
