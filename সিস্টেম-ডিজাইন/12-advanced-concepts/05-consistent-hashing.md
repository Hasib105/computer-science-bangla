# Consistent Hashing

## 🎯 Problem Statement

```
সাধারণ Hashing:
server = hash(key) % num_servers

Example:
hash("user123") % 3 = 1 → Server 1

Problem: Server add/remove করলে:
hash("user123") % 4 = 2 → Server 2 (Changed!)

প্রায় সব keys re-distribute হয়!
```

## 📊 Consistent Hashing Solution

```
Hash ring-এ servers ও keys দুটোই place করা।

┌──────────────────────────────────────────────────────────────┐
│                    Hash Ring                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                         0°                                   │
│                         │                                    │
│                    ┌────┴────┐                              │
│                    │ Server A│                              │
│                    └─────────┘                              │
│                  /             \                            │
│                 /               \                           │
│           270° │                 │ 90°                      │
│      ┌─────────┐                 ┌─────────┐               │
│      │ Server D│                 │ Server B│               │
│      └─────────┘                 └─────────┘               │
│                 \               /                           │
│                  \             /                            │
│                    ┌─────────┐                              │
│                    │ Server C│                              │
│                    └────┬────┘                              │
│                         │                                    │
│                        180°                                  │
│                                                              │
│  Key placement: Clockwise to nearest server                 │
│  key1 (45°) → Server B                                      │
│  key2 (100°) → Server C                                     │
│  key3 (200°) → Server D                                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## 🔄 Adding/Removing Servers

```
Server Add করলে:

Before: A, B, C, D (4 servers)
After: A, B, C, D, E (5 servers - E added between A and B)

Only keys between A and E are remapped!
Other keys unchanged.

┌──────────────────────────────────────────┐
│   Before        After                    │
│                                          │
│     A             A                      │
│    / \           / \                     │
│   D   B         D   E ← New              │
│    \ /           \ / \                   │
│     C             C   B                  │
│                                          │
│   Keys affected: Only A→E range         │
│   (Instead of ALL keys)                  │
└──────────────────────────────────────────┘

Average keys moved: K/N
(K = total keys, N = total servers)
```

## 🎲 Virtual Nodes

```
Problem: Uneven distribution with few servers

Solution: Each server gets multiple positions (virtual nodes)

┌──────────────────────────────────────────────────────────────┐
│                 Virtual Nodes                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Physical Servers: A, B, C                                   │
│                                                              │
│  Virtual Nodes:                                              │
│  A → A1, A2, A3 (at positions 30°, 150°, 270°)             │
│  B → B1, B2, B3 (at positions 60°, 180°, 300°)             │
│  C → C1, C2, C3 (at positions 90°, 210°, 330°)             │
│                                                              │
│          0°                                                  │
│          │                                                   │
│     A1 ──┼── B1                                             │
│    /     │     \                                            │
│   C3     │      C1                                          │
│   │      │       │                                          │
│   B3 ────┼────── A2                                         │
│    \     │      /                                           │
│     C2 ──┼── B2                                             │
│          │                                                   │
│         180°                                                 │
│                                                              │
│  Better distribution!                                        │
│  More virtual nodes = Better balance                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## 🔧 Implementation

```python
import hashlib
from bisect import bisect_left

class ConsistentHash:
    def __init__(self, nodes=None, virtual_nodes=100):
        self.virtual_nodes = virtual_nodes
        self.ring = {}  # hash -> node
        self.sorted_keys = []
        
        if nodes:
            for node in nodes:
                self.add_node(node)
    
    def _hash(self, key):
        return int(hashlib.md5(key.encode()).hexdigest(), 16)
    
    def add_node(self, node):
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}:{i}"
            hash_val = self._hash(virtual_key)
            self.ring[hash_val] = node
            self.sorted_keys.append(hash_val)
        self.sorted_keys.sort()
    
    def remove_node(self, node):
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}:{i}"
            hash_val = self._hash(virtual_key)
            del self.ring[hash_val]
            self.sorted_keys.remove(hash_val)
    
    def get_node(self, key):
        if not self.ring:
            return None
        
        hash_val = self._hash(key)
        idx = bisect_left(self.sorted_keys, hash_val)
        
        # Wrap around
        if idx == len(self.sorted_keys):
            idx = 0
        
        return self.ring[self.sorted_keys[idx]]

# Usage
ch = ConsistentHash(['server1', 'server2', 'server3'])
print(ch.get_node('user:123'))  # server2
print(ch.get_node('user:456'))  # server1

# Add new server
ch.add_node('server4')
print(ch.get_node('user:123'))  # might still be server2
```

## 💡 Real-World Usage

```
Systems using Consistent Hashing:
├── Amazon DynamoDB
├── Apache Cassandra
├── Akamai CDN
├── Discord
├── Memcached
└── Redis Cluster

Use Cases:
├── Distributed caching
├── Load balancing
├── Database sharding
├── CDN routing
└── Session affinity
```

## ✅ Benefits

```
✓ Minimal key redistribution on changes
✓ Horizontal scaling
✓ Fault tolerance
✓ Even distribution (with virtual nodes)
```

## 📚 পরবর্তী টপিক

[Bloom Filters →](./06-bloom-filters.md)
