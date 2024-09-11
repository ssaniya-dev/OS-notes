# Virtualizing the CPU
## Process
- Running instance of a program 
- OS should virtualize physical resource and let multiple processes share limited resoucre (sharing the CPU)
### Machine State
- Running process requires OS to remember machine state which contains:
    - Address space: memory space that process can address (virtualized)
        - Code: compiled machine code 
        - Data: any initial static data
        - Stack: space reserved for run-time function stack of process
        - Heap: space for new run-time data 
    - Registered context: CPU registers' values (i.e. program counter, stack pointer)
    - I/O information: states related to storage of network (i.e. list of currently open files)
### Process Status
- Process can be one of the states:
    - Initial (optional): being created and hasn't finished initialization 
    - Ready: is ready to be scheduled onto a CPU to run, but not scheduled 
    - Running; Scheduled on a CPU and executing instructions
    - Blocked: waiting for some event to happen 
    - Terminated (optional): has finished execution but info data structures haven't been cleaned up
### Process Control Block (PCB)
- Metadata structure that OS uses to keep track of process state
- Contains:
    - Process state: one of the states above
    - Program counter: address of next instruction to execute
    - CPU registers: values of registers
    - CPU scheduling information: priority, scheduling queue pointers
    - Memory-management information: memory allocated to process
    - Accounting information: CPU used, clock time elapsed
    - I/O status information: list of I/O devices allocated to process
- Process list: list of PCBs that OS maintains
### Process APIs
- Create: Initialize states, load program from presistent storage in executable format, and get process running at the entry point of code 
    - Fork: create a new process that is a copy of the current process (child process)
- Destroy: Forcefully destroy process in middle of execution 
- Wait: let process wait for termination of another process
- Status: get status of process
- System calls (syscalls): APIs OS provides to user programs 
### Time-Sharing of the CPU
- Virtualization must be performant (no excessive overhead) and controlled (OS controls which one runs at each time).
    - Limited Direct Execution (LDE): run user program directly on CPU (OS not simulating processor hardware but can re-gain control in some way to do coordination)
    - Time-sharing (Multiprogramming): Divide time into slots and scheudle process to run for a few slots then switch to another one, constantly going back and forth 
### Privilege Modes
- Privileged instructions: HALT, I/O, Turning off interupts
- Switching between user and kernel mode is called trap (changes the stack pointer to kernel stack and saves process's user registers into the kernel stack)
    - Trap table: table of addresses of kernel functions that are called when a trap occurs 
### Hardware Interrupts 
- Interrupts: signals from hardware devices that require immediate attention (ISR: Interrupt service routine)
- Timer interrupt: periodic interrupt that OS uses to regain control of CPU and do scheduling
### Context Switch & Scheduler
- Switching from process A to B:
    - Save A's context into PCB
    - Load B's context from PCB
    - Update PCB pointers (Jump to where B was left out -> most likely kernel stack)
- Timer interrupt ISR typically calls scheduler: a routine that chooses which process to run for next time slot 
### CPU Scheduling Policies
#### Basic Policies
- We use these metrics to compare policies:
    - Turnaround time: time from process arrival to completion
    - Response time: time from request to first response
- First-In-First-Out (FIFO): leads to convoy effect (short process behind long process)
- Shortest Job First (SJF): only works if jobs are scheduled at the same time (else defaults to FIFO) 
- Shortest-Time-to-Completion First (STCF): preemptive version of SJF
- Round Robin (RR): each process gets a time slice (quantum) and if it doesn't finish, it goes to the back of the queue (worst for turnaround time)
#### Multi-Level Feedback Queue (MLFQ)
- Problem with basic policies is they assume job durations are known 
- Number of distinct queues with different priority level 
- When job arrives, it is placed in the highest priority queue. If it usese entire time slice without relinquishing CPU, moves down one queue. If job gives up the CPU before time slice, stays on same priority level.
- Problems: Starvation (if quite a few interactive jobs that a long-running job might have no chance to run)
    - Priority boost: periodically boost priority of all jobs to prevent starvation
    - Gaming tolerance: if a job uses more than its fair share of CPU, it is penalized by moving down a queue
#### Lottery Scheduling (Fair-share scheduler)
- Assigns each process a number of lottery tickets and scheduler draws a ticket to determine which process to run next
- Mechanisms:
    - Ticket currency: A user can allocate tickets among their own child jobs in whatever scale (form hierarchy of currencies and eventual proportions are multiplied)
    - Ticket transfor: A user can transfer tickets to another user
    - Ticket inflation: A user can increase the number of tickets they have
#### Completely Fair Scheduler (CFS)
- Each job's vruntime increases at same rate with physical time; always pick process with smallest vruntime
- Parameter sched_latency decides lenght of whole round; CFS divides this value by number of process to determine the time slice length for each process
- CFS never assigns time slice lengths shorter than parameter min_granularity
- Each process has nice value from -20 to +19; positive nice implies lower priority 
