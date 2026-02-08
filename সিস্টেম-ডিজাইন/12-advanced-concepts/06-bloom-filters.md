# Bloom Filters

## 🎯 Bloom Filter কি?

**Bloom Filter** হলো একটি space-efficient probabilistic data structure যা membership test করে।

```
Set Membership:
"Is X in the set?"

Regular Set:    Store all elements → Exact answer
Bloom Filter:   Compact representation → Probabilistic answer

Answers:
- "Definitely NOT in set" (100% accurate)
- "Probably in set" (may have false positives)
```

## 📊 How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                    Bloom Filter                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Bit Array (m bits):                                            │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐            │
│  │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │            │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘            │
│    0   1   2   3   4   5   6   7   8   9  10  11               │
│                                                                  │
│  Add "apple" (k=3 hash functions):                              │
│  hash1("apple") = 2                                             │
│  hash2("apple") = 5                                             │
│  hash3("apple") = 9                                             │
│                                                                  │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐            │
│  │ 0 │ 0 │ 1 │ 0 │ 0 │ 1 │ 0 │ 0 │ 0 │ 1 │ 0 │ 0 │            │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘            │
│            ↑           ↑               ↑                        │
│                                                                  │
│  Check "apple": positions 2,5,9 all 1? → YES (probably in set) │
│  Check "banana": positions 1,4,7 → some 0 → NO (definitely not)│
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## 🔄 False Positives

```
False Positive: "Probably yes" কিন্তু actually না

Why happens?
Other elements filled those positions।

┌──────────────────────────────────────────────────────────────────┐
│  After adding "apple" and "orange":                             │
│                                                                  │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐            │
│  │ 0 │ 1 │ 1 │ 0 │ 1 │ 1 │ 0 │ 1 │ 0 │ 1 │ 0 │ 0 │            │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘            │
│                                                                  │
│  Check "grape" (never added):                                   │
│  hash1("grape") = 1 → 1 ✓                                       │
│  hash2("grape") = 4 → 1 ✓                                       │
│  hash3("grape") = 7 → 1 ✓                                       │
│                                                                  │
│  All 1s! → FALSE POSITIVE                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘

False Positive Rate: (1 - e^(-kn/m))^k
m = bit array size
n = number of elements
k = number of hash functions
```

## 🔧 Implementation

```python
import mmh3  # MurmurHash
from bitarray import bitarray

class BloomFilter:
    def __init__(self, size, hash_count):
        self.size = size
        self.hash_count = hash_count
        self.bit_array = bitarray(size)
        self.bit_array.setall(0)
    
    def add(self, item):
        for i in range(self.hash_count):
            index = mmh3.hash(item, i) % self.size
            self.bit_array[index] = 1
    
    def contains(self, item):
        for i in range(self.hash_count):
            index = mmh3.hash(item, i) % self.size
            if not self.bit_array[index]:
                return False  # Definitely not in set
        return True  # Probably in set
    
    def __contains__(self, item):
        return self.contains(item)

# Usage
bf = BloomFilter(size=1000, hash_count=3)
bf.add("apple")
bf.add("banana")

print("apple" in bf)   # True
print("orange" in bf)  # False (or True if false positive)
```

## 💡 Use Cases

```
1. Database Query Optimization
   ┌─────────────────────────────────────────────┐
   │ Query: "SELECT * FROM users WHERE id=123"  │
   │                                             │
   │ Without Bloom Filter:                       │
   │   Check disk for every query               │
   │                                             │
   │ With Bloom Filter:                          │
   │   Check bloom filter first                  │
   │   If "not in set" → Skip disk read         │
   │   If "probably in" → Check disk            │
   └─────────────────────────────────────────────┘

2. Web Crawlers (avoid re-crawling URLs)
3. Spam filters
4. CDN cache lookup
5. Password checking (have I been pwned?)
```

## 📊 Real-World Usage

```
Systems using Bloom Filters:
├── Google Bigtable
├── Apache Cassandra
├── PostgreSQL
├── Redis
├── Akamai CDN
├── Medium (article recommendations)
└── Chrome (malicious URL check)
```

## ⚖️ Trade-offs

```
Pros:
✓ Space efficient (1 byte per element vs 8+ bytes)
✓ O(k) insert and lookup
✓ No false negatives

Cons:
✗ False positives
✗ Cannot delete (unless counting bloom filter)
✗ Cannot list elements
```

## 📚 পরবর্তী টপিক

[Merkle Trees →](./07-merkle-trees.md)
