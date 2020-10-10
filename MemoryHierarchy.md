<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [Memory hierarchy](#memory-hierarchy)   
      - [1. Incentive of using memory hierarchy](#1-incentive-of-using-memory-hierarchy)   
      - [2. Designers' target](#2-designers-target)   
      - [3. When to write cache data into memory](#3-when-to-write-cache-data-into-memory)   
      - [4. Categories of cache misses](#4-categories-of-cache-misses)   
      - [5. Relationship between miss rate and misses per instruction](#5-relationship-between-miss-rate-and-misses-per-instruction)   
      - [6. CPU execution time and cache misses](#6-cpu-execution-time-and-cache-misses)   
      - [7. Misses per instruction](#7-misses-per-instruction)   
      - [8. Pros and cons of Misses per instruction](#8-pros-and-cons-of-misses-per-instruction)   
      - [9. Basic questions about cache](#9-basic-questions-about-cache)   
      - [10. Average memory access time](#10-average-memory-access-time)   
      - [11. Average Memory Access Time and Processor performance](#11-average-memory-access-time-and-processor-performance)   
      - [12. Miss penalty and out-of-order execution processor](#12-miss-penalty-and-out-of-order-execution-processor)   
      - [13. Six Basic Cache Optimizations](#13-six-basic-cache-optimizations)   
      - [14 Summary of basic cache optimizations](#14-summary-of-basic-cache-optimizations)   
      - [15 Ten advanced optimizations of cache performance](#15-ten-advanced-optimizations-of-cache-performance)   
      - [16 Cache optimization summary](#16-cache-optimization-summary)   
      - [17 Main memory Basics](#17-main-memory-basics)   
      - [18 SRAM Technology](#18-sram-technology)   
      - [19 DRAM Technology](#19-dram-technology)   
      - [20  Flash Memory](#20-flash-memory)   
      - [21 Enhancing dependability in memory systems](#21-enhancing-dependability-in-memory-systems)   
      - [22 Virtual Machines](#22-virtual-machines)   
      - [23 I/O cache coherency](#23-io-cache-coherency)   

<!-- /MDTOC -->
## Memory hierarchy
### 1. Incentive of using memory hierarchy  
The increasing gap between the performance of CPU and memory
### 2. Designers' target
Traditionally, the average memory access time. For PMD, power efficiency is more important.
### 3. When to write cache data into memory
**Write through** VS **Write back**
### 4. Categories of cache misses
- Compulsory
- Capacity
- Conflict
- Coherency

### 5. Relationship between miss rate and misses per instruction
![alt text](data/equation1.png)
### 6. CPU execution time and cache misses
![alt text](data/equation2.png)
### 7. Misses per instruction
![alt text](data/equation3.png)  
### 8. Pros and cons of Misses per instruction
- Pros: Independent of hardware implementation. Eg: Speculative fetch will decrease the miss rate if measured by misses per memory access
- Cons: Dependent on architecture. Hard to compare this index between different instruction sets.  

### 9. Basic questions about cache
- Block placement
  - Direct mapped
  - Fully associative
  - n-way Set associative
- Block identification
  - Tag bit
  - Valid bit
- Block replacement (strategies for set/fully associativity)
  - Random
  - LRU
  - FIFO (Pseudo FIFO)
- Write strategy
  - Difference from Read: The block can be read from the cache at the same time that the tag is read and compared
  - Options when writing to the cache:
    - Write-through:
      - Easier to implement
      - Simplifies data coherency
    - Write-back:
      - Use less bandwidth, so attractive to multi-processors
      - Saves power, attractive to embedded devices
    - dirty bit
  - Options when a write miss:
    - Write allocate (usually coupled with write-back)
    - No-write allocate (usually coupled with write-through)  

### 10. Average memory access time
![alt text](data/equation4.png)

### 11. Average Memory Access Time and Processor performance
- There are other reasons for CPU stall besides cache miss.
- Depending on the type of CPU. For the in-order execution CPU, the answer is yes.

![alt text](data/equation5.png)

![alt text](data/equation6.png)   

### 12. Miss penalty and out-of-order execution processor
![alt text](data/equation7.png)

### 13. Six Basic Cache Optimizations
- First Optimization: Large Block size to reduce miss rate
  - Reduce Compulsory misses
  - Increase the miss penalty
  - Increase the conflict misses
  - Popular block size: 32 bytes for 4 KB & 64 bytes for larger cache
- Second Optimization: Larger cache to reduce the miss rate
- Third Optimization: Higher associativity to reduce miss rate
  - 2:1 Cache rule of thumb: a direct-mapped cache of size N has the same miss rate as a two-way set associative cache of size N/2
- Forth Optimization: Multilevel caches to reduce miss penalty
  - Question about cache: Should I make the cache faster to keep pace with the speed of processors, or make the cache larger to overcome the widening gap between the processor and main memory?
  - ![alt text](data/equation8.png)
  - ![alt text](data/equation9.png)
  - For second-level caches, there are many fewer hits than in the first-level cache, so the emphasis shifts to fewer misses. This insight leads to much larger caches and techniques to lower the miss rate, such as higher associativity and larger blocks.
- Fifth Optimization: Giving priority to read misses over writes to reduce miss penalty
  - To solve read-after-write data hazard.
  - Checks the contents of the write buffer on a read miss, and if there are no conflicts and the memory system is available, let the read miss continue.
- Sixth Optimization: Avoiding address translation during indexing of the cache to reduce hit time
  - Physical cache and virtual cache
  - Indexing the cache and comparing addresses with physical address or virtual address.
  - Pros and cons of full virtual addressing for both indices and tag:
    - Pros: Eliminates address translation time from a cache hit.
    - Cons:
      - Page protection
      - Cache is flushed after switching the process
      - OS and user programs may use two different virtual addresses for the same physical address. Hardware solutions called antialiasing guarantee every cache block a unique physical address. Software methods like page coloring can also make this problem easier to handle.
      - IO usually uses physical addresses and this would require mapping to virtual addresses to interact with a virtual cache.
   - virtually indexed and physically tagged. However, the limitation of this method is that a direct-mapped cache can be no bigger than the page size. If we introduce the associativity in this approach, a bigger cache will be allowed.

### 14 Summary of basic cache optimizations
Technique | Hit time | Miss penalty | Miss rate | Hardware complexity | comment
--- | --- | --- | --- | --- | --- |
Larger block size | |-|+|0|Trival;Pentium 4 uses it
Larger cache size | - | | + |1|Widely used, especially for L2 caches
Higher associativity |-||+|1|Widely used
Multilevel cache||+||2|Costly hardware; harder if L1 block size != L2 block size; widely used
Read priority over writes||+||1|Widely used
Avoiding address translation during cache indexing | +|||1|Widely used

### 15 Ten advanced optimizations of cache performance
  - Categories of cache performance optimizations:
    - Reducing the hit Time
    - Increasing cache bandwidth
    - Reducing the miss penalty
    - Reducing the miss rate
    - Reducing the miss penalty or miss rate via parallelism
  - First optimization: Small and simple first-level caches to reduce hit time and power
    - The pressure of both a fast clock cycle and power limitations encourages limited size for first-level caches.
    - One approach to determining the impact on hit time and power consumption in advance of building a chip is to use CAD tools, such as CACTI.
    - Recently three factors have led to the use of higher associativity in first-level caches:
      - Many processors take at least two clock cycles to access the cache and thus the impact of a longer hit time may not be critical
      - Almost all L1 caches should be virtually indexed to keep the TLB out of the critical path. This limits the size of the cache to the page size times the associativity.
      - With the introduction of multithreading, conflict misses can increase, making higher associativity more attractive.
  - Second optimization: way prediction to reduce hit time
    - Way prediction and way access
  - Third optimization: pipelined cache access to increase cache bandwidth
  - Fourth optimization: nonblocking caches to increase cache bandwidth
    - hit under miss and hit under multiple miss
    - Difficult to evaluate the performance: a cache does not necessarily stall the processor
  - Fifth optimization: multibanked caches to increase cache bandwidth
    - Help to reduce the power as well.
  - Sixth optimization: critical word first and early restart to reduce miss penalty
  - Seventh optimization: merging write buffer to reduce miss penalty
    - Note that input/output device registers are often mapped into the physical address space. These I/O addresses cannot allow wirte merging.
  - Eighth optimization: compiler optimizations to reduce miss rate
    - Some examples: loop interchange & blocking
  - Ninth optimization: hardware prefetching of instructions and data to reduce miss penalty or miss rate.
  - Tenth optimization: compiler-controlled prefetching to reduce miss penalty or miss rate
    - Compilers to insert prefetch instructions to request data before the processor needs it.
    - Register prefetch and cache prefetch
    - faulting and non-faulting
    - Software pipelining

### 16 Cache optimization summary
Technique | Hit time | Bandwidth | Miss penalty | Power consumption | Hardware cost/complexity | comment
--- | --- | --- | ---| --- | --- | --- |     
Small and simple cache | + | |-|+|0|Trivial; widely used
Way-predicting caches|+|||+|1|Used in Pentium 4
Pipelined cache access|-|+|||1|Widely used
Nonblocking caches||+|+||3|Widely used
Banked caches|+|||+|1|Used in L2 of both i7 and Cortex-A8
Critical word first and early restart|||+||1|Widely used with write through
Compiler techniques to reduce cache misses|||+||0|Software is challenging but many compilers handle common linear algebra calculations
Hardware prefetching of instructions and data||+|+|-|2 instr. 3 data | Most provide prefetch instructions
Compiler-controlled prefetching||+|+||3|Needs nonblocking cache; possible instruction overhead; widely used

### 17 Main memory Basics
  - Main memory latency is the primary concern of the cache, while main memory bandwidth is the primary concern of multiprocessors and I/O.
  - access time: time between when a read is requested and when the desired word arrives
  - Cycle time: the minimum time between unrelated requests to memory

### 18 SRAM Technology
  - No refresh
  - Six transistors per bit
  - Around three to five times faster than accessing DRAM memory

### 19 DRAM Technology
  - One single transistor to store data. Must restore data after reading
  - Refresh data periodically meaning that DRAM is occasionally unavailable
  - Dual inline memory modules (DIMMs)
  - Improvements in DRAM performance:
    - Adding timing signals that allow repeated accesses to the row buffer without another row access time.
    - Synchronous DRAM (SDRAM)
    - Double data rate (DDR)
  - Naming of DRAM

    Standard | Clock rate (MHz) | M transfers per second | DRAM name | MB/sec/DIMM | DIMM name
    --- |---|---|---|---|---|
    DDR3 | 533 | 1066 | DDR3-1066 | 8528 | PC8500  

  - Graphics Data RAMs
    - Because of the lower locality of memory request in a GPU, burst mode generally is less useful for a GPU, but keeping open multiple memory banks and managing their use improves effective bandwidth.
  - Reducing Power
    - Voltage drops
    - Power down mode

### 20  Flash Memory
  - A type of Electronically Erasable Programmable Read-Only Memory
  - Differences between Flash and DRAM:
    - Flash memory must be erased before writes and it is erased in blocks.
    - Flash memory is static and draws significantly less power when not reading or writing.
    - Each block has limited write cycles.
    - Flash is cheaper and slower than SDRAM and more expensive and faster than disk.

### 21 Enhancing dependability in memory systems
  - Soft errors and hard errors
  - Error correcting codes (ECCs). Usually ECC can detect two errors and correct a single error with a cost of eight bits of overhead per 64 data bits.
  - Chipkill instead of ECC is used by a very large memory system.

### 22 Virtual Machines
 - System virtual machine
 - Virtual machine monitor
 - Qualitive requirements of a virtual machine monitor:
   - Guest software should behave on a VM exactly as if it were running on the native hardware, except for performance-related behavior or limitations of fixed resources shared by multiple VMs
   - Guest software should not be able to change allocation of real system resources directly
 - A problem: lack of instruction set architecture support for virtual machines
 - Shadow page table: mapping directly from the guest virtual address space to the physical address space of the hardware
 - Paravirtualization: allowing small modifications to the guest OS to simplify virtualization. An example is Xen VMM. In VMM: driver domains and guest domains.

### 23 I/O cache coherency
  - Usually use write-through between I/O device and memory
  - Usually ensures that no blocks of the input buffer are in the cache

### Apendix:
  - Memory accesses consist of instruction accesses and data accesses.
