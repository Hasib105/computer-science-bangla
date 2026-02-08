# Deadlock Avoidance ও Banker's Algorithm

## 📌 Avoidance কি?

**Deadlock Avoidance** মানে প্রতিটি resource request এ check করা যে এটা দিলে deadlock হতে পারে কিনা। Safe থাকলেই দেওয়া।

```
┌─────────────────────────────────────────────────────────────┐
│                  Avoidance Strategy                         │
│                                                             │
│   Process requests resource                                 │
│            │                                                │
│            ▼                                                │
│   ┌─────────────────┐                                      │
│   │  Will granting  │                                      │
│   │  this keep us   │                                      │
│   │  in safe state? │                                      │
│   └────────┬────────┘                                      │
│            │                                                │
│       ┌────┴────┐                                          │
│       │         │                                          │
│      YES        NO                                         │
│       │         │                                          │
│       ▼         ▼                                          │
│   ┌───────┐  ┌───────┐                                     │
│   │ Grant │  │ Wait  │                                     │
│   └───────┘  └───────┘                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔒 Safe State vs Unsafe State

### Safe State

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Safe State:                                                │
│  • এমন একটি sequence আছে যেখানে সব process শেষ হতে পারে    │
│  • <P1, P2, P3, ...> sequence exists                       │
│  • প্রতিটি Pi এর needs ≤ available + Σ(allocated to Pj)    │
│    where j < i                                              │
│                                                             │
│  Unsafe State:                                              │
│  • এমন কোনো sequence নেই                                   │
│  • Deadlock হতে পারে (guaranteed নয়)                       │
│                                                             │
│             Safe ──────────────────► Unsafe                 │
│               │                         │                   │
│               │                         │                   │
│               ▼                         ▼                   │
│         No Deadlock              Deadlock Possible          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Safe Sequence Example

```
Resources: 12 units total

Process  Max Need  Allocated  Remaining Need
─────────────────────────────────────────────
  P0        10        5            5
  P1         4        2            2
  P2         9        2            7

Available: 12 - (5+2+2) = 3

Safe Sequence: <P1, P0, P2>

Step 1: P1 needs 2, available 3 → P1 runs, releases 2+2=4
        Available = 3 + 2 = 5
        
Step 2: P0 needs 5, available 5 → P0 runs, releases 5+5=10
        Available = 5 + 5 = 10
        
Step 3: P2 needs 7, available 10 → P2 runs ✓

All processes complete! System is SAFE.
```

---

## 🏦 Banker's Algorithm

### প্রয়োজনীয় Data Structures

```c
int n;  // Number of processes
int m;  // Number of resource types

int Available[m];      // Available resources
int Max[n][m];         // Maximum demand of each process
int Allocation[n][m];  // Currently allocated
int Need[n][m];        // Remaining need (Max - Allocation)
```

```
Example:
                     A  B  C
Available:          [3, 3, 2]

Process   Max       Allocation    Need
         A B C       A B C       A B C
P0      [7,5,3]    [0,1,0]    [7,4,3]
P1      [3,2,2]    [2,0,0]    [1,2,2]
P2      [9,0,2]    [3,0,2]    [6,0,0]
P3      [2,2,2]    [2,1,1]    [0,1,1]
P4      [4,3,3]    [0,0,2]    [4,3,1]
```

---

## 🔍 Safety Algorithm

```c
// Check if system is in safe state
bool isSafe() {
    int Work[m];          // Available resources
    bool Finish[n];       // Process finished?
    
    // Initialize
    for (int i = 0; i < m; i++)
        Work[i] = Available[i];
    for (int i = 0; i < n; i++)
        Finish[i] = false;
    
    // Find a process that can finish
    while (true) {
        bool found = false;
        
        for (int i = 0; i < n; i++) {
            if (!Finish[i] && Need[i] <= Work) {
                // Process i can finish
                Work = Work + Allocation[i];
                Finish[i] = true;
                found = true;
            }
        }
        
        if (!found) break;
    }
    
    // Check if all finished
    for (int i = 0; i < n; i++)
        if (!Finish[i]) return false;
    
    return true;
}
```

### Step-by-Step Example

```
Initial: Work = [3,3,2]

Step 1: Check each process
  P0: Need[7,4,3] ≤ Work[3,3,2]? NO
  P1: Need[1,2,2] ≤ Work[3,3,2]? YES ✓
      Finish[1] = true
      Work = [3,3,2] + [2,0,0] = [5,3,2]

Step 2: 
  P0: Need[7,4,3] ≤ Work[5,3,2]? NO
  P2: Need[6,0,0] ≤ Work[5,3,2]? NO
  P3: Need[0,1,1] ≤ Work[5,3,2]? YES ✓
      Finish[3] = true
      Work = [5,3,2] + [2,1,1] = [7,4,3]

Step 3:
  P0: Need[7,4,3] ≤ Work[7,4,3]? YES ✓
      Finish[0] = true
      Work = [7,4,3] + [0,1,0] = [7,5,3]

Step 4:
  P2: Need[6,0,0] ≤ Work[7,5,3]? YES ✓
      Finish[2] = true
      Work = [7,5,3] + [3,0,2] = [10,5,5]

Step 5:
  P4: Need[4,3,1] ≤ Work[10,5,5]? YES ✓
      Finish[4] = true
      Work = [10,5,5] + [0,0,2] = [10,5,7]

All Finish[i] = true
Safe Sequence: <P1, P3, P0, P2, P4>
SAFE STATE! ✓
```

---

## 📝 Resource Request Algorithm

```c
// Process Pi requests resources Request[m]
bool requestResources(int i, int Request[]) {
    // Step 1: Check if request is valid
    if (Request > Need[i]) {
        error("Request exceeds maximum claim");
        return false;
    }
    
    // Step 2: Check if resources available
    if (Request > Available) {
        // Process must wait
        return false;
    }
    
    // Step 3: Pretend to allocate (temporarily)
    Available = Available - Request;
    Allocation[i] = Allocation[i] + Request;
    Need[i] = Need[i] - Request;
    
    // Step 4: Check if safe
    if (isSafe()) {
        // Allocation is safe, keep it
        return true;
    } else {
        // Rollback
        Available = Available + Request;
        Allocation[i] = Allocation[i] - Request;
        Need[i] = Need[i] + Request;
        return false;
    }
}
```

### Request Example

```
Current State (from above):
Available = [3,3,2]

P1 requests [1,0,2]

Step 1: Request[1,0,2] ≤ Need[1][1,2,2]? YES ✓

Step 2: Request[1,0,2] ≤ Available[3,3,2]? YES ✓

Step 3: Temporary allocation
  Available = [3,3,2] - [1,0,2] = [2,3,0]
  Allocation[1] = [2,0,0] + [1,0,2] = [3,0,2]
  Need[1] = [1,2,2] - [1,0,2] = [0,2,0]

Step 4: Run Safety Algorithm with new state
  → Safe sequence exists
  → Grant request ✓
```

---

## 📊 Complexity Analysis

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Safety Algorithm: O(m × n²)                                │
│  • Outer loop: O(n) times                                   │
│  • Finding process: O(n)                                    │
│  • Comparing vectors: O(m)                                  │
│                                                             │
│  Resource Request: O(m × n²)                                │
│  • Includes Safety Algorithm                                │
│                                                             │
│  Space: O(m × n)                                           │
│  • Storing matrices                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚠️ Banker's Algorithm এর সীমাবদ্ধতা

```
1. Fixed number of processes
   • নতুন process আসলে recalculate করতে হয়

2. Maximum need জানতে হয়
   • অনেক সময় আগে থেকে জানা যায় না

3. Fixed number of resources
   • Resource add/remove করা কঠিন

4. Processes must return resources
   • Process crash করলে?

5. Overhead
   • প্রতিটি request এ check করতে হয়
```

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Safe state ও Unsafe state এর পার্থক্য কি?
**উত্তর:**
- Safe: একটি sequence আছে যেখানে সব process complete হতে পারে
- Unsafe: এমন sequence নেই, deadlock হতে পারে

**প্রশ্ন ২:** Banker's Algorithm এ কোন data structures লাগে?
**উত্তর:** Available[], Max[][], Allocation[][], Need[][] (Need = Max - Allocation)

**প্রশ্ন ৩:** Unsafe state মানেই কি Deadlock?
**উত্তর:** না। Unsafe state মানে deadlock সম্ভব, কিন্তু নিশ্চিত নয়। যদি processes তাদের max claim থেকে কম চায়, deadlock নাও হতে পারে।

---
**পূর্ববর্তী:** [Deadlock Prevention](03-prevention.md)  
**পরবর্তী:** [Detection and Recovery](05-detection-recovery.md)
