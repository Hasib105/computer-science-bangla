# ACID ও BASE Properties

## 🎯 ACID Properties

**ACID** হলো traditional relational database-এর গ্যারান্টি।

```
A - Atomicity    (পুরোপুরি হবে অথবা মোটেই হবে না)
C - Consistency  (Valid state থেকে valid state)
I - Isolation    (Concurrent transactions আলাদা)
D - Durability   (Committed data হারাবে না)
```

### Atomicity
```
Bank Transfer Example:

Transaction:
1. Account A থেকে ৫০০০ টাকা কাটা
2. Account B-তে ৫০০০ টাকা যোগ করা

Atomicity গ্যারান্টি:
✓ দুটোই সফল → Commit
✗ একটা ফেইল → Rollback (দুটোই বাতিল)

কখনো এমন হবে না:
✗ A থেকে কেটে গেল কিন্তু B-তে যোগ হলো না
```

### Consistency
```
Rules/Constraints সবসময় মানা হবে।

Example:
- Bank balance কখনো negative হবে না
- Foreign key reference সবসময় valid থাকবে
- Age সবসময় positive হবে

Transaction শুরু: Valid state
Transaction শেষ: Valid state
```

### Isolation
```
Concurrent transactions একে অপরকে দেখে না।

User A: Reads balance = ১০০০
User B: Reads balance = ১০০০
User A: Withdraws ৫০০
User B: Withdraws ৫০০

Without Isolation:
A reads 1000, B reads 1000
A: 1000 - 500 = 500 ✓
B: 1000 - 500 = 500 ✗ (should be 0!)

With Isolation:
Transactions run as if they are sequential.
```

### Durability
```
একবার commit হলে, ডাটা হারাবে না।

Commit → Write to disk → Power failure
         ↓
    Data survives!

Techniques:
- Write-ahead logging (WAL)
- Disk persistence
- Replication
```

## 📊 Isolation Levels

```
┌─────────────────────┬────────────┬───────────────┬───────────────┐
│    Isolation Level  │ Dirty Read │ Non-repeatable│ Phantom Read  │
├─────────────────────┼────────────┼───────────────┼───────────────┤
│ Read Uncommitted    │     ✗      │      ✗        │      ✗        │
│ Read Committed      │     ✓      │      ✗        │      ✗        │
│ Repeatable Read     │     ✓      │      ✓        │      ✗        │
│ Serializable        │     ✓      │      ✓        │      ✓        │
└─────────────────────┴────────────┴───────────────┴───────────────┘

✓ = Problem prevented
✗ = Problem can occur

Higher isolation = More protection but lower performance
```

## 🔄 BASE Properties

**BASE** হলো NoSQL/Distributed systems-এর approach।

```
BA - Basically Available
S  - Soft state
E  - Eventual consistency
```

### Basically Available
```
সিস্টেম সবসময় response দেবে (হয়তো stale data সহ)।

Partition হলেও:
┌─────────┐     X     ┌─────────┐
│ Node A  │───────────│ Node B  │
└─────────┘           └─────────┘

User → Node A → "Here's the data (might be slightly old)"
```

### Soft State
```
Data সময়ের সাথে change হতে পারে, input ছাড়াই।

Example:
- Cache expiration
- Background sync
- Replication lag
```

### Eventual Consistency
```
Eventually, সব nodes একই data দেখাবে।

Write → Node A
         │
         ├── Sync → Node B (1 sec later)
         └── Sync → Node C (2 sec later)

Time T+0: Only A has new data
Time T+1: A and B have new data
Time T+2: All nodes consistent ✓
```

## ⚖️ ACID vs BASE

```
┌────────────────────┬────────────────────────────────────┐
│       ACID         │              BASE                  │
├────────────────────┼────────────────────────────────────┤
│ Strong consistency │ Eventual consistency               │
│ Pessimistic        │ Optimistic                         │
│ Complex, slower    │ Simpler, faster                    │
│ Vertical scaling   │ Horizontal scaling                 │
│ SQL databases      │ NoSQL databases                    │
│ Banking, Finance   │ Social media, Analytics            │
└────────────────────┴────────────────────────────────────┘
```

## 💡 কোনটা কখন?

```
ACID বেছে নিন:
- Financial transactions
- Inventory management
- Data integrity is critical

BASE বেছে নিন:
- Social media feeds
- Real-time analytics
- High scalability needed
- Minor inconsistency acceptable
```

## 📚 পরবর্তী টপিক

[ডাটাবেস নির্বাচন →](./05-choosing-database.md)
