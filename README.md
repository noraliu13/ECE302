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
---

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

---

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

--- 

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

---

# Lecture 6 – Basic IPC and Signals

## Introduction to IPC
- IPC (Interprocess Communication) is necessary because processes are independent.
- Even printing "Hello World" involves IPC (process writes to the terminal).
- IPC is just transferring bytes between processes.
- Reading and writing files is a form of IPC:
  - One process writes to a file.
  - Another process reads from the file.
- Processes on different machines can communicate via IPC (e.g., internet).

## Basic Read/Write Example

```c
#define BUFFER_SIZE 4096

char buffer[BUFFER_SIZE];
int bytes_read;

while ((bytes_read = read(0, buffer, sizeof(buffer))) > 0) {
    int bytes_written = write(1, buffer, bytes_read);
    assert(bytes_read == bytes_written);
}

if (bytes_read == -1) {
    perror("read error");
}
```

## Reading and Writing

- Reads from **standard input** (`fd = 0`) and writes to **standard output** (`fd = 1`).
- `read()` is **blocking**: waits until input is available.
- End of file is signaled by `read()` returning `0`.
- Errors are indicated by `read()` returning `-1`.



## Standard File Descriptors

- `0` → standard input (stdin)  
- `1` → standard output (stdout)  
- `2` → standard error (stderr)  

### Notes:

- File descriptors can be closed and reassigned:  
  Closing `fd 0` and opening a file assigns it to `fd 0`.
- Shell can redirect standard file descriptors before running a program:  
  ```bash
  ./program < file.txt
  ```
## Pipes

- Pipes connect the **stdout** of one process to the **stdin** of another.  
- Example:
  ```bash
  echo "Hello" | ./read_write_example
  ```
## Signals

- Signals are a form of **IPC** that interrupts a process.
- Default signal handlers:
  - Ignore the signal
  - Terminate the process
- Example: Pressing **Ctrl+C** sends `SIGINT` (default: terminate)

### Common Signals

| Signal   | Number | Description                     |
|-|--||
| SIGINT   | 2      | Keyboard interrupt              |
| SIGKILL  | 9      | Force terminate                 |
| SIGTERM  | 15     | Polite terminate                |
| SIGSEGV  | 11     | Memory access violation         |



## Handling Signals

- Register custom signal handlers with `sigaction`.

```c
void handle_signal(int signal_num) {
    printf("Received signal %d\n", signal_num);
}

struct sigaction sa;
sa.sa_handler = handle_signal;
sigaction(SIGINT, &sa, NULL);
sigaction(SIGTERM, &sa, NULL);
```

- Interrupted system calls return `-1` and set `errno` to `EINTR`.
- Can handle by **retrying the system call**.



## Killing Processes

- `kill <pid>` → sends `SIGTERM` by default  
- `kill -9 <pid>` → sends `SIGKILL` (cannot be ignored)  
- Only the process owner or root can kill processes.  
- Root can kill any process, but **PID 1 (systemd)** is protected.



## Non-blocking Wait

- `waitpid(pid, &status, WNOHANG)` → returns immediately if child has not exited.

### Example

```c
while (waitpid(-1, &status, WNOHANG) == 0) {
    sleep(1); // reduce wasted CPU cycles
}
```

## Tradeoff

- **Longer sleep** → slower reaction  
- **Shorter sleep** → higher CPU usage



## Signal Handling with Children

- The kernel sends `SIGCHLD` when a child process terminates.  
- Parent can register a handler:

```c
void sigchld_handler(int signal_num) {
    int status;
    wait(&status);
}
```

- Ensures child processes don’t become **zombies**.
- Signal handlers can be nested; careful **resource management** is needed (e.g., closing files).

## Summary

- IPC enables communication between independent processes.
- Standard file descriptors provide easy read/write access.
- Pipes allow chaining processes.
- Signals allow asynchronous notifications.
- Signal handlers can handle or ignore signals.
- Non-blocking waits and signal-driven waits provide flexibility in process management.
- Proper cleanup is essential to avoid **zombies** and **resource leaks**.

---

# Lecture 7: Processes and Pipes

## 1. Overview
- Today’s focus: process creation, zombies/orphans, signals, and interprocess communication (IPC) via pipes.
- Relevant for **Lab 2**: common mistakes can cause hours of debugging.
- System call layer: we are using, not implementing, OS internals.

## 2. Processes and `init`
- `init` process (from Unix):
  - Launches shell (`sh`) via `fork()` and `exec()`.
  - Uses an infinite loop to wait for child processes.
  - Cleans up zombies/orphans.
- Older OS: **uni-programming** - only one process at a time.
- Modern OS: **multi-programming**, parallel/concurrent execution using CPU cores.
- **Scheduler**:
  - Pauses current process, saves its state (registers, memory).
  - Chooses next process to run.
  - Loads its state and executes.

### Context Switching
- Mechanism to switch between processes.
- Saves current registers, loads next process registers.
- Overhead exists; optimized by OS (skip saving unused registers).

### Multitasking Styles
1. **Cooperative**: process must yield CPU voluntarily.
2. **Preemptive (True multitasking)**: OS can take CPU control, using:
   - Time slices
   - Interrupts

## 3. Pipes
- System call: `pipe(int fds[2])`
  - Returns `0` on success, `-1` on failure.
  - `fds[0]` → read end
  - `fds[1]` → write end
- Pipe: **one-way communication channel** between processes.
- Managed by kernel, accessed only via file descriptors.

### Key Points
- Parent and child processes **share file descriptors** after `fork()`.
- Important: **close unused ends** to avoid blocking reads/writes.

## 4. Example: Parent → Child Communication

### Child
```c
close(fds[1]); // Close write end
char buffer[496];
int bytes_read = read(fds[0], buffer, sizeof(buffer));
if (bytes_read > 0) {
    printf("%.*s\n", bytes_read, buffer);
}
close(fds[0]);
```

## Parent 
```c
close(fds[0]);          // Close read end
char *msg = "Howdy child";
write(fds[1], msg, strlen(msg));
close(fds[1]);
```

# Pipe Behavior Notes

## Pipe Behavior
- `read()` is **blocking**: waits for data unless all write ends are closed.
- Closing **unused file descriptors** ensures `read()` returns `0` when no more data is possible.
- **Standard file descriptors**:
  - `0` → stdin
  - `1` → stdout
  - `2` → stderr
- Closing `stdout` (`1`) disables `printf()` output.

## Important Tips
- **Always close file descriptors** as soon as they are no longer needed to prevent blocking or resource leaks.
- **Memory management**:
  - Allocated **before `fork()`** → exists in both parent and child; free in both.
  - Allocated **after `fork()`** → exists only in that process; free there.
- **Pipes must be created before `fork()`** to share between parent and child.
- **Multiple reads advance the pipe buffer**; data is **not repeated** unless explicitly repositioned.

## Common Pitfalls
- Forgetting to close **write end in child** → `read()` blocks forever.
- Forgetting to close **read end in parent** → `write()` may succeed but no reader exists.
- Creating a pipe **after `fork()`** → parent and child have independent pipes → no communication.
- Using **multiple writers/readers** → execution order affects output (race conditions).

## Fun / Advanced Notes
- `&` in the shell → runs processes in the background.
- Recursive pipe forks (`while true; fork`) → can crash the system.
- Multiple processes reading/writing → race conditions possible.
- `printf()` may fail if `stdout` is closed.

---

# Lecture 8 - Subproccesses

## Lecture Overview
- Topic: Completing Lab 2 – sending and receiving data from a process.
- Goals:
  1. Create a new process that launches a command-line argument.
  2. Send the string `"testing\n"` to that process.
  3. Receive and read any data printed to its standard output.

 

## APIs Covered
### `execve` / `execvp`
- `execve`: Starts running another program, replacing the current process.
- `execvp`: Convenient wrapper over `execve`.
  - Finds program in `PATH`, no need for full path.
  - Pass C strings directly; terminate with `NULL`.
  - If successful, does **not return**; on failure, returns `-1` and sets `errno`.

### `dup` / `dup2`
- `dup(oldfd)` → returns new FD pointing to the same resource.
- `dup2(oldfd, newfd)` → makes `newfd` point to same resource as `oldfd`.
  - Closes `newfd` first if already in use.
  - Useful to manipulate standard file descriptors (`stdin`, `stdout`, `stderr`) before `exec`.

 

## File Descriptors
- Standard FDs:
  - `0` → stdin
  - `1` → stdout
  - `2` → stderr
- Closing `stdout` disables `printf()` output.
- Pipes are **one-way communication channels**; need one pipe per direction.

 

## Pipe Setup & Forking
1. **Create pipe before fork** to share between parent and child.
2. **Fork**:
   - Parent: `fork()` returns child PID.
   - Child: `fork()` returns `0`.
3. Each process initially has the same FDs open (0, 1, 2, plus pipe FDs).

### Reading from a Pipe
- `read()` is **blocking**: waits until all write ends are closed and data is consumed.
- Must close unused pipe ends to prevent blocking.

### Sending Data via Pipe
- Write data to the pipe using `write(pipe_fd, buffer, size)`.
- Read from the pipe with `read(pipe_fd, buffer, size)`.

 

## Example: Capturing `uname` Output
- Parent creates a pipe (`out_pipe`) and forks.
- Child:
  - Redirects `stdout` to the write end of `out_pipe` via `dup2`.
  - Closes unused pipe FDs.
  - Executes `uname` via `execvp`.
- Parent:
  - Reads from the read end of `out_pipe`.
  - Prints output (e.g., `"Linux"`).

**Important**: Always close unused FDs to avoid blocking or resource leaks.

 

## Bidirectional Communication
- Use two pipes:
  1. `out_pipe`: child writes, parent reads.
  2. `in_pipe`: parent writes, child reads.
- Redirect FDs using `dup2`:
  - Child `stdout` → `out_pipe` write end.
  - Child `stdin` → `in_pipe` read end.
- Parent writes data to `in_pipe` and reads from `out_pipe`.

 

## Example: Sending `"testing\n"` to a child running `cat`
1. Parent writes `"testing\n"` to `in_pipe`.
2. Child reads from `stdin` (redirected to `in_pipe`) and writes to `stdout` (redirected to `out_pipe`).
3. Parent reads from `out_pipe` and sees `"testing\n"`.

## Best Practices
- **Close file descriptors** as soon as you are done using them:
  - Parent: close read end of `in_pipe` after writing.
  - Parent: close write end of `out_pipe` if not used.
  - Child: close unused ends of pipes before `exec`.
- **Wait for child process** using `wait(pid, &status, 0)` to prevent zombies/orphans.
- Use `assert` to check child exit status:
  ```c
  int wstatus;
  wait(pid, &wstatus, 0);
  assert(WIFEXITED(wstatus));
  assert(WEXITSTATUS(wstatus) == 0);
  ```

## Common Pitfalls
- Not closing the **write end** in the parent or **read end** in the child → can cause reads/writes to block.  
- Leaving pipes open → child may hang waiting for EOF (e.g., with `cat`).  
- Overwriting `stderr` unnecessarily → keep it available for error messages.  

## Lab 2 Steps Summary
1. Create two pipes: `in_pipe` and `out_pipe`.  
2. Fork a new process.  
3. **Child process**:  
   - Redirect `stdin` and `stdout` using `dup2`.  
   - Close any unused pipe file descriptors.  
   - Execute the target program with `execvp`.  
4. **Parent process**:  
   - Write data to `in_pipe`.  
   - Read output from `out_pipe`.  
   - Close any used file descriptors.  
   - Wait for the child to finish.  

## Key Takeaways
- Proper management of file descriptors is essential for inter-process communication.  
- Pipes provide a straightforward way for data transfer between parent and child.  
- `dup2` enables redirection of standard I/O to pipes.  
- Always wait for child termination to avoid creating zombies or orphaned processes.  


---

Lecture 9: Basic Scheduling 

# CPU Scheduling

## Overview
CPU scheduling determines **which process runs when**.  
This topic follows process management (creation, termination, adoption, etc.) and focuses on CPU resource sharing.

## Types of Resources

### Preemptable Resources
- Can be taken away and reassigned.
- Example: CPU.
- Shared via **scheduling**.

### Non-Preemptable Resources
- Cannot be taken without consent.
- Examples: Disk, memory.
- Shared via **allocation/deallocation**.

## Dispatcher vs Scheduler

- **Dispatcher**: Performs the actual context switch (saves and restores process state).
- **Scheduler**: Decides **which process runs next**.

Schedulers run when:
- A process terminates.
- A process blocks.
- (Preemptive) The OS interrupts and selects another process.

## Scheduling Evaluation Metrics

- Minimize **waiting time**  
- Minimize **response time**  
- Maximize **CPU utilization**  
- Maximize **throughput**  
- Ensure **fairness**

> Note: Fairness can conflict with other metrics (e.g., minimizing waiting time).

## Scheduling Algorithms

### 1. First Come First Serve (FCFS)
- Non-preemptive.
- Processes are executed in order of arrival (FIFO).
- Simple to implement.
- Can cause **long waiting times** for short jobs behind long ones (**convoy effect**).

### 2. Shortest Job First (SJF)
- Non-preemptive.
- Picks the process with the shortest CPU burst next.
- **Optimal** for minimizing average waiting time.
- Requires knowledge of burst times → **impractical**.
- Can lead to **starvation** (long jobs never run).

### 3. Shortest Remaining Time First (SRTF)
- Preemptive version of SJF.
- Preempts running process if a new one has shorter remaining time.
- Optimal for average waiting time.
- Still **impractical** and prone to **starvation**.

### 4. Round Robin (RR)
- **Preemptive and fair.**
- Each process gets a fixed **time quantum**.
- If unfinished, it moves to the back of the ready queue.
- Context switches occur between processes.

**Tradeoffs:**
- **Short quantum** → more context switches (higher overhead).
- **Long quantum** → less fairness (behaves like FCFS).

## Key Takeaways

- **FCFS:** Simple but unfair to short jobs.  
- **SJF / SRTF:** Optimal but impractical; risk of starvation.  
- **Round Robin:** Fair and widely used.  
- **Starvation:** Common in SJF and SRT







