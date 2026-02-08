# GPU পরিচিতি
## Chapter 16: Introduction to GPU Architecture

---

## 📌 GPU কী?

GPU (Graphics Processing Unit) হলো বিশেষায়িত প্রসেসর যা মূলত গ্রাফিক্স রেন্ডারিং এবং parallel computing এর জন্য ডিজাইন করা।

```
┌─────────────────────────────────────────────────────────────┐
│                   CPU vs GPU                                 │
│                                                              │
│  CPU (Few powerful cores):                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                    │    │
│  │ │Core │ │Core │ │Core │ │Core │    ┌──────────┐    │    │
│  │ │  0  │ │  1  │ │  2  │ │  3  │    │  Cache   │    │    │
│  │ └─────┘ └─────┘ └─────┘ └─────┘    │  (Large) │    │    │
│  │    (Complex, Fast)                  └──────────┘    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  GPU (Many simple cores):                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐   │    │
│  │ │ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ │   │    │
│  │ └─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘   │    │
│  │ ┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐   │    │
│  │ │ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ ││ │   │    │
│  │ └─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘   │    │
│  │    (Thousands of simple cores)                      │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 CPU vs GPU তুলনা

| বৈশিষ্ট্য | CPU | GPU |
|----------|-----|-----|
| Cores | 4-64 | 1000-10000+ |
| Clock Speed | 3-5 GHz | 1-2 GHz |
| Cache | বড় (MB) | ছোট (KB per core) |
| Control Logic | জটিল | সহজ |
| Best for | Serial tasks | Parallel tasks |
| Power | 65-125W | 200-450W |
| Memory BW | 50-100 GB/s | 500-2000 GB/s |

---

## 🎮 GPU এর ইতিহাস

```
Timeline:

1999: NVIDIA GeForce 256 (First "GPU")
      - Transform & Lighting hardware

2001: Programmable Shaders
      - Vertex and Pixel shaders

2007: NVIDIA CUDA
      - General Purpose GPU (GPGPU)

2009: OpenCL
      - Open standard

2012: Kepler Architecture
      - Dynamic Parallelism

2020: Ampere Architecture
      - Tensor Cores for AI

2022: Ada Lovelace/Hopper
      - AI acceleration
```

---

## 🏗️ GPU Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      GPU                                     │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                 Graphics Processing Cluster (GPC)      │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐      │  │
│  │  │     SM      │ │     SM      │ │     SM      │ ...  │  │
│  │  │ (Streaming  │ │             │ │             │      │  │
│  │  │ Multiproc.) │ │             │ │             │      │  │
│  │  └─────────────┘ └─────────────┘ └─────────────┘      │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                       L2 Cache                         │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Memory Controllers (GDDR6/HBM)            │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 Streaming Multiprocessor (SM)

```
┌─────────────────────────────────────────────────────────────┐
│                  Streaming Multiprocessor (SM)               │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                 Instruction Cache                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           │
│  │ Warp    │ │ Warp    │ │ Warp    │ │ Warp    │           │
│  │Scheduler│ │Scheduler│ │Scheduler│ │Scheduler│           │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘           │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              CUDA Cores (FP32)                       │    │
│  │  ┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐...       │    │
│  │  │  ││  ││  ││  ││  ││  ││  ││  ││  ││  │  (64-128) │    │
│  │  └──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────┐ ┌─────────────────┐                   │
│  │  Tensor Cores   │ │    RT Cores     │                   │
│  │  (Matrix ops)   │ │  (Ray Tracing)  │                   │
│  └─────────────────┘ └─────────────────┘                   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Shared Memory / L1 Cache                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  Register File                       │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧵 Thread Hierarchy

### Thread Organization

```
Grid (Kernel launch)
  │
  ├── Block 0 ─────────────────────────────────────────────┐
  │   │                                                     │
  │   ├── Warp 0: Thread 0-31                              │
  │   ├── Warp 1: Thread 32-63                             │
  │   └── Warp N: ...                                       │
  │                                                         │
  ├── Block 1 ─────────────────────────────────────────────┤
  │   │                                                     │
  │   └── Warps...                                          │
  │                                                         │
  └── Block N                                               │
```

### Thread vs Block vs Grid

| Level | বর্ণনা |
|-------|--------|
| Thread | একক execution unit |
| Warp | 32 threads একসাথে execute |
| Block | Threads এর গ্রুপ (max 1024) |
| Grid | Blocks এর গ্রুপ |

---

## 💾 GPU Memory Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    GPU Memory Hierarchy                      │
│                                                              │
│  Per-Thread:                                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Registers (Fastest, ~1 cycle)                       │    │
│  │ Local Memory (Slowest, in DRAM)                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Per-Block:                                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Shared Memory (Fast, ~5 cycles)                     │    │
│  │ L1 Cache                                             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Per-Device:                                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ L2 Cache (~30 cycles)                               │    │
│  │ Global Memory - GDDR6/HBM (~400 cycles)             │    │
│  │ Constant Memory (Cached)                             │    │
│  │ Texture Memory (Cached, Spatial locality)           │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 GPU Applications

### Graphics
```
- 3D Rendering
- Video Games
- CAD/CAM
- Video Processing
```

### General Purpose (GPGPU)
```
- Scientific Computing
- Machine Learning / AI
- Cryptocurrency Mining
- Physics Simulation
- Medical Imaging
```

### AI/ML Workloads
```
- Deep Learning Training
- Inference
- Natural Language Processing
- Computer Vision
```

---

## 📊 Modern GPU Specifications

### NVIDIA RTX 4090
```
CUDA Cores: 16,384
Tensor Cores: 512
RT Cores: 128
Memory: 24 GB GDDR6X
Memory BW: 1 TB/s
TDP: 450W
```

### NVIDIA H100 (Data Center)
```
CUDA Cores: 16,896
Tensor Cores: 528
Memory: 80 GB HBM3
Memory BW: 3.35 TB/s
TDP: 700W
FP16 Performance: 1979 TFLOPS
```

### AMD Radeon RX 7900 XTX
```
Stream Processors: 6,144
Memory: 24 GB GDDR6
Memory BW: 960 GB/s
TDP: 355W
```

---

## ⚡ GPU Compute Power

### FLOPS Calculation
```
FLOPS = Cores × Clock × Ops_per_cycle

Example: RTX 4090
FLOPS = 16384 × 2.52 GHz × 2
      = 82.6 TFLOPS (FP32)
```

### Tensor Core Performance
```
Matrix operations: 4x4 matrix multiply-accumulate per cycle
Much higher throughput for AI workloads
```

---

## ✏️ অনুশীলনী

1. CPU ও GPU এর মূল পার্থক্য কী?
2. Streaming Multiprocessor এর গঠন ব্যাখ্যা করুন।
3. GPU কোন ধরনের কাজে বেশি efficient এবং কেন?

---

## 📚 পরবর্তী অধ্যায়
[GPU আর্কিটেকচার বিস্তারিত](./02-GPU-আর্কিটেকচার-বিস্তারিত.md)
