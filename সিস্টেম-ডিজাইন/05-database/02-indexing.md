# ডাটাবেস ইনডেক্সিং (Database Indexing)

## 🎯 ইনডেক্স কি?

**ইনডেক্স** হলো ডাটাবেসের একটি ডাটা স্ট্রাকচার যা কুয়েরি পারফরম্যান্স বাড়ায় - অনেকটা বইয়ের সূচিপত্রের মতো।

```
ইনডেক্স ছাড়া (Full Table Scan):
┌──────────────────────────────────────┐
│  Row 1 → Row 2 → Row 3 → ... → Row N │
│  (সব row চেক করতে হয়)               │
│  Time: O(n)                          │
└──────────────────────────────────────┘

ইনডেক্স সহ:
┌──────────────────────────────────────┐
│  Index Tree                          │
│       ┌───┐                          │
│       │ M │                          │
│       └─┬─┘                          │
│    ┌────┴────┐                       │
│  ┌─┴─┐    ┌──┴──┐                   │
│  │ G │    │  S  │                   │
│  └───┘    └─────┘                   │
│  Time: O(log n)                      │
└──────────────────────────────────────┘
```

## 📊 কিভাবে কাজ করে?

### B-Tree Index (সবচেয়ে সাধারণ)
```
                    ┌─────────────┐
                    │  [M]        │
                    │   └─────────│
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
    ┌─────────┐      ┌─────────┐      ┌─────────┐
    │ [D, G]  │      │ [N, P]  │      │ [T, W]  │
    └────┬────┘      └────┬────┘      └────┬────┘
         │                │                │
    ┌────┼────┐      ┌────┼────┐      ┌────┼────┐
    ↓    ↓    ↓      ↓    ↓    ↓      ↓    ↓    ↓
   A-C  E-F  H-L    N    O-P  Q-S    T    U-V  X-Z
  [Leaf Nodes with pointers to actual data]

Search for "Karim":
1. Root: K < M → Go left
2. [D, G]: K > G → Go right
3. [H-L]: Found! → Get row pointer
```

### Hash Index
```
hash("ahmed@mail.com") = 42
hash("rahim@mail.com") = 17

Hash Table:
┌────┬─────────────────┐
│ 17 │ → Row pointer   │
│ 42 │ → Row pointer   │
│... │                 │
└────┴─────────────────┘

সুবিধা: O(1) lookup
অসুবিধা: Range queries সাপোর্ট করে না
```

## 🔧 ইনডেক্স তৈরি

### PostgreSQL
```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (multiple columns)
CREATE INDEX idx_users_city_age ON users(city, age);

-- Unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Partial index
CREATE INDEX idx_active_users ON users(email) 
    WHERE status = 'active';

-- Index দেখা
\d users
```

### MySQL
```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Show indexes
SHOW INDEX FROM users;

-- Explain query
EXPLAIN SELECT * FROM users WHERE email = 'test@mail.com';
```

## 📊 ইনডেক্স প্রকারভেদ

### ১. Primary Index
```sql
-- Automatically created with PRIMARY KEY
CREATE TABLE users (
    id SERIAL PRIMARY KEY,  -- Primary index
    email VARCHAR(255)
);
```

### ২. Secondary Index
```sql
-- Additional indexes
CREATE INDEX idx_email ON users(email);
```

### ৩. Composite Index
```sql
-- Multiple columns
CREATE INDEX idx_city_age ON users(city, age);

-- Query optimization:
-- ✓ WHERE city = 'ঢাকা'
-- ✓ WHERE city = 'ঢাকা' AND age = 25
-- ✗ WHERE age = 25 (leftmost column missing)
```

### ৪. Covering Index
```sql
-- Index contains all needed columns
CREATE INDEX idx_covering ON orders(user_id, order_date, total);

-- This query uses only the index (no table access)
SELECT order_date, total 
FROM orders 
WHERE user_id = 123;
```

### ৫. Full-Text Index
```sql
-- Text search
CREATE INDEX idx_search ON articles USING GIN(to_tsvector('english', content));

SELECT * FROM articles 
WHERE to_tsvector('english', content) @@ to_tsquery('database');
```

## ⚖️ ইনডেক্স Trade-offs

```
সুবিধা:
✓ Read performance বাড়ে (SELECT দ্রুত)
✓ Query plan optimize হয়
✓ Sorting দ্রুত হয়

অসুবিধা:
✗ Write performance কমে (INSERT, UPDATE, DELETE স্লো)
✗ Extra storage লাগে
✗ Index maintenance overhead
```

### Trade-off উদাহরণ
```
Users Table: ১ মিলিয়ন rows

Without Index:
├── SELECT: ৫০০ ms (Full scan)
├── INSERT: ১ ms
└── Storage: ৫০০ MB

With 3 Indexes:
├── SELECT: ১ ms (Index lookup)
├── INSERT: ৫ ms (৩টি index update)
└── Storage: ৭৫০ MB (+২৫০ MB)
```

## 🔍 Query Execution Plan

### EXPLAIN ব্যবহার
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@mail.com';

-- Output:
-- Index Scan using idx_users_email on users (cost=0.42..8.44 rows=1)
--   Index Cond: (email = 'test@mail.com'::text)
--   Actual time: 0.025..0.026 rows=1

-- দেখার বিষয়:
-- - Seq Scan = Bad (Full table scan)
-- - Index Scan = Good
-- - Actual time = কত সময় লাগছে
```

## 💡 ইনডেক্সিং Best Practices

### কখন ইনডেক্স করবেন
```
✓ WHERE clause এ ঘন ঘন ব্যবহৃত columns
✓ JOIN conditions এ ব্যবহৃত columns
✓ ORDER BY columns
✓ Foreign keys
✓ High cardinality columns (অনেক unique values)
```

### কখন ইনডেক্স করবেন না
```
✗ ছোট টেবিল (< ১০০০ rows)
✗ Low cardinality columns (যেমন: gender, status)
✗ যে columns বারবার update হয়
✗ BLOB/TEXT columns (সাধারণত)
```

### Index Maintenance
```sql
-- Unused indexes খুঁজে বের করা (PostgreSQL)
SELECT 
    indexrelname,
    idx_scan,
    idx_tup_read
FROM pg_stat_user_indexes
WHERE idx_scan = 0;

-- Index rebuild
REINDEX INDEX idx_users_email;

-- Index statistics update
ANALYZE users;
```

## 📚 পরবর্তী টপিক

[নরমালাইজেশন ও ডেনরমালাইজেশন →](./03-normalization.md)
