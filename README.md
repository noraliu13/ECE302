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


# Lecture 3 – System Calls, Kernels, and Libraries

## 🔧 System Calls Recap
- A **system call** is a function that runs in **kernel mode** instead of **user mode**.  
- They allow a program to request services from the kernel (e.g., access hardware).  
- Execution flow:  

## 🔧 System Call Flow
A system call transitions a program from **user mode** into **kernel mode** to request OS services.  

### Execution Flow
```text
Program (User Mode)
   ↓
System Call
   ↓
Switch to Kernel Mode
   ↓
Perform Operation
   ↓
Return to User Mode
```

## 🖥️ What Makes an Operating System?

- **Kernel**: The core of the OS that manages processes, memory, devices, and hardware access.  
- **Libraries**: Provide reusable functions and programming interfaces.  
- **Utilities**: User-facing tools and programs.  

### Formula
```text
Kernel + Libraries + Utilities = Operating System
```

### 🌍 Examples of Operating Systems
- **Apple platforms** (iOS, macOS, iPadOS, etc.) share the **same kernel**, but differ in libraries and user applications.  
- **Android vs Debian**: both rely on the **Linux kernel**, but feel very different because they use different libraries and utilities.  
- When people say *Linux*, they usually mean **GNU/Linux**:  
  - **Linux** → the kernel  
  - **GNU** → the C standard library + essential utilities  

 

## ⚙️ Compilation and Linking
- Each `.c` source file is compiled into an **object file** (`.o`) containing machine code.  
- The **linker** combines multiple `.o` files into a single executable.  

### Example
```text
main.c → main.o
util.c → util.o
foo.c  → foo.o
bar.c  → bar.o
     -
Linker → final executable
```

## 📚 Libraries

A **library** is a collection of precompiled object files that can be shared and reused across multiple programs.  
They help avoid rewriting common functionality and make programs more modular.  

 

### 🔒 Static Libraries (`.a`)
- Library code is **copied directly** into the executable during linking.  
- Produces a **self-contained** binary (no external dependencies at runtime).  

**Drawbacks**:  
- Larger executable size  
- Updating the library requires **relinking** every program  

 

### 🔗 Dynamic Libraries
- Linux: `.so`  
- macOS: `.dylib`  
- Windows: `.dll`  
- Code is **not embedded** into the executable. Instead, the program stores a **reference** to the library.  
- At runtime, the **dynamic linker** loads the required library.  

**Advantages**:  
- Smaller executables  
- Easy to update shared libraries without recompilation  

# Lecture 4 – Process Creation

## 1. 🔄 What is a Process?
When we run a program, it becomes a **process**.  
A process contains:
- **Virtual registers** (program counter, stack pointer, etc.)  
- **Virtual memory** (Heap, Stack, Globals)  
- **Open file descriptors** (array of ints managed by the kernel)  

## 2. 🖥️ Kernel vs User Space
- **Kernel Mode**
  - Runs privileged instructions  
  - Manages processes, files, memory, scheduling  

- **User Mode**
  - Runs regular applications  
  - Can only interact with the kernel via **system calls**  

- **Libraries**
  - Provide higher-level functions  
  - Often call system calls on behalf of programs  

### Diagram
```text
Application (User Mode)
   |
   | function calls
   v
Library
   |
   | system calls
   v
Kernel (Kernel Mode)
```

## 3. 📂 File Descriptors

Every process maintains its own **table of file descriptors (FDs)**.  
These are small integer indexes that map to open files, devices, or sockets managed by the kernel.  

### Standard File Descriptors
```text
0 → stdin   (standard input)
1 → stdout  (standard output)
2 → stderr  (standard error)

# 📘 Lecture 5 – Process Management

## 1. Introduction

- In the previous lecture, we learned **how to create a process**.
- Today: **managing processes**, understanding the **parent-child relationship**, and Linux-specific **process states**.

## 2. Linux Process States

- Linux has **five main states** for processes:

1. **Running (Runnable)**  
   - Process is currently running or ready to run.
   
2. **Interruptible Sleep (Sleeping)**  
   - Process has given up the CPU (e.g., waiting after a system call).  
   - Can be woken up if needed.

3. **Uninterruptible Sleep (Blocked)**  
   - Process is blocked and cannot run, typically waiting for I/O.

4. **Stopped**  
   - Process is explicitly paused. Can be resumed later.

5. **Zombie (Zed) State**  
   - Process has **terminated**, but its **exit status has not been acknowledged** by the parent.  
   - Exists in the process table but cannot run.


## 3. Unix Process Hierarchy

- When a computer boots:
  - Kernel runs a **single process** (PID 1, often `init` or `systemd`).
  - PID 1 creates all other processes either **directly or indirectly**.
- Parent-child relationship:
  - Strict in Unix.
  - Parent is responsible for its child’s termination acknowledgment.
  
### Example Process Tree

``` text
init/systemd (PID 1)
├─ journalD
├─ udev
├─ systemd-user
│ └─ gnome-shell
│ └─ Firefox
│ └─ Firefox tabs (each tab = separate process)
└─ terminal → shell → htop
```

## 2. Process IDs (PID)

- Every process gets a **unique PID** that does **not change** during its lifetime.
- Maximum PID in Linux: typically **32,000**, configurable.
- PID 0: reserved/invalid, used only as a return value in `fork()`.

## 3. Process States (Linux)

1. **Running (Runnable)**: Process is executing or ready to execute.
2. **Interruptible Sleep**: Process is blocked but can be awakened.
3. **Uninterruptible Sleep**: Process is blocked waiting for I/O; cannot run.
4. **Stopped**: Process explicitly paused.
5. **Zombie**: Process terminated but still has a PID until parent reads its exit status.

## 4. Zombies and Orphans

### 4.1 Zombie Processes
- **Definition**: Process terminated but **parent has not called `wait()`**.
- Still occupies:
  - Process ID
  - Process Control Block
- **Example**:
```c
if (fork() == 0) {
    sleep(2);
    exit(0);
} else {
    // Parent does not call wait()
    sleep(3); 
    // Child becomes a zombie after 2 seconds
}
```

## 4.2 Orphan Processes

- **Definition**: Occurs when a parent process exits while its child is still running.  
- **Reparenting**: Kernel automatically assigns the orphaned child to a new parent (default: PID 1, `init`/`systemd`).  
- **Behavior**: Orphaned processes continue to run normally and are eventually cleaned up by the system.

## 5. `wait()` System Call

**Purpose**: Allows a parent process to **acknowledge the termination of its child** and collect its exit status.

**Syntax**:
```c
pid_t wait(int *status);
```

## `wait()` System Call

**Parameters:**  
- `status` → pointer to an integer where the child's exit information will be stored.

**Return Value:**  
- PID of terminated child  
- `-1` on error  
- `0` for non-blocking calls

**Behavior:**  
- Blocks until **any child** terminates.

**Macros:**  
- `WIFEXITED(status)` → Returns true if the child exited normally  
- `WEXITSTATUS(status)` → Returns the child's exit code

**Example:**
```c
pid_t pid = fork();
if (pid == 0) { 
    // Child process
    sleep(2);
    exit(42);
} else {
    // Parent process
    int wstatus;
    pid_t cpid = wait(&wstatus);
    if (WIFEXITED(wstatus)) {
        printf("Child PID %d exited with status %d\n", cpid, WEXITSTATUS(wstatus));
    }
}
```

## Forking Multiple Processes

- Each `fork()` creates a **duplicate of the calling process**.  
- **Return values distinguish parent and child:**  
  - `0` → child process  
  - `>0` → parent process (PID of child)  
- Multiple forks can create **many processes**.  
- **Note:** Variables are copied; be careful with shared data.

## Process Behavior Examples

### Zombie Example
- Parent does **not call `wait()`**, child sleeps 2 seconds.  
- After the child exits, it becomes a **zombie** until the parent acknowledges termination.

### Orphan Example
- Parent sleeps 1 second, child sleeps 2 seconds.  
- Parent exits **before child**, making the child an **orphan**.  
- Kernel reassigns the orphaned child to **PID 1 (`systemd`)** for cleanup.

## Process Table Limits

- **Zombies and orphans consume PIDs**.  
- PID exhaustion can occur due to:  
  - **Infinite fork loops** → prevents new processes from being created  
  - **Zombie processes** → occupy PID slots until cleaned up

## Key Takeaways

- `fork()` creates new processes; return values distinguish parent and child.  
- Each process has a **unique PID** and **independent address space**.  
- **Zombies**: terminated but **unacknowledged by parent**.  
- **Orphans**: parent died, **reparented to PID 1**.  
- Always use `wait()` to **properly clean up child processes**.
