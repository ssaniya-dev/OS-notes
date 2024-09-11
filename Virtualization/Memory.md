# Virtualizing the Memory: Memory Management
- space-sharing: Put all process' in-memory data in memory, and let the OS manage the memory.
## Abstraction of Address Space
- Address space: The range of addresses that a process can use to address memory.
### Address Space Layout
- Code: The program's executable code; PC always points to somewhere in this segment.
- Static data: Holds global variables and static variables; could be separated into initialized (reads) and uninitialized data (writes allowed).
- Stack: the function call stack of the program to keep track of where it is in the call chain; function calls form a stack of frames, where the local variables of a function resides in its frame; SP & FP always point to somewhere in this segment; stack grows negatively
- Heap (data): Dynamically allocated memory; grows positively
### Memory APIs
- brk(), sbrk(): Change the size of the heap
- malloc, calloc, realloc, free: Allocate, deallocate memory
- nmap, munmap: create/unmap a memory region for mapping a file or device into address space 
## Address Mapping & Translation
- hardware-based address translation: The CPU translates the virtual address to a physical address.
### Base and Bounds
- Makes use of two registers:
    - Base register: starting offset the OS decides to put the current process's address space at
    - Bound register: the size (limit) of the current process's address space 
- Upon context switch, the two registers get loaded the base & bound values and requires OS intervention to decide where to relocate (memory management unit)
- Drawback: Requires physical memory has big, contiguous chunk of empty space to hold address space of process
- Both external and internal fragmentation
### Segmentation
- Segment: Contiguous portion of the address space
- Break the address space into segments: code, data, stack, heap, etc; apply base & bound to each segment
    - How does the hardware know which segment it is referring to?
        - Explicit: chop address space into fixed-size segments sot hat the highest several bits of virtual address is segment ID adn remaining bits is offset 
        - Implicit: hardware detects how address derived and determines segment 
- External fragmentation (if w/o periodic garbage collection)
### Paging 
- paging: chopping the address space into fixed-size pages 
- frame: fixed-size chunk of physical memory
- Page table: Maps virtual page number (VPN) o physical frame number (PFN) 
- Any virtual address can be split into two components: VPN of page & offset in the page 
    - bits for offset = log2 (page size)
    - bits for VPN = bits for virtual address - bits for offset
- Internal fragmentation (worse with larger page size)
#### Linear Page Table
- One-level array of page table entries
- OS indexes array by VPN and each entry contains PFN and control bits:
    - Valid bit: whether VPN is in-use or unused (if unset, then page fault)
    - Present bit: page is mapped but currently swapped out of memory to disk
    - Protection bits: read/write/execute permissions
    - Dirty bit: page has been written to since last loaded from disk
    - Reference bit: page has been read or written to since last loaded from disk
- Page tables are stored in memory in kernel-reserved memory 
    - Hardware MMU keeps a page-table base register (PTBR) to record a pointer to the start of current process's page table 
## Free-Space Management Policies
- How does the OS find proper free space when it needs to allocate something?
- Fragmentation: space wasted in address mapping scheme
- Internal fragmentation: unused bytes in the address space getting mapped
- External fragmentation: small free spaces being left over between two mapped regions
### Splitting & Coalescing
- Suppose we had af ree list data structure as linkedlist of free spaces. Two mechanisms play a role in all allocator policies:
    - Splitting: When a small request comes, split an existing large chunk and return requested size and keep remaining chunk
    - Coalescing: When a piece of memory is freed and there is free chunk on either side, merge them into one big chunk
### Basic Policies
- Best-fit: Search through free list and find smallest free chunk that is big enough for request
    - Intuitive but expensive
- Worst-fit: find largest chunk, split and return the requested amount, keep remaining chunk
    - If use max-heap can find chunk fast but performs badly and can lead to fragmentation
- First-fit: find first block that is big enough, split and return the request amount
    - Fast but can pollute the beginning of free list with small objects
- Next-fit: First-fit but remember where the last allocation was made and start searching from there
    - requires keeping an extra state 
### Segregated Lists
- If a particular application has one or a few popular-sized request it makes, keep separate lists just to manage objects of that size 
- All other requests are forwarded to general memory allocator 
### Buddy Allocation
- Binary buddy allocator is designed around splitting & coalescing in sizes of 2^i, allowing internal fragmentation and making coalescing procedure more efficient
    - In beginning, allocator holds one big space of 2^k size
    - When request comes, the search for free space recurisvely divides free space by 2 until the smallest power of 2 that is big enough for the request is found 
    - When that block is freed, the allocator checks if its buddy block (one produced by the last split, its sibling block) is free; if so, it coalesces the two blocks; this procedure repeats recursively up the tree until the top of the tree is reached
## Advanced Paging
### Translation Lookaside Buffer (TLB)
- cache for address translation (part of hardware so very fast)
    - At context switch to new process, if we do not have an address-space ID field in TLB entries, the TLB must be flushed
    - Upon an address translation, the MMU checks if the TLB holds the translation for the VPN
        - If yes, hit then we extract PFN 
        - If miss, go to page table in memory and decide whether to put translation for this VPN into TLB. If TLB is full, need to invict one 
### Multi-Level Page Tables
- 