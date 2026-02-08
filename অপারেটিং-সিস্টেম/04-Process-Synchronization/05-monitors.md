# Monitors (মনিটর)

## 📌 Monitor কি?

**Monitor** হলো একটি high-level synchronization construct যা shared data এবং তা access করার procedures একসাথে encapsulate করে।

```
┌─────────────────────────────────────────────────────────────┐
│                        MONITOR                              │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                                                       │ │
│  │            Shared Data                               │ │
│  │       ┌─────────────────────┐                        │ │
│  │       │  data1, data2, ...  │                        │ │
│  │       └─────────────────────┘                        │ │
│  │                                                       │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │ │
│  │  │ Procedure 1 │  │ Procedure 2 │  │ Procedure 3 │  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │ │
│  │                                                       │ │
│  │            Condition Variables                        │ │
│  │       ┌─────────────────────────┐                    │ │
│  │       │   x, y, ...            │                    │ │
│  │       └─────────────────────────┘                    │ │
│  │                                                       │ │
│  └───────────────────────────────────────────────────────┘ │
│                          │                                  │
│   Entry Queue ──────────►│◄────── Only ONE process at     │
│   (waiting to enter)     │        a time inside monitor    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔑 Monitor এর বৈশিষ্ট্য

```
┌─────────────────────────────────────────────────────────────┐
│  1. Mutual Exclusion স্বয়ংক্রিয়                            │
│     • একটি সময়ে শুধু একটি process monitor এ থাকে          │
│     • Programmer কে manually lock করতে হয় না             │
│                                                             │
│  2. Condition Variables                                     │
│     • wait() - process wait করে                            │
│     • signal() - waiting process কে জাগায়                 │
│                                                             │
│  3. Encapsulation                                           │
│     • Data এবং procedures একসাথে                           │
│     • শুধু monitor procedures দ্বারা data access           │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 Condition Variables

### wait() এবং signal()

```c
condition x;

x.wait();   // Process suspend হয়, monitor lock ছাড়ে
x.signal(); // একটি waiting process কে জাগায়
```

### Signal Semantics

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Signal and Wait (Hoare):                                   │
│  • signal() করার পর signaler wait করে                     │
│  • signaled process immediately run করে                    │
│                                                             │
│  Signal and Continue (Mesa/Java):                           │
│  • signal() করার পর signaler চলতে থাকে                    │
│  • signaled process ready queue তে যায়                    │
│  • condition পুনরায় check করতে হয় (while loop)           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 💻 Monitor Syntax (Pseudo-code)

```
monitor monitor_name {
    // Shared variables
    shared_variable declarations;
    
    // Condition variables
    condition c1, c2, ...;
    
    // Procedures
    procedure P1(...) {
        // code
    }
    
    procedure P2(...) {
        // code
    }
    
    // Initialization code
    initialization_code {
        // initialize shared variables
    }
}
```

---

## 💡 Producer-Consumer with Monitor

```c
monitor BoundedBuffer {
    int buffer[N];
    int count = 0, in = 0, out = 0;
    
    condition notFull, notEmpty;
    
    procedure produce(int item) {
        while (count == N) {
            notFull.wait();      // Buffer full, wait
        }
        
        buffer[in] = item;
        in = (in + 1) % N;
        count++;
        
        notEmpty.signal();       // Wake up consumer
    }
    
    procedure consume() {
        while (count == 0) {
            notEmpty.wait();     // Buffer empty, wait
        }
        
        int item = buffer[out];
        out = (out + 1) % N;
        count--;
        
        notFull.signal();        // Wake up producer
        
        return item;
    }
}

// Usage
void producer() {
    while (true) {
        item = produce_item();
        BoundedBuffer.produce(item);
    }
}

void consumer() {
    while (true) {
        item = BoundedBuffer.consume();
        consume_item(item);
    }
}
```

---

## 💡 Dining Philosophers with Monitor

```c
monitor DiningPhilosophers {
    enum {THINKING, HUNGRY, EATING} state[5];
    condition self[5];
    
    procedure pickup(int i) {
        state[i] = HUNGRY;
        test(i);
        
        if (state[i] != EATING) {
            self[i].wait();
        }
    }
    
    procedure putdown(int i) {
        state[i] = THINKING;
        
        // Check left and right neighbors
        test((i + 4) % 5);
        test((i + 1) % 5);
    }
    
    procedure test(int i) {
        if (state[(i+4) % 5] != EATING &&
            state[i] == HUNGRY &&
            state[(i+1) % 5] != EATING) {
            
            state[i] = EATING;
            self[i].signal();
        }
    }
    
    initialization_code {
        for (int i = 0; i < 5; i++) {
            state[i] = THINKING;
        }
    }
}

// Usage
void philosopher(int i) {
    while (true) {
        think();
        DiningPhilosophers.pickup(i);
        eat();
        DiningPhilosophers.putdown(i);
    }
}
```

---

## 📊 Semaphore vs Monitor

| বৈশিষ্ট্য | Semaphore | Monitor |
|----------|-----------|---------|
| Level | Low-level | High-level |
| Mutual Exclusion | Manual (wait/signal) | Automatic |
| Encapsulation | নেই | আছে |
| Error-prone | হ্যাঁ | কম |
| Condition sync | সরাসরি নয় | Condition variables |
| Language support | System call | Language construct |

---

## ☕ Java Monitor (synchronized)

```java
class BoundedBuffer {
    private int[] buffer = new int[N];
    private int count = 0, in = 0, out = 0;
    
    public synchronized void produce(int item) 
            throws InterruptedException {
        while (count == N) {
            wait();              // Buffer full
        }
        
        buffer[in] = item;
        in = (in + 1) % N;
        count++;
        
        notifyAll();             // Wake up consumers
    }
    
    public synchronized int consume() 
            throws InterruptedException {
        while (count == 0) {
            wait();              // Buffer empty
        }
        
        int item = buffer[out];
        out = (out + 1) % N;
        count--;
        
        notifyAll();             // Wake up producers
        return item;
    }
}
```

### Java synchronized keyword

```java
// Method-level synchronization
public synchronized void method() {
    // Only one thread at a time
}

// Block-level synchronization
public void method() {
    synchronized(this) {
        // Only one thread at a time
    }
}
```

---

## ⚠️ Monitor এর সীমাবদ্ধতা

```
┌─────────────────────────────────────────────────────────────┐
│  1. Language Support প্রয়োজন                               │
│     • সব language support করে না                           │
│     • Java, C#, Python (with locks) support করে           │
│                                                             │
│  2. Distributed Systems এ কাজ করে না                       │
│     • Single machine solution                               │
│                                                             │
│  3. Signal Semantics বিভ্রান্তিকর হতে পারে                 │
│     • Hoare vs Mesa                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Monitor ও Semaphore এর প্রধান পার্থক্য কি?
**উত্তর:**
- Monitor: High-level, automatic mutual exclusion
- Semaphore: Low-level, manual synchronization

**প্রশ্ন ২:** Condition Variable কেন প্রয়োজন?
**উত্তর:** Monitor এর ভিতরে বিশেষ শর্তের জন্য wait করতে এবং শর্ত পূর্ণ হলে জাগাতে।

**প্রশ্ন ৩:** কেন while loop ব্যবহার করা হয় wait() এর সাথে?
**উত্তর:** Signal and Continue (Mesa) semantics এ signal এর পর অন্য process আগে চলতে পারে এবং condition পরিবর্তন করতে পারে। তাই পুনরায় check করতে হয়।

---
**পূর্ববর্তী:** [Classic Problems](04-classic-problems.md)  
**পরবর্তী ফোল্ডার:** [Deadlock](../05-Deadlock/README.md)
