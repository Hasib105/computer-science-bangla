# CPU শিডিউলিং - মূল ধারণা (Scheduling Basics)

## 📌 CPU শিডিউলিং কি?

**CPU শিডিউলিং** হলো Ready Queue থেকে কোন প্রসেস পরবর্তীতে CPU পাবে সেটা নির্ধারণ করার প্রক্রিয়া।

```
                      CPU Scheduler
                           │
                           ↓
    ┌─────────────────────────────────────────┐
    │              Ready Queue                 │
    │  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐    │
    │  │ P1 │→│ P4 │→│ P2 │→│ P7 │→│ P3 │    │
    │  └────┘ └────┘ └────┘ └────┘ └────┘    │
    └───────────────────────┬─────────────────┘
                            │
                            │ Select one
                            ↓
                      ┌──────────┐
                      │   CPU    │
                      └──────────┘
```

---

## 🔄 শিডিউলার প্রকারভেদ

### ১. Long-term Scheduler (Job Scheduler)
```
┌────────────────┐     Long-term      ┌────────────┐
│   Job Queue    │ ─────Scheduler────→│Ready Queue │
│   (Disk)       │                    │ (Memory)   │
└────────────────┘                    └────────────┘

কাজ: কোন প্রসেস মেমরিতে লোড হবে
ফ্রিকোয়েন্সি: খুবই কম (সেকেন্ড/মিনিট)
```

### ২. Short-term Scheduler (CPU Scheduler)
```
┌────────────────┐     Short-term     ┌────────────┐
│  Ready Queue   │ ────Scheduler─────→│    CPU     │
│   (Memory)     │                    │            │
└────────────────┘                    └────────────┘

কাজ: কোন প্রসেস CPU পাবে
ফ্রিকোয়েন্সি: অনেক ঘন ঘন (মিলিসেকেন্ড)
```

### ৩. Medium-term Scheduler (Swapper)
```
┌────────────────┐    Medium-term     ┌────────────┐
│    Memory      │ ────Scheduler─────→│    Disk    │
│  (Process)     │                    │  (Swap)    │
└────────────────┘                    └────────────┘

কাজ: Swapping (মেমরি থেকে ডিস্কে সরানো)
```

### তিন শিডিউলারের সম্পর্ক
```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│    New ──Long-term──→ Ready ──Short-term──→ Running     │
│     │                   ↑                      │         │
│     │                   │                      │         │
│     │              Medium-term                 │         │
│     │                   ↑                      │         │
│     │               Suspended                  │         │
│     │                                          │         │
│     └──────────────────────────────────────────→ Exit   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 📊 CPU Burst ও I/O Burst

### Burst Cycle
```
┌──────────────────────────────────────────────────────────┐
│                    Process Execution                      │
│                                                          │
│   CPU      I/O       CPU      I/O       CPU              │
│  Burst    Burst     Burst    Burst     Burst    ...     │
│  ████    ░░░░░░    ████     ░░░░░    ████████           │
│   3ms      5ms      2ms      4ms       5ms              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### প্রসেসের ধরন

| ধরন | বৈশিষ্ট্য | উদাহরণ |
|-----|----------|--------|
| CPU Bound | দীর্ঘ CPU burst | Scientific calculation |
| I/O Bound | ছোট CPU burst, ঘন I/O | Text editor, Web server |

```
CPU-Bound:
CPU █████████████████░░░░█████████████████

I/O-Bound:
CPU ██░░░░██░░░░██░░░░██░░░░██░░░░██░░░░██
```

---

## 🔀 Preemptive vs Non-Preemptive

### Non-Preemptive (সহযোগিতামূলক)
```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Process A starts running                                │
│       │                                                  │
│       ▼                                                  │
│  ████████████████████████████████████████               │
│       │                              │                   │
│       │         A runs until         │                   │
│       │      completion or I/O       │                   │
│       │                              │                   │
│                                      ▼                   │
│                              Process B starts            │
│                                                          │
└──────────────────────────────────────────────────────────┘

CPU ছাড়ে:
• Process সম্পূর্ণ হলে
• I/O অপেক্ষায় গেলে
• wait() করলে
```

### Preemptive (অগ্রাধিকারমূলক)
```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Process A running                                       │
│       │                                                  │
│       ▼                                                  │
│  ████████████│                                          │
│              │ Timer interrupt / Higher priority         │
│              ▼                                           │
│         Process B starts                                 │
│         █████████████████████████                       │
│                                                          │
│  A কে জোর করে সরানো হলো                                 │
│                                                          │
└──────────────────────────────────────────────────────────┘

CPU ছাড়তে বাধ্য:
• Time quantum শেষ হলে
• Higher priority প্রসেস এলে
• Interrupt হলে
```

### তুলনা

| Non-Preemptive | Preemptive |
|----------------|------------|
| সরল | জটিল |
| Context switch কম | Context switch বেশি |
| Response time বেশি | Response time কম |
| Starvation কম | Race condition সম্ভব |
| FCFS, SJF | Round Robin, SRTF |

---

## 🔧 Dispatcher

### কাজ
CPU Scheduler এর সিদ্ধান্ত বাস্তবায়ন করে।

```
┌─────────────────────────────────────────────────────┐
│                    Dispatcher                        │
├─────────────────────────────────────────────────────┤
│  1. Context Switch                                   │
│     • বর্তমান প্রসেসের state সেভ                    │
│     • নতুন প্রসেসের state লোড                       │
│                                                      │
│  2. Switch to User Mode                             │
│     • Kernel mode থেকে User mode                    │
│                                                      │
│  3. Jump to Proper Location                         │
│     • নতুন প্রসেসের সঠিক জায়গায় jump              │
└─────────────────────────────────────────────────────┘
```

### Dispatch Latency
```
Time ─────────────────────────────────────────────→

Process A Running          Process B Running
████████████████│                    │████████████████
                │←─ Dispatch Latency ─→│
                │                    │
         Stop A │ Switch │ Start B   │
                │Context │           │
                │        │           │
                
Dispatch Latency = Context Switch + Mode Switch + Jump
```

---

## 📈 শিডিউলিং কখন হয়?

### চারটি পরিস্থিতি

```
1. Running → Waiting (I/O wait)
   Non-preemptive ✓

2. Running → Ready (Interrupt)
   Preemptive ✓

3. Waiting → Ready (I/O complete)
   Preemptive ✓

4. Running → Terminated
   Non-preemptive ✓
```

### ডায়াগ্রাম
```
                    ┌────────────┐
                    │   Ready    │←───────────┐
                    └─────┬──────┘            │
                          │                   │ ③
                    ②     │ dispatch          │
           ┌──────────────↓                   │
           │        ┌────────────┐      ┌──────────┐
           │        │  Running   │──────│ Waiting  │
           │        └─────┬──────┘  ①   └──────────┘
           │              │
           │              │ ④
           │              ↓
           │        ┌────────────┐
           └────────│ Terminated │
                    └────────────┘
```

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Short-term ও Long-term scheduler এর পার্থক্য কি?

| Long-term | Short-term |
|-----------|------------|
| Disk → Memory | Ready → Running |
| কম ঘন ঘন | অনেক ঘন ঘন |
| Degree of multiprogramming নিয়ন্ত্রণ | CPU utilization নিয়ন্ত্রণ |

**প্রশ্ন ২:** Preemptive শিডিউলিং এ কি সমস্যা হতে পারে?

**উত্তর:**
- Race condition (shared data)
- Context switch overhead
- জটিল implementation

**প্রশ্ন ৩:** Dispatcher ও Scheduler এর পার্থক্য কি?

**উত্তর:**
- Scheduler: কে CPU পাবে সিদ্ধান্ত নেয়
- Dispatcher: সেই সিদ্ধান্ত বাস্তবায়ন করে

---
**পরবর্তী:** [শিডিউলিং মাপকাঠি](02-scheduling-criteria.md)
