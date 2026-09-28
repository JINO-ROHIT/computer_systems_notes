# Basics of a computer system


- **Information = bits + context.** Everything is ultimately stored, moved, and processed as bits. The *same* bit pattern can mean an integer, a floating-point value, text, or a machine instruction depending on how it is interpreted.


## Hardware

now lets say we have a hello.c program, what components do we need to run this program? do remember this is just bits.

### Main components

| Component | Purpose |
|---|---|
| **Bus** | Electrical paths carrying fixed-size chunks of bytes (**words**) between components. Typical word sizes: 4 bytes (32-bit) or 8 bytes (64-bit). |
| **I/O devices** | Connect the system to the outside world: keyboard, mouse, display, disk, network. They connect through a controller or adapter. |
| **Main memory (DRAM)** | Temporary storage for executing programs and their data. it logically a byte-addressed linear array starting at address 0. |
| **CPU / processor** | Fetches and executes instructions stored in main memory. |
| **PC (program counter)** | Register holding the address of the next instruction. |
| **Register file** | Small, very fast collection of word-sized CPU storage locations. |
| **ALU** | Arithmetic/Logic Unit computes arithmetic and address values. |

### Instruction cycle

The CPU repeatedly **fetches** the instruction at the PC, interprets its bits, performs its operation, then updates the PC.

- **Load:** memory -> register
- **Store:** register -> memory
- **Operate:** registers -> ALU -> register
- **Jump:** instruction-supplied address -> PC


## 4. Caches and the memory hierarchy

Data movement costs time. Bigger storage tends to be slower faster storage is more expensive per byte. Caches bridge the growing CPU–memory speed gap.

```text
smaller / faster / costlier per byte
Registers (L0)
L1 cache
L2 cache
L3 cache
Main memory (DRAM)
Local disk / SSD
Remote storage
larger / slower / cheaper per byte
```

- A **cache** is smaller, faster storage holding copies of data likely needed soon.
- **Locality** is the tendency of programs to access code and data in localized regions it is why caching works.
- Each level acts as a cache for the level below it (registers cache L1 L1 caches L2 … DRAM caches disk).
- Cache-aware data layout and access order can improve program performance dramatically.

## 5. Operating system: hardware abstractions

The OS is software between applications and hardware. Applications use it rather than accessing hardware directly.

Its purposes are to:

1. **Protect** hardware from faulty or malicious applications.
2. Provide simple, uniform abstractions over complicated and varied hardware.

| OS abstraction | Hides / represents |
|---|---|
| **Process** | A running program abstracts CPU, memory, and I/O devices. |
| **Virtual memory** | A process’s memory abstracts main memory and disk. |
| **File** | An I/O device. |

### Processes and context switching

- A **process** is the OS abstraction for a running program. Multiple processes appear to have exclusive use of CPU, memory, and I/O.
- On one CPU, **concurrency** is normally achieved by interleaving process instructions, not by literally executing them at the same instant.
- A process’s **context** includes the PC, register values, and memory state needed to resume it.
- A **context switch** saves the current process context, restores another’s context, then transfers control. The new process resumes exactly where it stopped.
- The **kernel** is OS code/data structures resident in memory that manage processes. It is **not** a separate process. A **system call** transfers control from an application to the kernel to request an OS service.

### Threads

- A process may contain multiple **threads**: execution flows sharing that process’s code and global data.
- Threads enable concurrency, share data more easily than separate processes, and can exploit multiple processors.

### Virtual memory

Each process sees a private, uniform **virtual address space**, creating the illusion of exclusive main memory.

```text
High addresses
Kernel virtual memory      (inaccessible directly to user code)
User stack                 (grows/shrinks with function calls)
Memory-mapped shared libs
Run-time heap              (grows/shrinks via malloc/free)
Read/write data            (e.g., globals)
Read-only code and data
0 / program start
Low addresses
```

- Hardware translates every virtual address OS and hardware cooperate to make this work.
- Conceptually, virtual-memory contents are backed by disk and main memory acts as a cache for them.


## 7. Amdahl’s Law

Use Amdahl’s Law to assess whether optimizing one component will meaningfully speed up the whole system.

- `α` = fraction of original execution time spent in the part improved
- `k` = speedup factor of that part
- `T_old`, `T_new` = original and new total execution times

```text
T_new = T_old × [(1 − α) + α/k]

Overall speedup S = T_old / T_new = 1 / [(1 − α) + α/k]
```

- If `α = 0.6` and `k = 3`, then `S = 1 / (0.4 + 0.6/3) = 1.67×`.
- Even with an infinitely fast optimized part (`k -> ∞`), the maximum speedup is `1 / (1 − α)`.
- **Lesson:** optimize the parts consuming a large fraction of total time optimizing a small bottleneck has a hard overall limit.

## 8. Concurrency and parallelism

- **Concurrency:** multiple activities make progress during overlapping time periods.
- **Parallelism:** using concurrency specifically to make a system run faster.

| Level | Meaning |
|---|---|
| **Thread-level** | Multiple processes/threads run concurrently with multiple cores, they can actually run in parallel. |
| **Instruction-level (ILP)** | CPU overlaps/executes multiple instructions pipelining divides instruction execution into stages. **Superscalar** CPUs sustain more than one instruction per clock cycle. |
| **SIMD** | One instruction performs the same operation on multiple data elements at once common for image, audio, and video workloads. |

- A **multicore** chip contains multiple CPU cores, often with private lower caches and shared higher caches/memory interface.
- **Hyperthreading / simultaneous multithreading** lets one physical core maintain multiple execution contexts, using otherwise idle resources when one thread stalls.
- Multiple cores speed up one application only when its work is expressed as independently executable threads.