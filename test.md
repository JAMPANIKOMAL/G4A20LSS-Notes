# Unit 01 - UNIX Operating System

## Why UNIX Became So Popular

UNIX gained widespread adoption due to several key characteristics:

- **Portable**: Written in a high-level language (C), making it easy to read, understand, change, and move to other machines
- **User-Friendly**: Provides a powerful yet simple interface
- **Modular**: Offers primitives for building complex programs
- **Organized**: Uses a hierarchical file system structure
- **Consistent**: Employs the byte stream file format throughout
- **Device-Agnostic**: Uniform device interface for all peripherals
- **Multi-User/Process**: Supports concurrent execution of multiple processes
- **Hardware Abstraction**: Ensures portability across different systems

## High-level Architecture of UNIX Systems

The UNIX system is organized in a layered architecture:

### Layer 1: Hardware
Contains all hardware-related information and physical components.

### Layer 2: Kernel
- Core of the Operating System
- Software interface between hardware and software
- Handles critical tasks:
  - Memory management
  - File management
  - Network management
  - Process management

### Layer 3: Shell Commands
- Interface between user and kernel
- Shell utility processes user requests
- Interprets commands and calls appropriate programs
- Common commands: `cp`, `mv`, `cat`, `grep`, `id`, `wc`, `nroff`, `a.out`

### Layer 4: Application Layer
- Outermost layer
- Executes external applications

### System Interaction
- Operating system interacts directly with hardware
- Provides common services to programs
- Insulates programs from hardware idiosyncrasies
- Programs interact with kernel through well-defined **system calls**

## User Perspective

### Shell
The OS provides a command-line interface (shell) to interact with files, processes, and system services.

### The File System

#### Structure
- **Hierarchical Structure**: Tree-like organization with root ('/')
- **Consistent Data**: Uniform treatment of file content
- **File Management**: Easy creation and deletion of files
- **Dynamic Growth**: Files can expand as needed
- **Data Protection**: Security mechanisms for file access
- **Devices as Files**: Unified interface for peripherals

#### Node Types
- **Root**: Top-level directory
- **Directories**: Organize files (non-leaf nodes)
- **Files**: Data storage (leaf nodes: directories, regular files, special device files)

### Process Management
Users can run commands, scripts, and applications, managing system resources indirectly via system calls or shell commands.

## Operating System Services

- **Process Control**: Creation, termination, suspension, and inter-process communication
- **CPU Scheduling**: Fair CPU time sharing using time-sharing and context switching
- **Main Memory Management**: Memory allocation, address space protection, and memory sharing
- **Swapping/Paging**: Memory management by moving processes/pages to swap space when memory is low
- **File System Management**: Secondary storage allocation/reclamation, file system structuring, and data protection
- **Device Access**: Controlled access to I/O devices (disks, terminals, network interfaces)
- **Transparency**: Hides internal file formats and device distinctions, returning simple byte streams
- **Shell Support**: Services for input reading, process spawning, piping, and I/O redirection
- **Customizability**: Supports user-built environments using kernel services

## Hardware Assumptions

### Execution Modes
- **User Mode**: Limited access to user data and instructions
- **Kernel Mode**: Full access to system memory and devices
- **System Calls**: Switch execution from user to kernel mode

### Privileged Instructions
- Executable only in kernel mode
- Prevent unauthorized access to system-level operations

### Kernel-User Relationship
- Kernel runs as part of user processes
- Handles system-level tasks on behalf of user processes

### Operating System vs Hardware View
- **OS View**: Sees which *process* is running
- **Hardware View**: Sees which *mode* is active

### Interrupts
- Triggered by external devices (clock, I/O)
- Kernel saves context, handles event, and resumes execution
- Prioritized handling based on importance

### Exceptions
- Caused by process errors (divide by zero, illegal memory access)
- Handled mid-instruction and resumed afterward
- Treated differently from hardware interrupts

### Unified Handling
Single mechanism manages both interrupts and exceptions.

### Processor Execution Levels
- Kernel masks lower-priority interrupts during critical operations
- Execution level set using privileged instructions

### Memory Management
- Kernel resides permanently in main memory
- Compiler generates virtual addresses
- Kernel maps virtual to physical addresses using hardware
- Supports techniques like paging

# Introduction to the Kernel

## Architecture of the UNIX Operating System

The UNIX system supports two fundamental illusions:
- File system has "places"
- Processes have "life"

**Files** and **processes** are the two central concepts in the UNIX system model.

### Core Architecture Components

#### System Structure
- **Three layers**: User, Kernel, and Hardware
- **System Call Interface**: Boundary between user programs and kernel

#### Libraries and System Calls
- System calls appear as C functions to user programs
- Libraries translate these into system-level operations
- Assembly code can call system calls directly

### File Subsystem
- Manages files, space allocation, and access permissions
- **System calls**: `open`, `read`, `write`, `close`, `stat`, `chown`, `chmod`
- Uses **buffer cache** for efficient I/O
- Interacts with:
  - **Block I/O drivers** (e.g., disk)
  - **Character/raw I/O drivers** (e.g., terminals, tapes)

### Process Control Subsystem
Handles:
- Process creation & termination
- Process scheduling
- Memory management
- Inter-process communication

**System calls**: `fork`, `exec`, `exit`, `wait`, `brk`, `signal`

### Memory Management
- Allocates memory fairly across processes
- Swaps processes between main and secondary memory if needed
- **Two policies**: Swapping and Demand Paging
- **Swapper**: Manages memory allocation (distinct from CPU scheduler)

### CPU Scheduler
- Allocates CPU based on priority and time quantum
- Preempts processes that exceed their time slice

### Interprocess Communication
- **Signals**: Asynchronous communication
- **Message passing**: Synchronous communication

### Hardware Control
- Manages hardware interrupts (disk, terminal, etc.)
- Interrupts serviced by **kernel functions**, not separate processes
- Interrupted process resumes after interrupt handling

## Introduction to System Concepts

### An Overview of the File Subsystem

#### Inode (Index Node)
- Internal representation of a file
- Stores file metadata:
  - Owner information
  - Permissions
  - Timestamps
  - Disk layout
- Each file has **one inode**
- Multiple file names (links) can map to same inode
- Assigned by kernel when new file is created

#### In-Core Structures

1. **Inode Table**: Maintained in memory for file access
2. **File Table**: Global table tracking open files, byte offset, access rights
3. **User File Descriptor Table**: Per-process table storing open file descriptors

#### File Access Mechanism
- `open()`/`creat()` create entries in all three tables
- File descriptor returned → index into user file descriptor table
- `read()`/`write()` follow file descriptor to inode via table pointers

### Logical vs Physical Devices

#### Logical Devices
- Kernel treats file systems as logical devices with unique device numbers
- Device drivers map **logical addresses** to **physical disk locations**

#### Disk Usage
- UNIX uses **disks**, not tapes, for file systems
- File systems can be partitioned for easier management

### File System Layout

#### Logical Block
- Basic unit of file system storage (typically **1K bytes**)
- Size uniform within file system but can differ across systems
- **Trade-off**: Larger blocks = faster transfer; too large = wasted space

#### File System Components

1. **Boot Block**
   - First block in the file system
   - May contain bootstrap code for OS booting
   - Present in all file systems (even if unused)

2. **Super Block**
   - Contains metadata: size, free space, file limits
   - Essential for managing file system state

3. **Inode List**
   - Contains all file inodes
   - One inode designated as root inode (for mounting)
   - Size set during file system configuration

4. **Data Blocks**
   - Store actual file content and administrative data
   - Each block belongs to only one file

## Processes

### Definition
- A process is the execution of a program
- Consists of: **text** (instructions), **data**, and **stack**
- Communicates with others via **system calls**

### Process Creation
- Created using `fork()` system call
- **Parent process** creates **child process**
- All processes (except **process 0**) created this way
- **Process 0**: Manually created during boot → becomes **swapper**
- **Process 1 (init)**: Ancestor of all other processes

### Executable File Structure
- **Headers**: File attributes
- **Program text**: Machine instructions
- **Initialized data**: Stored in executable (e.g., `int version = 1;`)
- **Uninitialized data**: `bss` section (e.g., `char buffer[2048];`)
- **Other info**: Symbol tables, etc.

```c
char buffer[2048];  // bss
int version = 1;    // initialized data
```

### Memory Regions After `exec()`
- **Text**: Code section
- **Data**: Initialized + bss section
- **Stack**: Created at runtime, grows dynamically
  - Holds: function parameters, local variables, return data

### Stacks
- **User Stack**: Used during user mode execution
- **Kernel Stack**: Used after system call (kernel mode)
- Switch triggered via **trap instruction** on system call

### Process Data Structures

#### Process Table
- Holds global process information:
  - Process ID (PID)
  - State (running, sleeping, etc.)
  - User ID (UID)
  - Event descriptor for sleeping state

#### u Area
- **Kernel-only memory space**
- Stores process-specific runtime data:
  - Pointer to process table entry
  - System call parameters, return values, error codes
  - Open file descriptors
  - I/O parameters
  - Current directory and root
  - Size limits (file/process)

### Memory Regions and Sharing
Managed via:
- **Per-Process Region Table** → links to global **Region Table**

Supports:
- Shared memory across processes
- Copy-on-write on `fork()`
- Memory release on `exit()`

## Context of a Process

### Context Includes:
- Program code (text)
- Global variables and data
- CPU registers
- Process table slot
- u area
- User and kernel stacks

### Not Included:
- OS code and global kernel data (shared by all processes)

### Context Switch
- Happens when system switches from one process to another
- Saves current process state and loads the new one

### Important Distinctions
- **Mode Switch ≠ Context Switch**: Switching between user ↔ kernel mode is **not** a context switch
- **Interrupt Handling**: Interrupts served **in kernel mode**, handled **within the context** of interrupted process

## Process States

1. **User Running**: Process currently executing in user mode
2. **Kernel Running**: Process currently executing in kernel mode
3. **Ready to Run**: Process not executing, but ready to run when scheduler chooses it
4. **Sleeping**: Process is sleeping

> **Note**: Since a processor can execute only one process at a time, at most one process may be in states 1 and 2.

## State Transitions

Processes move continuously between states according to well-defined rules. A **state transition diagram** represents:
- **Nodes**: Process states
- **Edges**: Events triggering transitions

### Key Transition Rules
- Kernel **allows context switches only** when process moves from **kernel running** → **asleep in memory**
- Processes running in kernel mode are **non-preemptive** (cannot be interrupted by others)
- This protects kernel data structures from corruption during critical operations

### Critical Section Example
```c
// Inserting node bp1 into doubly linked list
bp1->forp = bp->forp;
bp1->backp = bp;
bp->forp = bp1;
// possible context switch here - DANGEROUS!
bp1->forp->backp = bp1;
```

If context switch occurs at marked line, linked list becomes inconsistent, risking corruption.

### Protection Mechanisms
To avoid corruption, kernel:
- Raises **processor execution level** during **critical regions** to block interrupts
- Critical regions are small code sections manipulating shared kernel data
- User-mode processes are **periodically preempted** by scheduler to prevent CPU monopolization

## Sleep and Wakeup

### Key Principles
- Process decides **on its own initiative** when to sleep or wake up
- Other processes can suggest alternatives, but process makes final decision
- **Interrupt handlers cannot sleep** (would suspend interrupted process by default)

### Sleep Mechanism
- Processes **sleep on an event**
- Remain in **sleep state** until event occurs
- When event occurs, **all processes sleeping on that event wake up**
- Move from **sleep** → **ready-to-run** state (not running immediately)
- Sleeping processes **do not consume CPU**

### Kernel Locking Mechanism

**Acquiring lock:**
```c
while (condition is true)
    sleep(event: condition becomes false);
set condition true;
```

**Releasing lock:**
```c
set condition false;
wakeup(event: condition is false);
```

### Example: Buffer Lock Contention
Three processes (A, B, C) contend for same locked buffer:
1. All sleep on buffer lock event
2. When unlocked, all wake up and move to ready state
3. Kernel picks one (say B), which locks buffer and proceeds
4. Others (A, C) sleep again if buffer remains locked

## Kernel Data Structures

### Design Characteristics
- Kernel data structures mostly use **fixed-size tables** instead of dynamic allocation
- **Advantage**: Simpler kernel code and easier algorithms
- **Disadvantage**: Limited number of entries based on initial configuration
  - If system runs out of entries, cannot allocate more dynamically
  - Must return error
  - Over-provisioning wastes memory but ensures stability
- Kernel algorithms often use **simple loops** to find free entries

## System Administration

### Administrative Processes
Perform tasks for overall user welfare:
- Disk formatting
- Creating and repairing file systems
- Kernel debugging

### Key Characteristics
- Use **same system calls** as regular user processes
- **Difference**: Have special rights and privileges
- Kernel distinguishes **superuser** with elevated privileges

### Becoming Superuser
- Logging in with special credentials
- Running special programs

Kernel treats administrative processes like normal processes but enforces privilege-based access control.

# The Buffer Cache

## Purpose
The buffer cache is an internal memory pool that the kernel uses to cache recently accessed disk blocks. It minimizes slow disk I/O operations by keeping frequently accessed data in faster main memory.

## How Buffer Cache Works

### Cache Operations
- When process requests file data, kernel first checks buffer cache
- **Cache hit**: Kernel serves data directly from memory—avoiding disk access
- **Cache miss**: Kernel reads data from disk into buffer cache, then provides to process
- For writing: Data cached first and written to disk later (**delayed write**) to reduce disk write frequency

### Architecture Placement
- Buffer cache sits between **file subsystem** (manages files/directories) and **block device drivers** (handle actual disks)
- Acts as bridge, caching disk blocks for faster access

### Key Data Structures Cached
- **File data blocks**: Actual content of files
- **Super block**: Metadata describing file system structure and free space
- **Inode**: Metadata describing individual files (layout, permissions, etc.)

### Advantages
- **Reduces disk I/O**: Keeps frequently accessed blocks in memory
- **Speeds up access**: Memory access significantly faster than disk
- **Improves system performance**: Enhances overall throughput and responsiveness

### Disadvantages
- **Memory overhead**: Consumes part of main memory
- **Added complexity**: Kernel algorithms for caching and eviction more complex
- **Risk of data loss**: If data is delayed-written and system crashes, unsaved changes might be lost

## Buffer Headers

During system initialization, kernel allocates space for configurable number of buffers based on memory size and performance constraints.

### Buffer Components
1. **Memory array**: Contains data from disk
2. **Buffer header**: Identifies the buffer

### Key Principles
- Data in buffer corresponds to data in logical disk block on file system
- **Important**: Disk block can **never** map into more than one buffer at a time

### Buffer Header Fields
- **Device number**: Specifies logical file system (not physical device)
- **Block number**: Block number of data on disk
- **Status field**: Summarizes current buffer status
- **Pointer to data area**: Points to data area (≥ disk block size)

### Buffer Status Conditions
- Buffer is locked/busy
- Buffer contains valid data
- Kernel must write buffer contents to disk before reassigning (**delayed-write**)
- Kernel currently reading/writing buffer contents to/from disk
- Process currently waiting for buffer to become free

### Buffer Header Structure
Two sets of pointers for traversal of buffer queues (doubly linked circular lists).

## Structure of the Buffer Pool

### LRU Algorithm
Kernel follows **Least Recently Used (LRU)** algorithm for buffer pool.

### Free List Management
- Kernel maintains **free list** preserving LRU order
- Dummy buffer header marks beginning and end of list
- All buffers put on free list when system boots
- When kernel wants **any** buffer: takes from head of free list
- When kernel wants **specific** buffer: can take from anywhere in list
- Used buffers, when freed: attached to end of list
- Buffers closer to head = least recently used

### Hash Queues
To avoid searching entire buffer pool, kernel organizes buffers into separate queues, **hashed** as function of device and block number.

#### Hash Queue Characteristics
- Hash queues are doubly linked circular lists
- Hashing function uniformly distributes buffers across lists
- Must be simple to maintain performance
- Hash function depends on both device number and block number

#### Important Rules
- Every disk block in buffer pool exists on **one and only one** hash queue
- Appears only **once** on that queue
- Presence on hash queue ≠ busy (could be on free list if status is free)
- **Key principle**: Buffer always on hash queue, but may/may not be on free list

### Buffer Allocation Strategy
- **Specific buffer needed**: Search appropriate hash queue
- **Any buffer needed**: Remove buffer from free list

## Scenarios for Retrieval of a Buffer

The `getblk` algorithm handles buffer allocation with five typical scenarios:

### Scenario 1: Block Found and Buffer Free
- Block found on hash queue
- Buffer is free and available

### Scenario 2: Block Not Found, Free Buffer Available
- Block not found on hash queue
- Buffer allocated from free list
- Buffer's device and block numbers changed

### Scenario 3: Block Not Found, Delayed Write Buffer Allocated
- Block not found on hash queue
- Buffer from free list marked "delayed write"
- Kernel must write delayed write buffer to disk first
- Then allocate another buffer

### Scenario 4: Block Not Found, No Free Buffers
- Block not found on hash queue
- Free list of buffers is empty
- Process must sleep until buffer becomes available

### Scenario 5: Block Found But Buffer Busy
- Block found on hash queue
- Buffer currently busy/locked
- Process must wait for buffer to become free

## Algorithm: getblk

```c
/*
 * Algorithm: getblk
 * Input: file system number, block number
 * Output: locked buffer that can now be used for block
 */
{
    while (buffer not found)
    {
        if (block in hash queue)
        {
            if (buffer busy)   // scenario 5
            {
                sleep (event: buffer becomes free);
                continue;      // back to while loop
            }
            mark buffer busy;  // scenario 1
            remove buffer from free list;
            return buffer;
        }
        else
        {
            if (there are no buffers on the free list)
            {
                sleep (event: any buffer becomes free);   // scenario 4
                continue;      // back to while loop
            }
            remove buffer from free list;
            if (buffer marked for delayed write)         // scenario 3
            {
                asynchronous write buffer to disk;
                continue;      // back to while loop
            }
            // scenario 2
            remove buffer from old hash queue;
            put buffer onto new hash queue;
            return buffer;
        }
    }
}
```

### Buffer Usage Protocol
- Kernel always marks buffer as **busy** during use
- Prevents other processes from accessing it
- When finished, kernel releases buffer using `brelse` algorithm

## Algorithm: brelse

```c
/*
 * Algorithm: brelse
 * Input: locked buffer
 * Output: none
 */
{
    wakeup all processes (event: waiting for any buffer to become free);
    wakeup all processes (event: waiting for this buffer to become free);
    raise processor execution level to block interrupts;
    if (buffer contents valid and buffer not old)
        enqueue buffer at end of free list;
    else
        enqueue buffer at beginning of free list;
    lower processor execution level to allow interrupts;
    unlock (buffer);
}
```

### Buffer Placement Strategy
- **Valid, recent data**: Buffer placed at **end** of free list (LRU strategy)
- **Invalid or old data**: Buffer placed at **beginning** of free list
  - "Old" = marked as "delayed write"
  - Invalid = I/O corruption or other issues

### Race Condition Handling
**Important guarantee**: Kernel ensures all processes waiting for buffers will wake up, because it:
- Allocates buffers during system call execution
- Frees them before returning

## Reading and Writing Disk Blocks

### Algorithm: bread (Block Read)

```c
/*
 * Algorithm: bread
 * Input: file system number, block number
 * Output: buffer containing data
 */
{
    get buffer for block (algorithm: getblk);
    if (buffer data valid)
        return buffer;
    initiate disk read;
    sleep (event: disk read complete);
    return buffer;
}
```

### Read Process
1. If data not in buffer pool, kernel initiates disk read
2. Driver "schedules" read request to disk controller
3. Controller copies data from disk to buffer
4. Disk interrupt handler awakens sleeping process

### Read-Ahead Optimization
Higher-level algorithms anticipate need for next disk block during sequential file access. Second read is asynchronous—kernel expects data to be available when needed.

### Algorithm: breada (Block Read-Ahead)

```c
/*
 * Algorithm: breada
 * Input: file system number and block number for immediate read
 *        file system number and block number for asynchronous read
 * Output: buffer containing data for immediate read
 */
{
    if (first block not in cache)
    {
        get buffer for first block (algorithm: getblk);
        if (buffer data not valid)
            initiate disk read;
    }
    if (second block not in cache)
    {
        get buffer for second block (algorithm: getblk);
        if (buffer data valid)
            release buffer (algorithm: brelse);
        else
            initiate disk read;
    }
    if (first block was originally in the cache)
    {
        read first block (algorithm: bread);
        return buffer;
    }
    sleep (event: first buffer contains valid data);
    return buffer;
}
```

### Read-Ahead Strategy
- If second block data found in buffer cache: release immediately (not needed right away)
- Will be acquired when data actually needed

### Algorithm: bwrite (Block Write)

```c
/*
 * Algorithm: bwrite
 * Input: buffer
 * Output: none
 */
{
    initiate disk write;
    if (I/O synchronous)
    {
        sleep (event: I/O complete);
        release buffer (algorithm: brelse);
    }
    else if (buffer marked for delayed write)
        mark buffer to put at head of free list;
}
```

### Asynchronous I/O Considerations
Due to two asynchronous I/O operations:
- Block read ahead
- Delayed write

Kernel can invoke `brelse` from interrupt handler, so it must prevent interrupts in any procedure manipulating buffer free list.
