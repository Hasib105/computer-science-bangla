# শিডিউলিং অ্যালগরিদম তুলনা (Comparison)

## 📊 সব অ্যালগরিদমের সারসংক্ষেপ

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      CPU Scheduling Algorithms                              │
├───────────────┬─────────────┬─────────────┬─────────────┬──────────────────┤
│ Algorithm     │ Selection   │ Preemptive  │ Starvation  │ Complexity       │
│               │ Criteria    │             │             │                  │
├───────────────┼─────────────┼─────────────┼─────────────┼──────────────────┤
│ FCFS          │ Arrival     │ No          │ No          │ Simple           │
│               │ Time        │             │             │                  │
├───────────────┼─────────────┼─────────────┼─────────────┼──────────────────┤
│ SJF           │ Burst Time  │ No          │ Yes         │ Medium           │
│               │ (shortest)  │             │ (long jobs) │                  │
├───────────────┼─────────────┼─────────────┼─────────────┼──────────────────┤
│ SRTF          │ Remaining   │ Yes         │ Yes         │ High             │
│               │ Time        │             │             │                  │
├───────────────┼─────────────┼─────────────┼─────────────┼──────────────────┤
│ Priority      │ Priority    │ Both        │ Yes         │ Medium           │
│               │ Value       │             │ (low prio)  │                  │
├───────────────┼─────────────┼─────────────┼─────────────┼──────────────────┤
│ Round Robin   │ Time        │ Yes         │ No          │ Medium           │
│               │ Quantum     │             │             │                  │
├───────────────┼─────────────┼─────────────┼─────────────┼──────────────────┤
│ Multilevel Q  │ Queue +     │ Varies      │ Possible    │ High             │
│               │ Priority    │             │             │                  │
├───────────────┼─────────────┼─────────────┼─────────────┼──────────────────┤
│ MLFQ          │ Feedback    │ Yes         │ No (aging)  │ Very High        │
│               │             │             │             │                  │
└───────────────┴─────────────┴─────────────┴─────────────┴──────────────────┘
```

---

## 📈 পারফরম্যান্স তুলনা

### Average Waiting Time

```
সাধারণ ক্রম (সেরা থেকে খারাপ):

1. SRTF/SJF    ─────█
2. Priority   ───────██
3. MLFQ       ────────███
4. RR         ──────────████
5. FCFS       ────────────█████

Note: Burst time বিন্যাসের উপর নির্ভর করে
```

### Response Time

```
Interactive system এর জন্য:

1. Round Robin ─────█
2. MLFQ        ───────██
3. SRTF        ─────────███
4. Priority    ───────────████
5. SJF         ─────────────█████
6. FCFS        ───────────────██████
```

### Throughput

```
Batch system এর জন্য:

1. SJF/SRTF   ─────█████████████████
2. Priority   ─────████████████████
3. FCFS       ─────██████████████
4. RR (big TQ)─────████████████
5. RR (sm TQ) ─────█████████
```

---

## 🎯 কোন অ্যালগরিদম কখন ব্যবহার করবেন

### সিস্টেম টাইপ অনুযায়ী

```
┌────────────────────────────────────────────────────────────────┐
│  Batch System                                                  │
│  ├── প্রধান লক্ষ্য: Throughput, CPU Utilization               │
│  └── সেরা: SJF, FCFS                                          │
│                                                                │
│  Interactive System                                            │
│  ├── প্রধান লক্ষ্য: Response Time                             │
│  └── সেরা: Round Robin, MLFQ                                  │
│                                                                │
│  Real-time System                                              │
│  ├── প্রধান লক্ষ্য: Meeting Deadlines                         │
│  └── সেরা: Priority (Preemptive), EDF                         │
│                                                                │
│  General Purpose (Desktop)                                     │
│  ├── প্রধান লক্ষ্য: Balance                                   │
│  └── সেরা: MLFQ                                               │
└────────────────────────────────────────────────────────────────┘
```

### পরিস্থিতি অনুযায়ী

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  যদি সব burst time সমান হয়:                                  │
│  → FCFS সবচেয়ে সহজ, RR ও সমান ফল দেবে                        │
│                                                                │
│  যদি burst time জানা থাকে:                                    │
│  → SJF/SRTF সেরা Waiting Time দেবে                            │
│                                                                │
│  যদি burst time অজানা থাকে:                                   │
│  → RR বা MLFQ ব্যবহার করুন                                    │
│                                                                │
│  যদি process importance আলাদা হয়:                            │
│  → Priority Scheduling ব্যবহার করুন                           │
│                                                                │
│  যদি fairness গুরুত্বপূর্ণ হয়:                                │
│  → Round Robin ব্যবহার করুন                                   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 💡 একই সমস্যায় সব অ্যালগরিদমের তুলনা

### সমস্যা
```
Process  Arrival Time  Burst Time  Priority
P1       0             10          3
P2       0             1           1
P3       0             2           4
P4       0             1           5
P5       0             5           2
```

### FCFS (Order: P1→P2→P3→P4→P5)
```
Gantt: |  P1   | P2 | P3  | P4 | P5    |
       0      10   11    13   14     19

WT:  P1=0, P2=10, P3=11, P4=13, P5=14
Avg WT = 9.6
```

### SJF (Order: P2→P4→P3→P5→P1)
```
Gantt: |P2|P4| P3  |  P5   |    P1    |
       0  1  2    4       9         19

WT:  P1=9, P2=0, P3=2, P4=1, P5=4
Avg WT = 3.2
```

### Priority (Order: P2→P5→P1→P3→P4)
```
Gantt: |P2|  P5   |    P1    | P3  |P4|
       0  1      6         16    18  19

WT:  P1=6, P2=0, P3=16, P4=18, P5=1
Avg WT = 8.2
```

### Round Robin (TQ=2)
```
Gantt: |P1|P2|P3|P4|P5|P1|P5|P1|P5|P1|P1|
       0  2  3  5  6  8 10 12 13 15 17 19

WT:  P1=9, P2=2, P3=3, P4=5, P5=8
Avg WT = 5.4
```

### তুলনা সারণী

| Algorithm | Avg WT | Avg TAT |
|-----------|--------|---------|
| FCFS | 9.6 | 13.4 |
| SJF | **3.2** | **7.0** |
| Priority | 8.2 | 12.0 |
| RR (TQ=2) | 5.4 | 9.2 |

**Winner: SJF** (সর্বনিম্ন Waiting Time)

---

## 📊 Trade-off বিশ্লেষণ

```
                    Response Time
                         ▲
                         │
                         │    ★ RR
                         │
                    ★ MLFQ
                         │
                         │        ★ Priority
                         │
                         │            ★ SJF
                         │
                         │                ★ FCFS
                         │
                         └──────────────────────────────→
                              Throughput / CPU Utilization

★ = Algorithm position (approximate)
```

---

## 🔧 বাস্তব OS এ ব্যবহার

### Linux
```
┌────────────────────────────────────────────────────────────┐
│  Linux CFS (Completely Fair Scheduler)                     │
│  • Red-black tree based                                    │
│  • Virtual runtime tracking                                │
│  • O(log n) complexity                                     │
│  • Nice values for priority (-20 to +19)                  │
└────────────────────────────────────────────────────────────┘
```

### Windows
```
┌────────────────────────────────────────────────────────────┐
│  Windows Thread Scheduler                                  │
│  • Preemptive, priority-based                              │
│  • 32 priority levels (0-31)                              │
│  • Round robin within same priority                        │
│  • Priority boost for I/O completion                       │
└────────────────────────────────────────────────────────────┘
```

### macOS
```
┌────────────────────────────────────────────────────────────┐
│  macOS Grand Central Dispatch (GCD)                        │
│  • Quality of Service (QoS) classes                        │
│  • User Interactive > User Initiated > Utility > Background │
│  • Thread pool management                                   │
└────────────────────────────────────────────────────────────┘
```

---

## ✍️ পরীক্ষায় গুরুত্বপূর্ণ প্রশ্ন

### প্রশ্ন ১: কোনটি সর্বনিম্ন Average Waiting Time দেয়?
**উত্তর:** SJF (প্রমাণিতভাবে optimal)

### প্রশ্ন ২: কোনটিতে Starvation নেই?
**উত্তর:** FCFS, Round Robin

### প্রশ্ন ৩: কোনটি Interactive System এ সেরা?
**উত্তর:** Round Robin বা MLFQ

### প্রশ্ন ৪: কোনটিতে Context Switch সবচেয়ে বেশি?
**উত্তর:** Round Robin (ছোট TQ তে)

### প্রশ্ন ৫: কোনটি সবচেয়ে সহজ implement করতে?
**উত্তর:** FCFS

---

## 📝 Quick Reference Card

```
┌──────────────────────────────────────────────────────────────┐
│                    QUICK REFERENCE                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FCFS:     আগে আসলে আগে পায় | No Starvation | Simple      │
│                                                              │
│  SJF:      সবচেয়ে ছোট আগে | Optimal WT | Starvation আছে   │
│                                                              │
│  SRTF:     SJF + Preemptive | Best WT | Complex             │
│                                                              │
│  Priority: গুরুত্বপূর্ণ আগে | Flexible | Starvation        │
│            Aging দিয়ে সমাধান                                │
│                                                              │
│  RR:       সবাই সমান সময় পায় | Fair | বেশি Switching      │
│            TQ বড় = FCFS, TQ ছোট = overhead                 │
│                                                              │
│  MLQ:      আলাদা Queue | Different treatment | Complex      │
│                                                              │
│  MLFQ:     Adaptive | Best overall | Most Complex           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---
**পূর্ববর্তী:** [Multilevel Queue](07-multilevel-queue.md)  
**পরবর্তী ফোল্ডার:** [Process Synchronization](../04-Process-Synchronization/README.md)
