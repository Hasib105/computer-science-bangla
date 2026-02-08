# Memory Mapped Files

## 📌 Memory Mapped Files কি?

**Memory Mapped Files** হলো এমন technique যেখানে disk file কে virtual memory address space এ map করা হয়। File access তখন memory access এর মতোই হয়।

```
┌─────────────────────────────────────────────────────────────┐
│                  Memory Mapped I/O                          │
│                                                             │
│   Traditional File I/O:                                     │
│   ┌─────────┐    read()    ┌───────────┐   ┌──────────┐   │
│   │ Process │ ──────────► │OS Buffer  │◄──│   File   │   │
│   │ Buffer  │◄───────────  │(Kernel)   │   │  (Disk)  │   │
│   └─────────┘    copy      └───────────┘   └──────────┘   │
│                                                             │
│   Memory Mapped:                                            │
│   ┌──────────────────────────────────────┐  ┌──────────┐  │
│   │     Process Virtual Address Space     │  │          │  │
│   │  ┌─────────────────────────────────┐ │  │   File   │  │
│   │  │   Mapped Region (file data)     │◄┼──│  (Disk)  │  │
│   │  │   Direct access like memory!    │ │  │          │  │
│   │  └─────────────────────────────────┘ │  └──────────┘  │
│   └──────────────────────────────────────┘                 │
│                                                             │
│   No copy needed! Direct access to file content            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 💡 কিভাবে কাজ করে?

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. mmap() system call                                      │
│     File এর একটি portion virtual address এ map করে        │
│                                                             │
│  2. Initial Access                                          │
│     Page fault হয় (page not in memory)                    │
│     OS file থেকে page load করে                            │
│                                                             │
│  3. Subsequent Access                                       │
│     Normal memory access (no system call!)                  │
│     Fast, no context switch                                 │
│                                                             │
│  4. Modifications                                           │
│     Dirty pages eventually written back to file             │
│     Or on munmap()/msync()                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 mmap() System Call

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);

// Parameters:
// addr   - preferred starting address (NULL = let OS choose)
// length - how many bytes to map
// prot   - protection (PROT_READ, PROT_WRITE, PROT_EXEC)
// flags  - MAP_SHARED or MAP_PRIVATE
// fd     - file descriptor
// offset - starting point in file
```

### Example: Reading a File

```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>

int main() {
    // Open file
    int fd = open("data.txt", O_RDONLY);
    
    // Get file size
    struct stat sb;
    fstat(fd, &sb);
    
    // Map file to memory
    char *mapped = mmap(NULL, sb.st_size, PROT_READ, 
                        MAP_PRIVATE, fd, 0);
    
    // Now access file like memory!
    printf("First char: %c\n", mapped[0]);
    printf("Last char: %c\n", mapped[sb.st_size - 1]);
    
    // Unmap when done
    munmap(mapped, sb.st_size);
    close(fd);
    
    return 0;
}
```

---

## 📊 MAP_SHARED vs MAP_PRIVATE

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  MAP_SHARED:                                                │
│  • Changes are visible to other processes                   │
│  • Changes are written back to file                         │
│  • Good for IPC (Inter-Process Communication)               │
│                                                             │
│  Process A ──────┐                                         │
│                  ▼                                          │
│              ┌──────────────┐                              │
│              │ Shared Page  │ ◄──── File                   │
│              └──────────────┘                              │
│                  ▲                                          │
│  Process B ──────┘                                         │
│                                                             │
│  MAP_PRIVATE:                                               │
│  • Copy-on-Write                                            │
│  • Changes are private to process                           │
│  • Changes NOT written to file                              │
│                                                             │
│  Process A:  [Page] ──write──► [Private Copy]              │
│  File:       [Page] (unchanged)                             │
│  Process B:  [Page] (original)                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 💡 Use Cases

### ১. Large File Processing

```c
// Traditional: Read entire file to buffer
char *buffer = malloc(file_size);  // কয়েক GB!
read(fd, buffer, file_size);       // Slow!

// Memory mapped: Lazy loading
char *mapped = mmap(...);
// শুধু accessed pages load হয়
// Virtual memory system handles it!
```

### ২. Shared Memory (IPC)

```c
// Process 1
int fd = shm_open("/shared_mem", O_CREAT | O_RDWR, 0666);
ftruncate(fd, SIZE);
int *shared = mmap(NULL, SIZE, PROT_READ | PROT_WRITE,
                   MAP_SHARED, fd, 0);
shared[0] = 42;  // Write

// Process 2
int fd = shm_open("/shared_mem", O_RDONLY, 0666);
int *shared = mmap(NULL, SIZE, PROT_READ,
                   MAP_SHARED, fd, 0);
printf("%d\n", shared[0]);  // Read: 42
```

### ৩. Executables ও Libraries

```
যখন program run হয়:

executable file disk এ থাকে
mmap() দিয়ে memory তে map হয়
শুধু needed parts load হয় (demand paging)

Same for shared libraries (.so, .dll)
Multiple processes share same physical pages!

┌──────────────────────────────────────────────────────────┐
│  Process A     Process B     Process C                   │
│     │              │             │                        │
│     │              │             │                        │
│     ▼              ▼             ▼                        │
│  ┌───────────────────────────────────────────┐           │
│  │           libc.so (read-only)             │           │
│  │           One copy in memory!             │           │
│  └───────────────────────────────────────────┘           │
└──────────────────────────────────────────────────────────┘
```

---

## 📊 Advantages & Disadvantages

| সুবিধা | অসুবিধা |
|--------|---------|
| No buffer copy | Complex error handling |
| Lazy loading | File size changes tricky |
| Automatic page management | Memory limit for large files |
| Easy sharing between processes | Platform differences |
| Faster than read()/write() | |

---

## 🔒 Memory Mapped I/O (Device)

```
┌─────────────────────────────────────────────────────────────┐
│           Memory Mapped Device I/O                          │
│                                                             │
│   Different from memory mapped files!                       │
│                                                             │
│   Device registers mapped to memory addresses               │
│                                                             │
│   Address Space:                                            │
│   ┌──────────────────────────┐                             │
│   │        RAM               │  0x00000000                 │
│   ├──────────────────────────┤                             │
│   │   Video Memory           │  0xA0000000                 │
│   ├──────────────────────────┤                             │
│   │   Device Registers       │  0xFFFF0000                 │
│   │   (Keyboard, Disk, etc)  │                             │
│   └──────────────────────────┘                             │
│                                                             │
│   Write to 0xA0000000 → appears on screen!                 │
│   Read from 0xFFFF0000 → keyboard input                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** Memory mapped files traditional file I/O থেকে দ্রুত কেন?
**উত্তর:**
- No buffer copy (kernel ↔ user space)
- No system call for each access
- Virtual memory system handles page management

**প্রশ্ন ২:** MAP_SHARED কখন ব্যবহার করবেন?
**উত্তর:**
- File এ changes write করতে চাইলে
- Multiple processes এ share করতে চাইলে
- IPC এর জন্য

**প্রশ্ন ৩:** Executable files mmap() ব্যবহার করে কেন?
**উত্তর:** Code pages demand paging এ load হয়, multiple processes same library share করতে পারে, memory efficient।

---
**পূর্ববর্তী:** [Thrashing](04-thrashing.md)  
**পরবর্তী ফোল্ডার:** [File System](../08-File-System/README.md)
