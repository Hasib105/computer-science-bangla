# I/O Methods (I/O পদ্ধতি)

## 📌 তিনটি প্রধান পদ্ধতি

```
┌─────────────────────────────────────────────────────────────┐
│                     I/O Methods                             │
│                                                             │
│   1. Programmed I/O (Polling)                               │
│      CPU continuously checks device status                  │
│                                                             │
│   2. Interrupt-Driven I/O                                   │
│      Device signals CPU when ready                          │
│                                                             │
│   3. Direct Memory Access (DMA)                             │
│      Device transfers data directly to memory               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Programmed I/O (Polling)

```
┌─────────────────────────────────────────────────────────────┐
│                    Programmed I/O                           │
│                                                             │
│   CPU polls device status in a loop                         │
│                                                             │
│   ┌─────┐     Check status    ┌────────────┐               │
│   │     │ ───────────────────►│            │               │
│   │     │◄─────────────────── │            │               │
│   │     │     Not ready       │  Device    │               │
│   │ CPU │                     │ Controller │               │
│   │     │     Check status    │            │               │
│   │     │ ───────────────────►│            │               │
│   │     │◄─────────────────── │            │               │
│   │     │     Ready!          │            │               │
│   │     │ ───────────────────►│            │               │
│   │     │     Transfer data   │            │               │
│   └─────┘                     └────────────┘               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Code Example

```c
// Polling example - reading from device
void read_device(char *buffer, int count) {
    int i;
    for (i = 0; i < count; i++) {
        // Poll: wait for device to be ready
        while ((inb(STATUS_PORT) & READY_BIT) == 0) {
            // Busy waiting - CPU does nothing useful
        }
        
        // Device ready, read data
        buffer[i] = inb(DATA_PORT);
    }
}

// CPU is 100% busy during polling!
```

### সুবিধা ও অসুবিধা

```
✓ সুবিধা:
• Simple implementation
• No interrupt overhead
• Good for very fast devices

✗ অসুবিধা:
• CPU wastes time in busy waiting
• Cannot do other work
• Inefficient for slow devices
```

---

## 2️⃣ Interrupt-Driven I/O

```
┌─────────────────────────────────────────────────────────────┐
│                  Interrupt-Driven I/O                       │
│                                                             │
│   Step 1: CPU initiates I/O                                 │
│   ┌─────┐     Start I/O       ┌────────────┐               │
│   │ CPU │ ───────────────────►│  Device    │               │
│   └─────┘                     └────────────┘               │
│                                                             │
│   Step 2: CPU does other work                               │
│   ┌─────┐                                                   │
│   │ CPU │  Executing other processes...                    │
│   └─────┘                                                   │
│                                                             │
│   Step 3: Device interrupts when ready                      │
│   ┌─────┐     INTERRUPT!      ┌────────────┐               │
│   │ CPU │◄─────────────────── │  Device    │               │
│   └─────┘                     └────────────┘               │
│                                                             │
│   Step 4: CPU handles interrupt                             │
│   ┌─────┐     Transfer data   ┌────────────┐               │
│   │ CPU │◄═══════════════════►│  Device    │               │
│   └─────┘     (in ISR)        └────────────┘               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Interrupt Handling Process

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   1. Device raises interrupt signal                         │
│                    │                                        │
│                    ▼                                        │
│   2. CPU finishes current instruction                       │
│                    │                                        │
│                    ▼                                        │
│   3. CPU saves current state (PC, registers)                │
│                    │                                        │
│                    ▼                                        │
│   4. CPU looks up ISR address in Interrupt Vector Table     │
│                    │                                        │
│                    ▼                                        │
│   5. CPU jumps to Interrupt Service Routine (ISR)           │
│                    │                                        │
│                    ▼                                        │
│   6. ISR handles the interrupt (read/write data)            │
│                    │                                        │
│                    ▼                                        │
│   7. ISR sends EOI (End of Interrupt) to controller         │
│                    │                                        │
│                    ▼                                        │
│   8. CPU restores saved state                               │
│                    │                                        │
│                    ▼                                        │
│   9. CPU resumes interrupted program                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Code Example

```c
// Interrupt handler for keyboard
void keyboard_isr() {
    // Read the scancode from keyboard
    unsigned char scancode = inb(KEYBOARD_DATA_PORT);
    
    // Convert to ASCII and add to buffer
    char c = scancode_to_ascii(scancode);
    keyboard_buffer[kb_index++] = c;
    
    // Signal end of interrupt
    outb(PIC_EOI, PIC_COMMAND_PORT);
}

// Main program continues while waiting for interrupt
void main() {
    // Register interrupt handler
    register_handler(KEYBOARD_IRQ, keyboard_isr);
    
    // Do other work - NOT busy waiting
    while (1) {
        do_useful_work();
    }
}
```

### সুবিধা ও অসুবিধা

```
✓ সুবিধা:
• CPU can do other work while waiting
• Efficient for slow devices
• Better CPU utilization

✗ অসুবিধা:
• Interrupt overhead (context save/restore)
• Still CPU involved in data transfer
• Can overwhelm CPU with many interrupts
```

---

## 3️⃣ Direct Memory Access (DMA)

```
┌─────────────────────────────────────────────────────────────┐
│                         DMA                                 │
│                                                             │
│   ┌──────────────── System Bus ─────────────────┐          │
│   │                                              │          │
│   ▼                                              ▼          │
│ ┌─────┐                                      ┌────────┐    │
│ │ CPU │                                      │ Memory │    │
│ └─────┘                                      └────────┘    │
│    │                                              ▲         │
│    │ 1. Setup                                     │         │
│    ▼    DMA                                       │         │
│ ┌─────────────┐                                   │         │
│ │    DMA      │                                   │         │
│ │ Controller  │───────────────────────────────────┘         │
│ └─────────────┘    3. Transfer data directly               │
│    │    ▲                                                   │
│    │    │ 4. Interrupt                                      │
│    │    │    when done                                      │
│    ▼    │                                                   │
│ ┌────────────┐                                             │
│ │   Device   │                                             │
│ │   (Disk)   │                                             │
│ └────────────┘                                             │
│        │                                                    │
│        │ 2. Read data                                       │
│        ▼                                                    │
│    [Disk sectors]                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### DMA Steps

```
Step 1: CPU programs DMA controller
        - Source address (device)
        - Destination address (memory)
        - Number of bytes
        - Direction (read/write)

Step 2: CPU starts DMA transfer
        CPU is now FREE to do other work!

Step 3: DMA controller transfers data
        - Reads from device
        - Writes to memory
        - One byte/word at a time
        - "Steals" bus cycles from CPU

Step 4: DMA sends interrupt when complete
        CPU only involved at start and end!
```

### Code Example

```c
// Setting up DMA transfer
void dma_read_disk(void *dest, int sector, int count) {
    // 1. Set up DMA controller
    outb(DMA_COMMAND, DMA_READ_COMMAND);
    
    // 2. Set memory address
    outl(DMA_ADDR_REG, (uint32_t)dest);
    
    // 3. Set transfer count
    outl(DMA_COUNT_REG, count * SECTOR_SIZE);
    
    // 4. Set up disk controller
    outb(DISK_SECTOR_REG, sector);
    outb(DISK_COMMAND_REG, DISK_READ);
    
    // 5. Start DMA
    outb(DMA_START_REG, 1);
    
    // CPU is now free! Will get interrupt when done
}

// Interrupt handler
void dma_complete_isr() {
    // Check status
    int status = inb(DMA_STATUS_REG);
    if (status & DMA_ERROR) {
        handle_error();
    }
    
    // Signal process that data is ready
    wake_up_process(waiting_process);
    
    // End of interrupt
    outb(PIC_EOI, PIC_COMMAND);
}
```

---

## 📊 Method Comparison

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Data Transfer Rate vs CPU Overhead                        │
│                                                             │
│   CPU                                                       │
│   Overhead                                                  │
│      │                                                      │
│   High├─ ● Programmed I/O                                  │
│      │                                                      │
│      │                                                      │
│   Med ├─────────── ● Interrupt I/O                         │
│      │                                                      │
│      │                                                      │
│   Low ├─────────────────────────── ● DMA                   │
│      │                                                      │
│      └──────────────────────────────────────── Device Speed │
│        Slow                              Fast               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

| Feature | Programmed I/O | Interrupt I/O | DMA |
|---------|----------------|---------------|-----|
| CPU during transfer | 100% busy | Busy in ISR | Free |
| Best for | Very fast devices | Medium speed | Block transfer |
| Complexity | Simple | Medium | Complex |
| Hardware needed | None | Interrupt controller | DMA controller |

---

## ❓ গুরুত্বপূর্ণ প্রশ্ন

**প্রশ্ন ১:** DMA কেন দ্রুত?
**উত্তর:** CPU involvement কম। CPU শুধু DMA setup করে, বাকি transfer DMA controller করে। CPU অন্য কাজ করতে পারে।

**প্রশ্ন ২:** কখন Polling ব্যবহার করবেন?
**উত্তর:**
- Very fast devices (data immediately available)
- Short polling loops
- Real-time systems where interrupt latency is issue

**প্রশ্ন ৩:** "Cycle stealing" কি?
**উত্তর:** DMA controller যখন bus ব্যবহার করে তখন CPU bus access করতে পারে না। DMA এইভাবে CPU এর bus cycles "steal" করে।

---
**পূর্ববর্তী:** [I/O Hardware](01-io-hardware.md)  
**পরবর্তী:** [Disk Scheduling](03-disk-scheduling.md)
