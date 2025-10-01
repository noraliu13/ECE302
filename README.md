# ECE 344 – Operating Systems  

Lecture 1

## 📌 Core OS Themes  

### 1. Virtualization  
Making **one resource look like many**.  
- Example: One CPU → many processes appear to run “at once.”  
- Example: One memory → each process sees its own private memory.  

### 2. Concurrency  
Multiple things happening at once (real or apparent).  
- Example: Two programs running on one CPU, switching back and forth.  

### 3. Persistence  
Information survives **power loss**.  
- OS manages disks & filesystems → ensures data is not lost when system reboots.  



### 💡 Computer Science Trick: **Indirection**  
- Problems can often be solved with another **layer of abstraction**.  
- OS rarely gives you hardware directly — instead, it gives you **handles**:  
  - Process → handle to CPU + memory  
  - File descriptor → handle to file  
  - Virtual address → handle to physical memory  



## 📌 First Abstraction: The Process  

### Program vs. Process  
- **Program** = file on disk (instructions + data).  
- **Process** = a running instance of a program.  



### Process Components  
Each process is given by the OS:  
- **Virtual Registers** → private copy of CPU registers.  
- **Virtual Memory** → private address space containing:  
  - **Stack** (local variables)  
  - **Heap** (dynamic memory)  
  - **Globals** (global variables + static data)  
  - **Code** (instructions being executed)  



## 🔍 Example from Lecture  

Two copies of the same program are run:  
- Both print **local** and **global** variables.  
- Observation:  
  - Local variables naturally independent.  
  - Global variables **also independent**, even if they appear at the same address.  

✅ Proof: Each process gets its **own virtual address space**.  



## 🖼 Diagram: Process Virtualization  

```text
          ┌──────────────────────────────┐
          │          Process A           │
          │ ┌─────────┐  ┌───────────┐  │
Virtual    │ │ Stack   │  │  Globals  │  │
Memory     │ ├─────────┤  ├───────────┤  │
           │ │ Heap    │  │  Code     │  │
          └─┴─────────┴──┴───────────┴──┘
               ↑  Private to Process A
               (address 0x1234 here may mean 
                something totally different 
                in Process B)

          ┌──────────────────────────────┐
          │          Process B           │
          │ ┌─────────┐  ┌───────────┐  │
Virtual    │ │ Stack   │  │  Globals  │  │
Memory     │ ├─────────┤  ├───────────┤  │
           │ │ Heap    │  │  Code     │  │
          └─┴─────────┴──┴───────────┴──┘

```
---
Lecture 2

# Lecture 2 – Kernels, System Calls, and Hello World Internals

## 🌐 Introduction
- Explore what happens when running **Hello World**.
- Learn about **kernels**, **system calls**, and how programs interact with the OS.
- Some of this is exam-relevant, some is just fun background.

## ⚙️ Assembly & Instruction Set Architectures (ISA)
- **Machine Code** = actual binary numbers CPU executes.  
- **Assembly** = human-readable version of machine code.  

### Major ISAs Today
- **x86-64** → desktops, servers, Windows laptops.  
- **ARM64 (AArch64)** → phones, tablets, Apple M-series Macs, Snapdragon laptops.  
- **RISC-V** → open-source ISA, popular in academia/research.  

📌 In this course:
- You **won’t write** assembly.
- You **may need to read** small snippets.

## 🖥️ Processes & File Descriptors
- **Process** = running program (with its own memory + registers).  
- **File Descriptor (FD)** = integer handle used for I/O.  
- FDs can point to:
  - File
  - Terminal
  - Network socket
  - Pipe

### Standard FDs
```text
0 → stdin   (input)
1 → stdout  (output)
2 → stderr  (errors/logs)
```

## 🔧 System Calls

- A **system call (syscall)** is a request from a user program to the OS kernel.  
- They **look like functions** in C but actually invoke kernel services.

### Minimal Hello World
Two syscalls are required:

1. **`write(fd, buffer, count)`**
   - `fd` = file descriptor (1 = stdout)  
   - `buffer` = pointer to characters  
   - `count` = length of string  
   - returns number of bytes written  

2. **`exit_group(status)`**
   - Ends the process  
   - Usually called when `main` returns  

### Example (low-level C-like pseudocode)
```c
void _start() {
    write(1, "Hello World\n", 12);
    exit_group(0);
}
```

## 📚 API vs ABI

- **API (Application Programming Interface):**  
  Describes functions at a high level (names, arguments, return values).  

- **ABI (Application Binary Interface):**  
  Defines low-level details of how functions are invoked (register usage, stack layout, calling conventions).  

➡️ System calls rely on the **ABI**, not just the API.  

## 🛠️ How Syscalls Work

- Syscalls are **not normal function calls**.  
- Instead, they use a **special CPU instruction** to transition into kernel mode.  

### On ARM64
- Syscall number → stored in register `X8`  
- Arguments → placed in `X0` … `X5`  
- Instruction `SVC` (Supervisor Call) → traps into the kernel  

### Example Flow
```text
X8 = syscall_number
X0 = arg0
X1 = arg1
...
SVC → trap into kernel
```

