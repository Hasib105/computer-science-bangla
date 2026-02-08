# CUDA ও প্যারালেল প্রোগ্রামিং
## Chapter 18: CUDA and Parallel Programming

---

## 📌 CUDA কী?

CUDA (Compute Unified Device Architecture) হলো NVIDIA এর parallel computing platform এবং programming model।

```
┌─────────────────────────────────────────────────────────────┐
│                    CUDA Programming Model                    │
│                                                              │
│  ┌──────────────┐        ┌──────────────────────────────┐  │
│  │     Host     │        │          Device (GPU)         │  │
│  │    (CPU)     │        │                               │  │
│  │              │        │  ┌──────────────────────────┐ │  │
│  │  ┌────────┐  │        │  │      Grid                 │ │  │
│  │  │ C/C++  │  │ ─────▶ │  │  ┌──────┐ ┌──────┐      │ │  │
│  │  │ Code   │  │        │  │  │Block │ │Block │ ...  │ │  │
│  │  └────────┘  │        │  │  └──────┘ └──────┘      │ │  │
│  │              │        │  └──────────────────────────┘ │  │
│  └──────────────┘        └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 CUDA Programming Basics

### Hello World (Vector Addition)

```cuda
// Kernel definition
__global__ void vectorAdd(float *a, float *b, float *c, int n) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;
    if (i < n) {
        c[i] = a[i] + b[i];
    }
}

// Host code
int main() {
    int n = 1000000;
    size_t size = n * sizeof(float);
    
    // Allocate host memory
    float *h_a = (float*)malloc(size);
    float *h_b = (float*)malloc(size);
    float *h_c = (float*)malloc(size);
    
    // Initialize vectors
    for (int i = 0; i < n; i++) {
        h_a[i] = i;
        h_b[i] = i * 2;
    }
    
    // Allocate device memory
    float *d_a, *d_b, *d_c;
    cudaMalloc(&d_a, size);
    cudaMalloc(&d_b, size);
    cudaMalloc(&d_c, size);
    
    // Copy to device
    cudaMemcpy(d_a, h_a, size, cudaMemcpyHostToDevice);
    cudaMemcpy(d_b, h_b, size, cudaMemcpyHostToDevice);
    
    // Launch kernel
    int threadsPerBlock = 256;
    int blocksPerGrid = (n + threadsPerBlock - 1) / threadsPerBlock;
    vectorAdd<<<blocksPerGrid, threadsPerBlock>>>(d_a, d_b, d_c, n);
    
    // Copy result back
    cudaMemcpy(h_c, d_c, size, cudaMemcpyDeviceToHost);
    
    // Cleanup
    cudaFree(d_a); cudaFree(d_b); cudaFree(d_c);
    free(h_a); free(h_b); free(h_c);
    
    return 0;
}
```

---

## 🧵 Thread Hierarchy

### Thread Indexing

```cuda
// 1D Grid, 1D Block
int idx = blockIdx.x * blockDim.x + threadIdx.x;

// 2D Grid, 2D Block
int x = blockIdx.x * blockDim.x + threadIdx.x;
int y = blockIdx.y * blockDim.y + threadIdx.y;
int idx = y * width + x;

// 3D Grid, 3D Block
int x = blockIdx.x * blockDim.x + threadIdx.x;
int y = blockIdx.y * blockDim.y + threadIdx.y;
int z = blockIdx.z * blockDim.z + threadIdx.z;
```

### Built-in Variables

```cuda
// Thread index within block
threadIdx.x, threadIdx.y, threadIdx.z

// Block index within grid
blockIdx.x, blockIdx.y, blockIdx.z

// Block dimensions
blockDim.x, blockDim.y, blockDim.z

// Grid dimensions
gridDim.x, gridDim.y, gridDim.z
```

---

## 💾 Memory Types

### Memory Qualifiers

```cuda
// Global memory (device)
__device__ float globalVar;

// Constant memory (read-only, cached)
__constant__ float constArray[256];

// Shared memory (per block)
__shared__ float sharedArray[256];

// Local/Register (per thread)
float localVar;
```

### Memory Comparison

| Type | Scope | Lifetime | Speed |
|------|-------|----------|-------|
| Register | Thread | Thread | Fastest |
| Local | Thread | Thread | Slow (DRAM) |
| Shared | Block | Block | Fast (~5 cycles) |
| Global | Grid | Application | Slow (~400 cycles) |
| Constant | Grid | Application | Fast (cached) |
| Texture | Grid | Application | Fast (cached) |

---

## 🔄 Synchronization

### Block-level Synchronization

```cuda
__global__ void kernel() {
    __shared__ float sharedData[256];
    
    // Load data
    sharedData[threadIdx.x] = globalData[blockIdx.x * blockDim.x + threadIdx.x];
    
    // Synchronize all threads in block
    __syncthreads();
    
    // Now safe to use sharedData from other threads
    float sum = sharedData[threadIdx.x] + sharedData[threadIdx.x + 1];
}
```

### Atomic Operations

```cuda
// Atomic add
atomicAdd(&sum, value);

// Atomic compare and swap
atomicCAS(&value, compare, newValue);

// Other atomics
atomicMin(), atomicMax(), atomicAnd(), atomicOr(), atomicXor()
```

---

## 📊 Optimization Techniques

### ১. Memory Coalescing

```cuda
// Good - Coalesced access
float val = input[threadIdx.x + blockIdx.x * blockDim.x];

// Bad - Strided access
float val = input[threadIdx.x * stride];

// Visualization:
// Coalesced:     T0─[0] T1─[1] T2─[2] T3─[3]  → 1 transaction
// Non-coalesced: T0─[0] T1─[4] T2─[8] T3─[12] → 4 transactions
```

### ২. Shared Memory Usage

```cuda
__global__ void matrixMul(float *A, float *B, float *C, int N) {
    __shared__ float As[TILE_SIZE][TILE_SIZE];
    __shared__ float Bs[TILE_SIZE][TILE_SIZE];
    
    int row = blockIdx.y * TILE_SIZE + threadIdx.y;
    int col = blockIdx.x * TILE_SIZE + threadIdx.x;
    
    float sum = 0.0f;
    
    for (int tile = 0; tile < N / TILE_SIZE; tile++) {
        // Load tile to shared memory
        As[threadIdx.y][threadIdx.x] = A[row * N + tile * TILE_SIZE + threadIdx.x];
        Bs[threadIdx.y][threadIdx.x] = B[(tile * TILE_SIZE + threadIdx.y) * N + col];
        
        __syncthreads();
        
        // Compute using shared memory
        for (int k = 0; k < TILE_SIZE; k++) {
            sum += As[threadIdx.y][k] * Bs[k][threadIdx.x];
        }
        
        __syncthreads();
    }
    
    C[row * N + col] = sum;
}
```

### ৩. Avoiding Bank Conflicts

```cuda
// Bank conflict (all threads access same bank)
__shared__ float data[32];
float val = data[threadIdx.x * 32];  // Bad!

// No bank conflict
float val = data[threadIdx.x];  // Good!

// Padding to avoid conflict
__shared__ float data[32][33];  // Extra column
float val = data[threadIdx.x][threadIdx.y];
```

### ৪. Occupancy Optimization

```cuda
// Check occupancy
int blockSize = 256;
int minGridSize;
int gridSize;

cudaOccupancyMaxPotentialBlockSize(&minGridSize, &blockSize, kernel, 0, 0);

// Launch with optimal configuration
kernel<<<gridSize, blockSize>>>(args);
```

---

## 🔀 Streams and Concurrency

### CUDA Streams

```cuda
cudaStream_t stream1, stream2;
cudaStreamCreate(&stream1);
cudaStreamCreate(&stream2);

// Async operations in different streams
cudaMemcpyAsync(d_a, h_a, size, cudaMemcpyHostToDevice, stream1);
cudaMemcpyAsync(d_b, h_b, size, cudaMemcpyHostToDevice, stream2);

kernel<<<grid, block, 0, stream1>>>(d_a);
kernel<<<grid, block, 0, stream2>>>(d_b);

cudaStreamSynchronize(stream1);
cudaStreamSynchronize(stream2);

cudaStreamDestroy(stream1);
cudaStreamDestroy(stream2);
```

### Concurrency Diagram

```
Stream 1: [H2D copy][Kernel A][D2H copy]
Stream 2:          [H2D copy][Kernel B][D2H copy]
          ─────────────────────────────────────────▶ Time

Overlapping operations = Better GPU utilization
```

---

## 🧮 cuBLAS ও cuDNN

### cuBLAS (Basic Linear Algebra)

```cuda
#include <cublas_v2.h>

cublasHandle_t handle;
cublasCreate(&handle);

// Matrix multiplication: C = α*A*B + β*C
cublasSgemm(handle,
    CUBLAS_OP_N, CUBLAS_OP_N,
    m, n, k,
    &alpha,
    A, lda,
    B, ldb,
    &beta,
    C, ldc);

cublasDestroy(handle);
```

### cuDNN (Deep Neural Networks)

```cuda
#include <cudnn.h>

cudnnHandle_t cudnn;
cudnnCreate(&cudnn);

// Convolution forward
cudnnConvolutionForward(cudnn,
    &alpha,
    inputDesc, input,
    filterDesc, filter,
    convDesc,
    algo,
    workspace, workspaceSize,
    &beta,
    outputDesc, output);
```

---

## 📈 Performance Analysis

### NVIDIA Nsight

```bash
# Profile application
nsys profile ./myapp

# Compute analysis
ncu ./myapp
```

### Key Metrics

```
- SM Occupancy
- Memory Throughput
- Compute Throughput
- Warp Execution Efficiency
- Shared Memory Efficiency
```

---

## 🌐 OpenCL Alternative

```c
// OpenCL Kernel
__kernel void vectorAdd(__global float *a,
                        __global float *b,
                        __global float *c,
                        int n) {
    int i = get_global_id(0);
    if (i < n) {
        c[i] = a[i] + b[i];
    }
}
```

### CUDA vs OpenCL

| Feature | CUDA | OpenCL |
|---------|------|--------|
| Vendor | NVIDIA only | Cross-platform |
| Performance | Optimized | Good |
| Ecosystem | Rich | Growing |
| Learning Curve | Easier | Harder |

---

## ✏️ অনুশীলনী

1. Vector addition kernel লিখুন এবং thread indexing ব্যাখ্যা করুন।
2. Shared memory ব্যবহার করে matrix transpose লিখুন।
3. Memory coalescing optimization ব্যাখ্যা করুন।

---

## 📚 পরবর্তী অধ্যায়
[ইনপুট/আউটপুট সিস্টেম](../07-IO-ও-বাস-সিস্টেম/01-ইনপুট-আউটপুট-সিস্টেম.md)
