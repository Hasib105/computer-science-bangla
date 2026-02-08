# Mutex ও Semaphore

## 📌 Mutex (Mutual Exclusion)

**Mutex** হলো একটি লক যা একটি সময়ে শুধুমাত্র একটি থ্রেডকে Critical Section এ প্রবেশের অনুমতি দেয়।

```
┌─────────────────────────────────────────────────────────────┐
│                         MUTEX                               │
│                                                             │
│       Lock (key)                                            │
│          🔐                                                 │
│           │                                                 │
│   ┌───────┴───────┐                                        │
│   │               │                                        │
│   │  Critical     │    Thread 1: Has the key 🔑            │
│   │   Section     │                                        │
│   │               │    Thread 2: Waiting...                │
│   └───────────────┘    Thread 3: Waiting...                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Mutex Operations

```c
mutex m;

// Thread
lock(m);        // Acquire the lock
// Critical Section
unlock(m);      // Release the lock
```

### Mutex উদাহরণ (pthreads)

```c
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int shared_counter = 0;

void* increment(void* arg) {
    for (int i = 0; i < 100000; i++) {
        pthread_mutex_lock(&mutex);     // Lock
        shared_counter++;               // Critical Section
        pthread_mutex_unlock(&mutex);   // Unlock
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    
    printf("Counter: %d\n", shared_counter);  // 200000 ✓
    return 0;
}
```

---

## 📌 Semaphore (সেমাফোর)

**Semaphore** হলো একটি পূর্ণসংখ্যা ভেরিয়েবল যা shared resource access নিয়ন্ত্রণ করে।

### দুই ধরনের Semaphore

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Binary Semaphore (0 বা 1)                                 │
│  ═══════════════════════════                               │
│  • Mutex এর মতো                                            │
│  • একটি resource                                           │
│  • Value: 0 (locked) or 1 (unlocked)                       │
│                                                             │
│  Counting Semaphore (0 থেকে N)                             │
│  ═══════════════════════════════                           │
│  • Multiple resources                                       │
│  • N টি identical resource থাকলে                          │
│  • Value: 0 to N                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Semaphore Operations

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  wait(S) / P(S) / down(S)                                  │
│  ════════════════════════                                  │
│  if (S > 0)                                                │
│      S = S - 1;                                            │
│  else                                                       │
│      block();  // Queue তে যাও                             │
│                                                             │
│  signal(S) / V(S) / up(S)                                  │
│  ════════════════════════                                  │
│  S = S + 1;                                                │
│  if (কেউ waiting আছে)                                      │
│      wakeup(one process);                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Semaphore Implementation

```c
typedef struct {
    int value;
    struct process *list;  // Waiting queue
} semaphore;

void wait(semaphore *S) {
    S->value--;
    if (S->value < 0) {
        // Add this process to S->list
        block();
    }
}

void signal(semaphore *S) {
    S->value++;
    if (S->value <= 0) {
        // Remove a process P from S->list
        wakeup(P);
    }
}
```

---

## 💡 Semaphore ব্যবহার

### ১. Mutual Exclusion (Binary Semaphore)

```c
semaphore mutex = 1;

// Process
wait(mutex);        // Entry section
// Critical Section
signal(mutex);      // Exit section
```

```
Timeline:
┌─────────────────────────────────────────────────────────────┐
│  Semaphore value:  1 → 0 → 0 → 1 → 0 → 1                   │
│                    ↑   ↑       ↑   ↑   ↑                    │
│                    │   │       │   │   │                    │
│  P1:           wait() │    signal()│   │                    │
│  P2:               wait()      │ signal()                   │
│                    (blocked)   │       │                    │
│                             (wakeup)   │                    │
└─────────────────────────────────────────────────────────────┘
```

### ২. Resource Counting (Counting Semaphore)

```c
// 5 টি printer আছে
semaphore printers = 5;

void use_printer() {
    wait(printers);      // প্রিন্টার নাও (value--)
    // Use the printer
    signal(printers);    // প্রিন্টার ছাড়ো (value++)
}
```

### ৩. Process Synchronization (Ordering)

```c
// P1 এর statement S1 এর পর P2 এর statement S2 চলবে

semaphore sync = 0;

// Process P1
S1;
signal(sync);   // S1 শেষ হয়েছে বলো

// Process P2
wait(sync);     // S1 শেষ হওয়া পর্যন্ত অপেক্ষা
S2;
```

---

## 📊 Mutex vs Semaphore

| বৈশিষ্ট্য | Mutex | Semaphore |
|----------|-------|-----------|
| Value | Binary (0/1) | Integer (0 to N) |
| Ownership | আছে (যে lock করে সেই unlock) | নেই |
| Purpose | Mutual Exclusion | Synchronization + Resource counting |
| Release | শুধু owner | যেকোনো process |
| Typical Use | Critical section | Producer-Consumer, Resource pool |

---

## ⚠️ Semaphore সমস্যা

### ১. Deadlock

```c
// Two semaphores
semaphore S = 1, Q = 1;

// Process P0              // Process P1
wait(S);                   wait(Q);
wait(Q);                   wait(S);  // Deadlock!
...                        ...
signal(S);                 signal(Q);
signal(Q);                 signal(S);
```

```
P0: wait(S) → S=0 ✓
P1: wait(Q) → Q=0 ✓
P0: wait(Q) → blocked (Q=0)
P1: wait(S) → blocked (S=0)

দুজনেই অপেক্ষায়! Deadlock!
```

### ২. Starvation

```
P1, P2, P3 সব সময় ready
P4 কখনোই CPU পায় না

সমাধান: FIFO waiting queue
```

### ৩. Priority Inversion

```
Low priority process holds lock
High priority process waits for that lock
Medium priority process runs instead

সমাধান: Priority inheritance
```

---

## 💻 Producer-Consumer with Semaphore

```c
#define BUFFER_SIZE 10

int buffer[BUFFER_SIZE];
int in = 0, out = 0;

semaphore mutex = 1;      // Mutual exclusion
semaphore empty = BUFFER_SIZE;  // Empty slots
semaphore full = 0;       // Filled slots

// Producer
void producer() {
    int item;
    while (true) {
        item = produce_item();
        
        wait(empty);      // Wait for empty slot
        wait(mutex);      // Enter CS
        
        buffer[in] = item;
        in = (in + 1) % BUFFER_SIZE;
        
        signal(mutex);    // Exit CS
        signal(full);     // Increment full count
    }
}

// Consumer
void consumer() {
    int item;
    while (true) {
        wait(full);       // Wait for item
        wait(mutex);      // Enter CS
        
        item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        
        signal(mutex);    // Exit CS
        signal(empty);    // Increment empty count
        
        consume_item(item);
    }
}
```

```
Buffer State Diagram:
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│   │ A │ B │ C │   │   │   │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
      ↑           ↑
     out          in

empty = 7 (খালি স্লট)
full = 3 (ভর্তি স্লট)
```

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Mutex ও Binary Semaphore এর পার্থক্য কি?

| Mutex | Binary Semaphore |
|-------|------------------|
| Ownership আছে | Ownership নেই |
| শুধু owner unlock করে | যেকেউ signal করতে পারে |
| Locking purpose | Synchronization purpose |

**প্রশ্ন ২:** wait() ও signal() কে atomic হতে হয় কেন?
**উত্তর:** না হলে race condition হবে। দুই প্রসেস একসাথে semaphore modify করলে সমস্যা হবে।

**প্রশ্ন ৩:** Counting Semaphore এর value negative হলে কি বোঝায়?
**উত্তর:** |value| সংখ্যক প্রসেস waiting queue তে আছে।

---
**পূর্ববর্তী:** [Race Condition](02-race-condition.md)  
**পরবর্তী:** [Classic Problems](04-classic-problems.md)
