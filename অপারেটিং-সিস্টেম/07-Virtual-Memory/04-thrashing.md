# Thrashing ও Working Set

## 📌 Thrashing কি?

**Thrashing** হলো এমন অবস্থা যেখানে system page replacement এ এত busy থাকে যে actual work করার সময় পায় না।

```
┌─────────────────────────────────────────────────────────────┐
│                       Thrashing                             │
│                                                             │
│   CPU Utilization                                           │
│        ▲                                                    │
│        │                    ╱╲                              │
│        │                   ╱  ╲                             │
│        │                  ╱    ╲                            │
│        │       ╱─────────╱      ╲                           │
│        │      ╱                  ╲                          │
│        │     ╱                    ╲ Thrashing!              │
│        │    ╱                      ╲                        │
│        │   ╱                        ╲                       │
│        │──╱                          ╲────────              │
│        └──────────────────────────────────────► Degree of   │
│                                          Multiprogramming   │
│                                                             │
│   • কম process = CPU idle                                   │
│   • বেশি process = page faults, CPU idle (thrashing)       │
│   • মাঝামাঝি = optimal                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 💥 কেন হয়?

```
┌─────────────────────────────────────────────────────────────┐
│                 Thrashing Scenario                          │
│                                                             │
│  1. বেশি process run করছে                                  │
│                │                                            │
│                ▼                                            │
│  2. প্রতিটি process এর জন্য frames কম                      │
│                │                                            │
│                ▼                                            │
│  3. Page faults বাড়ে                                       │
│                │                                            │
│                ▼                                            │
│  4. Processes blocked (I/O wait for page)                  │
│                │                                            │
│                ▼                                            │
│  5. CPU utilization কমে                                    │
│                │                                            │
│                ▼                                            │
│  6. OS thinks "CPU idle, add more processes!"              │
│                │                                            │
│                ▼                                            │
│  7. More processes = Even less frames per process          │
│                │                                            │
│                ▼                                            │
│  8. Even more page faults → Thrashing!                     │
│                                                             │
│  VICIOUS CYCLE!                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Working Set Model

### Working Set কি?

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Working Set = "locality" এর একটি window                  │
│                                                             │
│  WS(t, Δ) = pages referenced in time interval (t-Δ, t)     │
│                                                             │
│  Example (Δ = 10):                                          │
│  Reference: ...2, 6, 1, 5, 7, 7, 7, 7, 5, 1 | current time │
│                                            ↑                │
│                       Working Set Window                    │
│                                                             │
│  WS = {1, 2, 5, 6, 7}                                      │
│  |WS| = 5 (working set size)                               │
│                                                             │
│  Process needs at least 5 frames to avoid thrashing        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Working Set Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Total Demand (D) = Σ |WSi| (all processes)                │
│  Available Frames = m                                       │
│                                                             │
│  If D > m:                                                  │
│      Thrashing will occur!                                  │
│      Solution: Suspend some processes                       │
│                                                             │
│  If D < m:                                                  │
│      Can add more processes                                 │
│      Or give extra frames to existing processes             │
│                                                             │
│  Strategy:                                                  │
│  1. Monitor each process's working set                      │
│  2. Allocate enough frames for working set                  │
│  3. If D > m, suspend processes until D ≤ m                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📈 Page Fault Frequency (PFF)

### Alternative to Working Set

```
┌─────────────────────────────────────────────────────────────┐
│                 Page Fault Frequency                        │
│                                                             │
│   Page Fault                                                │
│   Rate    ▲                                                 │
│           │                                                 │
│           │ ───────────────  Upper Bound                   │
│           │      ╲                                          │
│           │       ╲                                         │
│           │        ╲                                        │
│           │ ────────╲──────  Lower Bound                   │
│           │          ╲                                      │
│           │           ╲                                     │
│           └─────────────────────────────► Frames           │
│                                                             │
│   If fault rate > upper bound:                              │
│       Process needs more frames                             │
│                                                             │
│   If fault rate < lower bound:                              │
│       Process may have too many frames                      │
│       (can take some away)                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 Frame Allocation Strategies

### Equal Allocation

```
Total frames = 100
Processes = 5

Each process gets: 100 / 5 = 20 frames

সমস্যা: বড় process ও ছোট process সমান frames পায়!
```

### Proportional Allocation

```
Total frames = 64
Process sizes: P1 = 10, P2 = 127

Total size = 137
P1's share = 10/137 × 64 ≈ 5 frames
P2's share = 127/137 × 64 ≈ 59 frames

বড় process বেশি frames পায়
```

### Priority Allocation

```
Higher priority process → More frames

If page fault occurs:
• Select victim from lower priority process
• Higher priority processes get better service
```

---

## 🔒 Global vs Local Replacement

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Global Replacement:                                        │
│  • Page replacement যেকোনো process থেকে হতে পারে           │
│  • High priority process low priority এর frame নিতে পারে  │
│  • Better overall throughput                                │
│  • Process এর performance unpredictable                    │
│                                                             │
│  Local Replacement:                                         │
│  • Process শুধু নিজের frames থেকে replace করে             │
│  • Each process isolated                                    │
│  • Consistent performance                                   │
│  • May not use memory optimally                             │
│                                                             │
│  Most systems use Global replacement                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 💡 Thrashing Prevention

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. Working Set Model                                       │
│     Process এর working set size ≤ allocated frames         │
│                                                             │
│  2. Page Fault Frequency                                    │
│     Monitor fault rate, adjust frames accordingly           │
│                                                             │
│  3. Load Control                                            │
│     Limit degree of multiprogramming                        │
│     Suspend processes if memory pressure high               │
│                                                             │
│  4. Swapping                                                │
│     Swap out entire processes temporarily                   │
│                                                             │
│  5. Add More RAM!                                           │
│     The ultimate solution 💰                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Symptoms of Thrashing

| Symptom | Normal | Thrashing |
|---------|--------|-----------|
| CPU Utilization | High | Very Low |
| Disk Activity | Moderate | Very High |
| Page Faults/sec | Low | Very High |
| Response Time | Fast | Very Slow |
| Throughput | Good | Near Zero |

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Thrashing কেন একটি vicious cycle?
**উত্তর:** Page faults বাড়ে → CPU idle → OS বেশি process add করে → আরো page faults → আরো CPU idle → cycle চলতে থাকে।

**প্রশ্ন ২:** Working Set size কিভাবে determine করা হয়?
**উত্তর:** একটি time window (Δ) এ কতগুলো unique pages reference হয়েছে সেটা count করে।

**প্রশ্ন ৩:** Global vs Local replacement - কোনটা কখন ভালো?
**উত্তর:**
- Global: Better memory utilization, unpredictable per-process performance
- Local: Predictable per-process, may waste memory
- Most systems: Global with some isolation mechanisms

---
**পূর্ববর্তী:** [Page Replacement](03-page-replacement.md)  
**পরবর্তী:** [Memory Mapped Files](05-memory-mapped-files.md)
