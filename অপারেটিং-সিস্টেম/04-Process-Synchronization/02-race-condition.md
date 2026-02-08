# Race Condition (রেস কন্ডিশন)

## 📌 Race Condition কি?

**Race Condition** হলো এমন পরিস্থিতি যেখানে একাধিক প্রসেস/থ্রেড একই সময়ে shared data অ্যাক্সেস করে এবং ফলাফল execution order এর উপর নির্ভর করে।

```
┌─────────────────────────────────────────────────────────────┐
│                    Race Condition                           │
│                                                             │
│   Thread 1          Shared Variable (x=0)      Thread 2    │
│       │                    │                       │        │
│       │    x++             │              x++      │        │
│       │────────────────────│──────────────────────►│        │
│       │                    │                       │        │
│       │    Expected: x=2   │   Actual: x=1 or 2    │        │
│       │                    │                       │        │
│   "Race" to access the variable!                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 💥 কেন হয়?

### x++ একটি Atomic Operation নয়!

```
┌─────────────────────────────────────────────────────────────┐
│   x++ আসলে তিনটি step:                                     │
│                                                             │
│   1. LOAD:  Register ← Memory[x]                           │
│   2. ADD:   Register ← Register + 1                        │
│   3. STORE: Memory[x] ← Register                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### সমস্যার উদাহরণ

```
Initial: x = 5

Thread 1                    Thread 2
─────────                   ─────────
1. LOAD R1, x   (R1=5)
                           1. LOAD R2, x   (R2=5)
2. ADD R1, 1    (R1=6)
                           2. ADD R2, 1    (R2=6)
3. STORE x, R1  (x=6)
                           3. STORE x, R2  (x=6)

Final: x = 6 (হওয়া উচিত ছিল 7!)
```

---

## 💻 বাস্তব উদাহরণ

### Bank Account Problem

```c
// Shared account balance
int balance = 1000;

// Thread 1: Deposit
void deposit(int amount) {
    int temp = balance;       // Read
    temp = temp + amount;     // Modify
    balance = temp;           // Write
}

// Thread 2: Withdraw
void withdraw(int amount) {
    int temp = balance;       // Read
    temp = temp - amount;     // Modify
    balance = temp;           // Write
}

// Scenario:
// Thread 1: deposit(200)
// Thread 2: withdraw(100)
// Expected: 1000 + 200 - 100 = 1100

// Possible outcome with race condition:
// Final balance could be: 1200, 900, 1000, or 1100!
```

### Race Condition Timeline

```
Balance = 1000

Thread 1 (Deposit 200)           Thread 2 (Withdraw 100)
─────────────────────            ────────────────────────
1. temp1 = balance (1000)
                                 1. temp2 = balance (1000)
2. temp1 = 1000 + 200 (1200)
                                 2. temp2 = 1000 - 100 (900)
3. balance = temp1 (1200)
                                 3. balance = temp2 (900)

Final: balance = 900 ❌ (Lost deposit of 200!)
```

---

## 🔍 Race Condition চেনার উপায়

```
┌─────────────────────────────────────────────────────────────┐
│  Race Condition এর সম্ভাবনা আছে যদি:                        │
│                                                             │
│  1. Shared data থাকে (global variable, heap memory)        │
│  2. Multiple threads/processes access করে                  │
│  3. At least একটি write operation থাকে                    │
│  4. Synchronization না থাকে                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛡️ Race Condition প্রতিরোধ

### ১. Mutual Exclusion (Locks)

```c
pthread_mutex_t lock;

void deposit(int amount) {
    pthread_mutex_lock(&lock);     // 🔒 Lock
    
    int temp = balance;
    temp = temp + amount;
    balance = temp;
    
    pthread_mutex_unlock(&lock);   // 🔓 Unlock
}
```

### ২. Atomic Operations

```c
#include <stdatomic.h>

atomic_int balance = 1000;

void deposit(int amount) {
    atomic_fetch_add(&balance, amount);  // একটাই atomic instruction
}
```

### ৩. Semaphores

```c
sem_t sem;
sem_init(&sem, 0, 1);  // Binary semaphore

void deposit(int amount) {
    sem_wait(&sem);     // P operation
    
    balance += amount;
    
    sem_post(&sem);     // V operation
}
```

---

## 📊 Read-Write Race Conditions

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Read-Read:   Safe ✓ (কোনো সমস্যা নেই)                     │
│                                                             │
│  Read-Write:  RACE! ✗ (Reader পুরনো/নতুন data পেতে পারে)   │
│                                                             │
│  Write-Write: RACE! ✗ (একজনের write হারিয়ে যেতে পারে)     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 Data Race vs Race Condition

```
Data Race:
• দুই থ্রেড একই memory access করছে
• একটি write
• No synchronization
• Technical term

Race Condition:
• Program behavior depends on timing
• May or may not involve data race
• Broader term
• Can occur even with synchronization (logic error)
```

---

## 💡 ক্লাসিক উদাহরণ: Counter

### Without Synchronization (Wrong)

```c
#include <pthread.h>
#include <stdio.h>

int counter = 0;

void* increment(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        counter++;  // Race condition!
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    
    printf("Counter: %d\n", counter);
    // Expected: 2000000
    // Actual: ~1500000 to ~2000000 (unpredictable!)
    return 0;
}
```

### With Synchronization (Correct)

```c
#include <pthread.h>
#include <stdio.h>

int counter = 0;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void* increment(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&lock);
        counter++;
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    
    printf("Counter: %d\n", counter);
    // Always: 2000000 ✓
    return 0;
}
```

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Race Condition কেন Debugging কঠিন?
**উত্তর:** 
- Non-deterministic (প্রতিবার আলাদা ফলাফল)
- Timing dependent
- Debugging করলে timing বদলায়, তাই bug নাও দেখা যেতে পারে (Heisenbug)

**প্রশ্ন ২:** Race Condition সমাধানের উপায় কি কি?
**উত্তর:**
1. Mutex/Lock
2. Semaphore
3. Atomic operations
4. Thread-safe data structures

**প্রশ্ন ৩:** Critical Section ও Race Condition এর সম্পর্ক কি?
**উত্তর:** Race Condition প্রতিরোধ করতে Critical Section চিহ্নিত করে সেখানে mutual exclusion প্রয়োগ করতে হয়।

---
**পূর্ববর্তী:** [Critical Section](01-critical-section.md)  
**পরবর্তী:** [Mutex ও Semaphore](03-mutex-semaphore.md)
