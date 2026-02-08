# Deadlock Detection ও Recovery

## 📌 Detection কি?

**Deadlock Detection** মানে system কে deadlock হতে দেওয়া এবং পরে detect করে recover করা।

```
┌─────────────────────────────────────────────────────────────┐
│                 Detection & Recovery                        │
│                                                             │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐          │
│   │  Allow   │────►│  Detect  │────►│ Recover  │          │
│   │ Deadlock │     │ Deadlock │     │   from   │          │
│   │ to occur │     │          │     │ Deadlock │          │
│   └──────────┘     └──────────┘     └──────────┘          │
│                                                             │
│   Prevention/Avoidance এর মতো restrict করা হয় না          │
│   বরং হতে দিয়ে পরে সমাধান করা হয়                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Detection Algorithm

### Single Instance of Each Resource Type

**Wait-For Graph ব্যবহার করো:**

```
Resource Allocation Graph → Wait-For Graph
(Remove resource nodes)

RAG:                         Wait-For:
  R1                           
  ↓                           P1 ──────► P2
  P1 ──► R2 ──► P2            ↑          │
  ↑              │             │          │
  │              ▼             └──────────┘
  └───── R3 ◄───┘
  
Wait-For Graph এ cycle = Deadlock
```

```c
// DFS based cycle detection
bool detectDeadlock() {
    bool visited[n], recStack[n];
    
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            if (hasCycleDFS(i, visited, recStack))
                return true;  // Deadlock found
        }
    }
    return false;
}
```

### Multiple Instances of Resources

**Banker's Algorithm এর মতো পদ্ধতি:**

```c
// Data structures
int Available[m];
int Allocation[n][m];
int Request[n][m];     // Current requests (not max)

bool detectDeadlock() {
    int Work[m];
    bool Finish[n];
    
    // Initialize
    for (int i = 0; i < m; i++)
        Work[i] = Available[i];
    
    for (int i = 0; i < n; i++) {
        if (Allocation[i] == 0)
            Finish[i] = true;
        else
            Finish[i] = false;
    }
    
    // Find unmarked process whose request can be satisfied
    while (true) {
        bool found = false;
        
        for (int i = 0; i < n; i++) {
            if (!Finish[i] && Request[i] <= Work) {
                Work = Work + Allocation[i];
                Finish[i] = true;
                found = true;
            }
        }
        
        if (!found) break;
    }
    
    // Check for deadlock
    for (int i = 0; i < n; i++) {
        if (!Finish[i])
            return true;  // Process i is deadlocked
    }
    return false;
}
```

---

## 💡 Detection Example

```
Resources: A=7, B=2, C=6

Process  Allocation    Request     Available
          A B C        A B C        A B C
─────────────────────────────────────────────
  P0     [0,1,0]     [0,0,0]     [0,0,0]
  P1     [2,0,0]     [2,0,2]
  P2     [3,0,3]     [0,0,0]
  P3     [2,1,1]     [1,0,0]
  P4     [0,0,2]     [0,0,2]

Step 1: Work = [0,0,0]

Step 2: Find process with Request ≤ Work
  P0: [0,0,0] ≤ [0,0,0]? YES ✓
      Finish[0] = true
      Work = [0,0,0] + [0,1,0] = [0,1,0]

  P2: [0,0,0] ≤ [0,1,0]? YES ✓
      Finish[2] = true
      Work = [0,1,0] + [3,0,3] = [3,1,3]

  P3: [1,0,0] ≤ [3,1,3]? YES ✓
      Finish[3] = true
      Work = [3,1,3] + [2,1,1] = [5,2,4]

  P1: [2,0,2] ≤ [5,2,4]? YES ✓
      Finish[1] = true
      Work = [5,2,4] + [2,0,0] = [7,2,4]

  P4: [0,0,2] ≤ [7,2,4]? YES ✓
      Finish[4] = true
      Work = [7,2,4] + [0,0,2] = [7,2,6]

All Finish[i] = true → No Deadlock ✓
```

### Deadlock Case

```
যদি P2 এর Request = [0,0,1] হতো:

Step 1: Work = [0,0,0]
  P0: Request [0,0,0] ≤ Work [0,0,0]? YES ✓
      Finish[0] = true, Work = [0,1,0]

Step 2:
  P1: [2,0,2] ≤ [0,1,0]? NO
  P2: [0,0,1] ≤ [0,1,0]? NO (0 ≤ 0 but 1 > 0)
  P3: [1,0,0] ≤ [0,1,0]? NO
  P4: [0,0,2] ≤ [0,1,0]? NO

No more progress possible!

Finish = [T, F, F, F, F]
Deadlocked processes: P1, P2, P3, P4
```

---

## ⏰ কখন Detection চালাবে?

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Option 1: প্রতিটি resource request এ                       │
│  • সবচেয়ে দ্রুত detect করে                                │
│  • সবচেয়ে বেশি overhead                                    │
│                                                             │
│  Option 2: নির্দিষ্ট সময় পর পর                            │
│  • যেমন প্রতি ঘণ্টায়                                       │
│  • Moderate overhead                                        │
│                                                             │
│  Option 3: CPU utilization কমে গেলে                        │
│  • Deadlock হলে CPU idle থাকে                              │
│  • Smart approach                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 Recovery Methods

### ১. Process Termination

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Option A: সব deadlocked process terminate করো             │
│  • Simple কিন্তু costly                                    │
│  • সব কাজ হারিয়ে যায়                                      │
│                                                             │
│  Option B: একটি একটি করে terminate করো                    │
│  • প্রতিবার detection run করো                              │
│  • কাকে terminate করবে?                                    │
│    - Lowest priority                                        │
│    - Least computation done                                 │
│    - Most resources held                                    │
│    - Longest remaining time                                 │
│    - Interactive vs batch                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### ২. Resource Preemption

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Selecting a victim:                                        │
│  • কার resource নেবে?                                      │
│  • Minimum cost এর victim বাছাই                            │
│                                                             │
│  Rollback:                                                  │
│  • Victim process কে কোথায় rollback করবে?                 │
│  • Safe state এ ফিরে যেতে হবে                              │
│  • Checkpointing দরকার                                     │
│                                                             │
│  Starvation:                                                │
│  • একই process বারবার victim হলে?                          │
│  • Rollback count limit রাখতে হবে                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Recovery Visualization

```
Deadlock State:
    P1 ←───── R1 ←───── P2
    │                    ↑
    └──── R2 ────────────┘

Recovery Option 1: Kill P1
    P2 gets R1 → continues ✓
    P1's work is lost ✗

Recovery Option 2: Preempt R2 from P1
    P2 gets R2 → gets R1 → finishes
    P2 releases R1, R2
    P1 restarts → gets resources ✓
```

---

## 📊 Method তুলনা

| পদ্ধতি | Overhead | Resource Use | Complexity |
|--------|----------|--------------|------------|
| Prevention | Low | Low (wasteful) | Low |
| Avoidance | Medium | Medium | Medium |
| Detection | High | High (efficient) | High |
| Ignore | None | Best | None |

---

## 💡 Combined Approach

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Real systems often combine methods:                        │
│                                                             │
│  Internal resources (kernel structures):                    │
│  • Prevention via ordering                                  │
│                                                             │
│  Main memory:                                               │
│  • Prevention via preemption (swap out)                     │
│                                                             │
│  Job resources (printers, etc.):                            │
│  • Avoidance (Banker's Algorithm)                           │
│                                                             │
│  Swappable space:                                           │
│  • Prevention via allocation at once                        │
│                                                             │
│  User files:                                                │
│  • Detection and recovery (or just ignore!)                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Detection ও Avoidance এর পার্থক্য কি?
**উত্তর:**
- Avoidance: Request grant করার আগে check করে
- Detection: Deadlock হতে দেয়, পরে detect করে

**প্রশ্ন ২:** Recovery তে Rollback কেন কঠিন?
**উত্তর:** 
- Safe state জানতে হয়
- Checkpointing দরকার
- কিছু operation (I/O) rollback করা যায় না

**প্রশ্ন ৩:** Ostrich Algorithm কি?
**উত্তর:** Deadlock ignore করা। যদি deadlock rare হয় এবং prevention/detection এর cost বেশি হয়, তাহলে এই approach practical। UNIX/Linux এটি ব্যবহার করে।

---
**পূর্ববর্তী:** [Deadlock Avoidance](04-avoidance.md)  
**পরবর্তী ফোল্ডার:** [Memory Management](../06-Memory-Management/README.md)
