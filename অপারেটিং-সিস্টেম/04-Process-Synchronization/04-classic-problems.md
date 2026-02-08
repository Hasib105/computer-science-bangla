# ক্লাসিক সিনক্রোনাইজেশন সমস্যা

## 📚 তিনটি ক্লাসিক সমস্যা

```
┌─────────────────────────────────────────────────────────────┐
│  1. Producer-Consumer Problem (Bounded Buffer)              │
│  2. Readers-Writers Problem                                 │
│  3. Dining Philosophers Problem                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Producer-Consumer Problem

### সমস্যার বর্ণনা

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Producer(s)          Buffer           Consumer(s)         │
│       │              ┌───────┐              │               │
│       │  produce     │ │ │ │ │    consume   │               │
│       ├─────────────►│ │▓│▓│ │◄────────────┤               │
│       │              └───────┘              │               │
│                      (bounded)                              │
│                                                             │
│   সমস্যা:                                                   │
│   • Buffer full হলে Producer wait করবে                    │
│   • Buffer empty হলে Consumer wait করবে                   │
│   • একসাথে দুইজন buffer access করবে না                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Semaphore Solution

```c
#define N 10  // Buffer size

int buffer[N];
int in = 0, out = 0;

semaphore mutex = 1;    // Mutual exclusion
semaphore empty = N;    // Count of empty slots
semaphore full = 0;     // Count of full slots

void producer() {
    int item;
    while (true) {
        item = produce_item();
        
        wait(empty);           // Decrement empty count
        wait(mutex);           // Enter critical section
        
        buffer[in] = item;
        in = (in + 1) % N;
        
        signal(mutex);         // Exit critical section
        signal(full);          // Increment full count
    }
}

void consumer() {
    int item;
    while (true) {
        wait(full);            // Decrement full count
        wait(mutex);           // Enter critical section
        
        item = buffer[out];
        out = (out + 1) % N;
        
        signal(mutex);         // Exit critical section
        signal(empty);         // Increment empty count
        
        consume_item(item);
    }
}
```

### Buffer State Visualization

```
Initial: empty=5, full=0, mutex=1
Buffer: [ _ | _ | _ | _ | _ ]

After Producer produces A:
empty=4, full=1
Buffer: [ A | _ | _ | _ | _ ]
            ↑
           in

After Producer produces B, C:
empty=2, full=3
Buffer: [ A | B | C | _ | _ ]
                    ↑
                   in

After Consumer consumes A:
empty=3, full=2
Buffer: [ _ | B | C | _ | _ ]
            ↑       ↑
           out     in
```

---

## 2️⃣ Readers-Writers Problem

### সমস্যার বর্ণনা

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│         ┌──────────────────────┐                           │
│  R1 ───►│                      │                           │
│  R2 ───►│    Shared Database   │◄─── W1                   │
│  R3 ───►│                      │                           │
│         └──────────────────────┘                           │
│                                                             │
│  নিয়ম:                                                     │
│  • Multiple Readers একসাথে পড়তে পারে ✓                   │
│  • Writer থাকলে কেউ access পায় না ✗                      │
│  • Reader থাকলে Writer wait করে                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### First Readers-Writers Solution (Reader Priority)

```c
semaphore mutex = 1;       // Protect read_count
semaphore rw_mutex = 1;    // Exclusive access for writer
int read_count = 0;        // Number of readers

void reader() {
    while (true) {
        wait(mutex);
        read_count++;
        if (read_count == 1) {
            wait(rw_mutex);    // First reader blocks writers
        }
        signal(mutex);
        
        // Read the data
        read_database();
        
        wait(mutex);
        read_count--;
        if (read_count == 0) {
            signal(rw_mutex);  // Last reader unblocks writers
        }
        signal(mutex);
    }
}

void writer() {
    while (true) {
        wait(rw_mutex);        // Get exclusive access
        
        // Write to database
        write_database();
        
        signal(rw_mutex);      // Release exclusive access
    }
}
```

### Timeline Example

```
Time →  1    2    3    4    5    6    7    8    9
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

R1:     ████████████████
R2:          ████████████████
R3:               ████████████
W1:                              blocked... ████████

read_count: 1→2→3→3→2→1→0
            ↑           ↑
        First reader   Last reader
        locks rw_mutex  unlocks rw_mutex

সমস্যা: Writer starvation possible!
```

---

## 3️⃣ Dining Philosophers Problem

### সমস্যার বর্ণনা

```
                    🍝 P0 🍝
                   ⟋      ⟍
              🥢          🥢
             ⟋              ⟍
        P4 🍝                 🍝 P1
            \                /
          🥢                🥢
            \              /
             \            /
         🍝 P3 ───🥢─── P2 🍝

5 জন দার্শনিক, 5 টি চপস্টিক
• চিন্তা করে → খেতে চায় → দুই পাশের chopstick তুলে → খায় → chopstick রাখে
• সমস্যা: সবাই একসাথে বাম chopstick তুললে → Deadlock!
```

### Naive Solution (DEADLOCK!)

```c
#define N 5
semaphore chopstick[N] = {1, 1, 1, 1, 1};

void philosopher(int i) {
    while (true) {
        think();
        
        wait(chopstick[i]);           // Pick left
        wait(chopstick[(i+1) % N]);   // Pick right
        
        eat();
        
        signal(chopstick[i]);         // Put left
        signal(chopstick[(i+1) % N]); // Put right
    }
}

// DEADLOCK scenario:
// P0: pick left (chopstick[0])
// P1: pick left (chopstick[1])
// P2: pick left (chopstick[2])
// P3: pick left (chopstick[3])
// P4: pick left (chopstick[4])
// 
// Now all try to pick right → All blocked!
```

### Solution 1: Asymmetric (সবাই আলাদা নিয়ম)

```c
void philosopher(int i) {
    while (true) {
        think();
        
        if (i % 2 == 0) {
            // Even: pick left first
            wait(chopstick[i]);
            wait(chopstick[(i+1) % N]);
        } else {
            // Odd: pick right first
            wait(chopstick[(i+1) % N]);
            wait(chopstick[i]);
        }
        
        eat();
        
        signal(chopstick[i]);
        signal(chopstick[(i+1) % N]);
    }
}
```

### Solution 2: Limit Philosophers at Table

```c
semaphore room = 4;  // শুধু 4 জন একসাথে বসতে পারবে

void philosopher(int i) {
    while (true) {
        think();
        
        wait(room);                   // Enter room
        wait(chopstick[i]);
        wait(chopstick[(i+1) % N]);
        
        eat();
        
        signal(chopstick[i]);
        signal(chopstick[(i+1) % N]);
        signal(room);                 // Leave room
    }
}
```

### Solution 3: All or Nothing (Atomic Pickup)

```c
semaphore mutex = 1;
enum {THINKING, HUNGRY, EATING} state[N];
semaphore s[N] = {0, 0, 0, 0, 0};

void test(int i) {
    if (state[i] == HUNGRY &&
        state[(i+N-1) % N] != EATING &&
        state[(i+1) % N] != EATING) {
        state[i] = EATING;
        signal(s[i]);
    }
}

void pickup(int i) {
    wait(mutex);
    state[i] = HUNGRY;
    test(i);
    signal(mutex);
    wait(s[i]);      // Block if can't eat
}

void putdown(int i) {
    wait(mutex);
    state[i] = THINKING;
    test((i+N-1) % N);   // Check left neighbor
    test((i+1) % N);     // Check right neighbor
    signal(mutex);
}

void philosopher(int i) {
    while (true) {
        think();
        pickup(i);
        eat();
        putdown(i);
    }
}
```

---

## 📊 সমস্যা তুলনা

| সমস্যা | মূল চ্যালেঞ্জ | সমাধান |
|--------|-------------|--------|
| Producer-Consumer | Buffer synchronization | Three semaphores |
| Readers-Writers | Multiple readers, exclusive writer | Read count + mutex |
| Dining Philosophers | Circular wait → Deadlock | Break symmetry |

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Producer-Consumer এ তিনটি semaphore কেন?
**উত্তর:**
- `mutex`: Critical section protection
- `empty`: Empty slot count
- `full`: Filled slot count

**প্রশ্ন ২:** Readers-Writers এ Writer starvation কেন হয়?
**উত্তর:** নতুন Reader আসতেই থাকলে read_count কখনো 0 হয় না, তাই Writer কখনো rw_mutex পায় না।

**প্রশ্ন ৩:** Dining Philosophers এ Deadlock কখন হয়?
**উত্তর:** সবাই একসাথে বাম chopstick তুললে circular wait হয়।

---
**পূর্ববর্তী:** [Mutex ও Semaphore](03-mutex-semaphore.md)  
**পরবর্তী:** [Monitors](05-monitors.md)
