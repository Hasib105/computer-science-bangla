# Page Replacement Algorithms (পেজ রিপ্লেসমেন্ট অ্যালগরিদম)

## 📌 কখন Page Replacement দরকার?

যখন page fault হয় কিন্তু কোনো free frame নেই।

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Page fault occurs                                         │
│         │                                                   │
│         ▼                                                   │
│   Any free frame?                                           │
│    /          \                                             │
│   YES          NO                                           │
│    │           │                                            │
│    ▼           ▼                                            │
│   Use it     Select a VICTIM page                          │
│              │                                              │
│              ▼                                              │
│              Is victim DIRTY?                               │
│              (modified bit = 1?)                            │
│              /              \                               │
│            YES               NO                             │
│             │                 │                             │
│             ▼                 │                             │
│         Write to disk        │                              │
│             │                 │                             │
│             └────────┬────────┘                            │
│                      ▼                                      │
│              Load new page                                  │
│              Update page table                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Reference String

Analysis এর জন্য page references এর sequence

```
Example Reference String:
7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1

Memory Frames: 3

আমরা দেখব কোন algorithm এ কত page fault হয়
```

---

## 1️⃣ FIFO (First In First Out)

```
যে page আগে এসেছে সে আগে out হবে

Reference: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1

Frames: 3

Step  Ref  Frame1 Frame2 Frame3  Page Fault?
──────────────────────────────────────────────
1     7     7      -      -       ✓
2     0     7      0      -       ✓
3     1     7      0      1       ✓
4     2     2      0      1       ✓ (7 out, oldest)
5     0     2      0      1       ✗ (0 already in)
6     3     2      3      1       ✓ (0 out)
7     0     2      3      0       ✓ (1 out)
8     4     4      3      0       ✓ (2 out)
9     2     4      2      0       ✓ (3 out)
10    3     4      2      3       ✓ (0 out)
11    0     0      2      3       ✓ (4 out)
12    3     0      2      3       ✗
13    2     0      2      3       ✗
14    1     1      2      3       ✓ (0 out)
15    2     1      2      3       ✗
16    0     1      0      3       ✓ (2 out)
17    1     1      0      3       ✗
18    7     7      0      3       ✓ (1 out)
19    0     7      0      3       ✗
20    1     7      0      1       ✓ (3 out)

Total Page Faults: 15
```

### Belady's Anomaly

```
FIFO তে more frames = more page faults হতে পারে!

Reference: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

3 Frames: 9 page faults
4 Frames: 10 page faults (বেশি!)

এটাই Belady's Anomaly!
```

---

## 2️⃣ Optimal Algorithm (OPT)

```
ভবিষ্যতে সবচেয়ে বেশি সময় পর ব্যবহার হবে এমন page out করো

Reference: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1

Step  Ref  Frame1 Frame2 Frame3  Page Fault?  Reason
─────────────────────────────────────────────────────────────
1     7     7      -      -       ✓
2     0     7      0      -       ✓
3     1     7      0      1       ✓
4     2     2      0      1       ✓  7→next use: 18, furthest
5     0     2      0      1       ✗
6     3     2      0      3       ✓  1→next use: 14, furthest
7     0     2      0      3       ✗
8     4     2      4      3       ✓  0→next use: 11, furthest
9     2     2      4      3       ✗
10    3     2      4      3       ✗
11    0     2      0      3       ✓  4→never used again
12    3     2      0      3       ✗
13    2     2      0      3       ✗
14    1     2      0      1       ✓  3→next use: never
15    2     2      0      1       ✗
16    0     2      0      1       ✗
17    1     2      0      1       ✗
18    7     7      0      1       ✓  2→never used
19    0     7      0      1       ✗
20    1     7      0      1       ✗

Total Page Faults: 9 (Minimum possible!)
```

**সমস্যা:** ভবিষ্যত জানা impossible! এটা শুধু benchmark হিসেবে ব্যবহার হয়।

---

## 3️⃣ LRU (Least Recently Used)

```
সবচেয়ে বেশিক্ষণ ব্যবহার হয়নি এমন page out করো

Reference: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1

Step  Ref  Frame1 Frame2 Frame3  Page Fault?  LRU order
─────────────────────────────────────────────────────────────
1     7     7      -      -       ✓          7
2     0     7      0      -       ✓          7,0
3     1     7      0      1       ✓          7,0,1
4     2     2      0      1       ✓          0,1,2 (7 LRU)
5     0     2      0      1       ✗          1,2,0
6     3     2      3      0       ✓          2,0,3 (1 LRU)
7     0     2      3      0       ✗          2,3,0
8     4     4      3      0       ✓          3,0,4 (2 LRU)
9     2     4      2      0       ✓          0,4,2 (3 LRU)
10    3     4      2      3       ✓          4,2,3 (0 LRU)
11    0     0      2      3       ✓          2,3,0 (4 LRU)
12    3     0      2      3       ✗          2,0,3
13    2     0      2      3       ✗          0,3,2
14    1     1      2      3       ✓          3,2,1 (0 LRU)
15    2     1      2      3       ✗          3,1,2
16    0     0      2      3       ✓          1,2,0 (3 LRU)
17    1     1      2      0       ✓          2,0,1 (Why?)
18    7     7      2      1       ✓          2,1,7 (0 LRU)
19    0     7      0      1       ✓          1,7,0 (2 LRU)
20    1     7      0      1       ✗          7,0,1

Total Page Faults: 12
```

---

## 🔧 LRU Implementation

### Counter Implementation

```c
struct page_entry {
    int frame;
    long long counter;  // Last access time
};

// On every memory reference
page_table[page].counter = ++logical_clock;

// On page fault, find min counter
victim = find_min_counter(page_table);
```

### Stack Implementation

```
Every reference এ page কে stack এর top এ রাখো

Access sequence: 4, 7, 0, 7, 1, 0, 1, 2, 1, 2, 7, 1, 2

Stack (top → bottom):

After 4:  [4]
After 7:  [7,4]
After 0:  [0,7,4]
After 7:  [7,0,4]     ← 7 moves to top
After 1:  [1,7,0,4]
After 0:  [0,1,7,4]   ← 0 moves to top
After 1:  [1,0,7,4]
After 2:  [2,1,0,7,4]
After 1:  [1,2,0,7,4]
After 2:  [2,1,0,7,4]
After 7:  [7,2,1,0,4]
...

LRU page is at BOTTOM
```

---

## 4️⃣ LRU Approximation Algorithms

### Second Chance (Clock)

```
┌─────────────────────────────────────────────────────────────┐
│                     Clock Algorithm                         │
│                                                             │
│         ┌────► Page 0 (ref=0) → VICTIM!                    │
│         │      Page 1 (ref=1) → ref=0, skip                │
│    ┌────┴────┐                                             │
│    │ Clock   │ Page 2 (ref=0) → Check                      │
│    │ Pointer │ Page 3 (ref=1) → ref=0, skip                │
│    └─────────┘ Page 4 (ref=0) → Check                      │
│                ...                                          │
│                                                             │
│   Reference Bit:                                            │
│   • Set to 1 when page is accessed                         │
│   • Clock hand clears to 0 as it passes                    │
│   • Replace first page with ref=0                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Enhanced Second Chance

```
Two bits: (reference, modify)

Priority for replacement:
1. (0,0) - not recently used, not modified → BEST victim
2. (0,1) - not recently used, modified
3. (1,0) - recently used, not modified
4. (1,1) - recently used, modified → WORST victim

Modified page → write to disk before replacing
```

---

## 📊 Algorithm Comparison

| Algorithm | Page Faults | Complexity | Notes |
|-----------|-------------|------------|-------|
| FIFO | 15 | O(1) | Belady's anomaly |
| Optimal | 9 | - | Theoretical only |
| LRU | 12 | O(n) or hardware | Best practical |
| Clock | ~LRU | O(1) | Approximates LRU |

```
Page Faults (3 frames):
                FIFO    OPT     LRU     Clock
Reference       15      9       12      ~12
String

Less faults = Better!
OPT is lower bound
```

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Optimal Algorithm practical নয় কেন?
**উত্তর:** ভবিষ্যতে কোন page access হবে জানা সম্ভব নয়। এটা শুধু benchmark হিসেবে ব্যবহৃত হয়।

**প্রশ্ন ২:** LRU ভালো কেন?
**উত্তর:** Locality of reference এর কারণে recently used pages আবার use হওয়ার সম্ভাবনা বেশি। LRU এটা exploit করে।

**প্রশ্ন ৩:** Clock algorithm LRU এর approximation কেন?
**উত্তর:** Reference bit দিয়ে "recently used" track করে। Perfect timing নয়, কিন্তু recent vs old distinguish করে।

---
**পূর্ববর্তী:** [Demand Paging](02-demand-paging.md)  
**পরবর্তী:** [Thrashing](04-thrashing.md)
