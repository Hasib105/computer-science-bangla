# Distributed Systems Deep Dive

## 🎯 Distributed System কি?

**Distributed System** হলো এমন একটি সিস্টেম যেখানে একাধিক কম্পিউটার একসাথে কাজ করে একটি unified system হিসেবে।

```
Single Machine:
┌────────────────────────────────┐
│           Server               │
│  ┌─────┐ ┌─────┐ ┌─────┐      │
│  │ CPU │ │ RAM │ │ Disk│      │
│  └─────┘ └─────┘ └─────┘      │
└────────────────────────────────┘

Distributed System:
┌─────────┐     ┌─────────┐     ┌─────────┐
│ Node 1  │────│ Node 2  │────│ Node 3  │
└─────────┘     └─────────┘     └─────────┘
     │               │               │
     └───────────────┴───────────────┘
              Network
```

## 📊 Fallacies of Distributed Computing

```
৮টি ভুল ধারণা যা প্রোগ্রামাররা করে:

1. The network is reliable
   → Packets drop, connections fail

2. Latency is zero
   → Network calls take time (ms to seconds)

3. Bandwidth is infinite
   → Data transfer has limits

4. The network is secure
   → Man-in-the-middle, eavesdropping

5. Topology doesn't change
   → Nodes come and go

6. There is one administrator
   → Multiple teams, policies

7. Transport cost is zero
   → Serialization, network fees

8. The network is homogeneous
   → Different hardware, protocols
```

## 🔄 Distributed System Challenges

### ১. Network Partitions
```
Normal:
Node A ←──────→ Node B

Partition (Split-brain):
Node A    X    Node B
  │             │
  ↓             ↓
"I'm the      "I'm the
 leader"       leader"

উভয়ই মনে করে সে leader!

Solution: Quorum-based decisions
```

### ২. Clock Synchronization
```
Problem:
Node A: 10:00:00.000
Node B: 10:00:00.050  (50ms ahead)
Node C: 09:59:59.990  (10ms behind)

Event ordering কঠিন হয়ে যায়!

Solutions:
├── NTP (Network Time Protocol)
├── Logical Clocks (Lamport)
├── Vector Clocks
└── Hybrid Logical Clocks (HLC)
```

### ৩. Byzantine Failures
```
Node ভুল তথ্য দিতে পারে (malicious/buggy)

Normal Failure:
Node fails → No response

Byzantine Failure:
Node responds with wrong/malicious data

Solution: Byzantine Fault Tolerance (BFT)
Requires: 3f + 1 nodes to tolerate f failures
```

## 🏗️ Distributed System Patterns

### ১. Leader Election
```
কোন node decisions নেবে?

┌─────────────────────────────────────────┐
│           Leader Election               │
├─────────────────────────────────────────┤
│                                         │
│  Initial: All nodes are followers       │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐   │
│  │  F  │  │  F  │  │  F  │  │  F  │   │
│  └─────┘  └─────┘  └─────┘  └─────┘   │
│                                         │
│  Election: One becomes leader           │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐   │
│  │  L  │←─│  F  │──│  F  │──│  F  │   │
│  └─────┘  └─────┘  └─────┘  └─────┘   │
│  Leader   Followers                     │
│                                         │
│  Leader fails: Re-election              │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐   │
│  │  X  │  │  L  │←─│  F  │──│  F  │   │
│  └─────┘  └─────┘  └─────┘  └─────┘   │
│  Dead    New Leader                     │
│                                         │
└─────────────────────────────────────────┘

Algorithms: Bully, Ring, Raft, Paxos
```

### ২. Quorum
```
Majority agreement for decisions।

N = 5 nodes
Quorum = ⌊N/2⌋ + 1 = 3

Write Quorum (W): Minimum nodes to confirm write
Read Quorum (R): Minimum nodes to read from

Strong Consistency: W + R > N

Example:
N=5, W=3, R=3
W + R = 6 > 5 ✓

At least 1 node has latest data in any read.
```

### ৩. Gossip Protocol
```
Nodes randomly exchange information।

┌─────────────────────────────────────────┐
│           Gossip Protocol               │
├─────────────────────────────────────────┤
│                                         │
│  Time T0: Node A has info               │
│  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐   │
│  │ A*│  │ B │  │ C │  │ D │  │ E │   │
│  └───┘  └───┘  └───┘  └───┘  └───┘   │
│                                         │
│  Time T1: A tells B                     │
│  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐   │
│  │ A*│─→│ B*│  │ C │  │ D │  │ E │   │
│  └───┘  └───┘  └───┘  └───┘  └───┘   │
│                                         │
│  Time T2: A→C, B→D                      │
│  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐   │
│  │ A*│─→│ B*│  │ C*│  │ D*│  │ E │   │
│  └───┘  └───┘  └───┘  └───┘  └───┘   │
│                                         │
│  Eventually: All nodes have info        │
│                                         │
└─────────────────────────────────────────┘

Used by: Cassandra, DynamoDB, Consul
```

### ৪. Crdt (Conflict-free Replicated Data Types)
```
Automatic conflict resolution।

G-Counter (Grow-only Counter):
┌─────────────────────────────────────────┐
│  Node A: {A: 5, B: 0, C: 0} = 5        │
│  Node B: {A: 0, B: 3, C: 0} = 3        │
│  Node C: {A: 0, B: 0, C: 2} = 2        │
│                                         │
│  Merge: {A: 5, B: 3, C: 2} = 10        │
│  (Take max of each)                     │
└─────────────────────────────────────────┘

No conflicts possible!
Used by: Riak, Redis CRDT
```

## 📊 CAP Theorem Deep Dive

```
┌─────────────────────────────────────────┐
│              CAP Theorem                │
├─────────────────────────────────────────┤
│                                         │
│           Consistency                   │
│               /\                        │
│              /  \                       │
│             /    \                      │
│            / CP   \                     │
│           /        \                    │
│          /    AP    \                   │
│         /            \                  │
│        /______________\                 │
│   Availability    Partition            │
│                   Tolerance             │
│                                         │
│  CP: MongoDB, HBase, Redis Cluster      │
│  AP: Cassandra, DynamoDB, CouchDB      │
│  CA: Traditional RDBMS (single node)    │
│                                         │
└─────────────────────────────────────────┘

Network partition হলে choose করতে হয়:
- Consistency (reject some requests)
- Availability (serve potentially stale data)
```

## 🔧 PACELC Theorem

```
CAP এর extension:

If Partition (P):
  Choose Availability (A) or Consistency (C)
Else (E):
  Choose Latency (L) or Consistency (C)

Examples:
┌────────────────┬───────────┬───────────┐
│    System      │ Partition │   Else    │
├────────────────┼───────────┼───────────┤
│ DynamoDB       │    PA     │    EL     │
│ Cassandra      │    PA     │    EL     │
│ MongoDB        │    PC     │    EC     │
│ MySQL (single) │    PC     │    EC     │
│ PNUTS (Yahoo)  │    PC     │    EL     │
└────────────────┴───────────┴───────────┘
```

## 💡 Design Considerations

```
Distributed System Design করার সময়:

□ Failure modes identify করুন
□ Idempotent operations design করুন
□ Retry with exponential backoff
□ Circuit breaker pattern
□ Timeout সঠিকভাবে set করুন
□ Monitoring ও observability
□ Data consistency model choose করুন
□ Partition tolerance plan করুন
```

## 📚 পরবর্তী টপিক

[Consensus Algorithms →](./02-consensus.md)
