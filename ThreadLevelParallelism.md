## Thread-level parallelism (TLP)

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [Thread-level parallelism (TLP)](#thread-level-parallelism-tlp)   
      - [1 Introduction](#1-introduction)   
      - [2 Multiprocessor architecture: issues and approach](#2-multiprocessor-architecture-issues-and-approach)   
      - [3 Challenges of parallel processing](#3-challenges-of-parallel-processing)   
      - [4 Cache coherence issue](#4-cache-coherence-issue)   
      - [5 Snooping Coherence Protocol](#5-snooping-coherence-protocol)   
      - [6 Implementing snooping cache coherence](#6-implementing-snooping-cache-coherence)   
      - [7 Performance of symmetric shared-memory multiprocessors](#7-performance-of-symmetric-shared-memory-multiprocessors)   
      - [8 Distributed shared-memory and director-based coherence](#8-distributed-shared-memory-and-director-based-coherence)   
      - [9 Directory-based cache coherence protocols](#9-directory-based-cache-coherence-protocols)   
      - [10 Synchronization: the basics](#10-synchronization-the-basics)   
      - [11 Models of memory consistency: an introduction](#11-models-of-memory-consistency-an-introduction)   
      - [12 Crosscutting issues](#12-crosscutting-issues)   
      - [13 Fallacies and pitfalls](#13-fallacies-and-pitfalls)   

<!-- /MDTOC -->

### 1 Introduction
  - The view that advances in uniprocessor architecture were nearing an end has been held by some researchers for many years (even in 1970s). This view was premature before 1990s. Nonetheless, the importance of multiprocessors was growing after 1990s by the following reasons:
    - The dramatically lower efficiencies in silicon and energy use that were encountered between 2000 and 2005 as designers attempted to find and exploit more ILP
    - The insight that desktop is less important since the complex computation can be done by cloud computing.
    - Large datasets, individual requests are bringing more natural TLP.
  - Definition of multiprocessors here: Computers consisting of tightly coupled processors whose coordination and usage are typically controlled by a single operating system and that share memory through a shared memory address space:
    - Parallel processing
    - Request level parallelism
      - multiprogramming
  - Multiprocessor and multicore
  - Multiprocessor -> cluster -> warehouse-scale computers (as size grows)

### 2 Multiprocessor architecture: issues and approach
   - Important difference between ILP and TLP: TLP is identified at a high level by the software system or programmer and that the threads consist of hundreds to mullions of instructions that may be executed in parallel.
   - The amount of computation assigned to one thread is called grain size.
   - Multiprocessor Categories:
     - Symmetric multiprocessors (SMPs) or centralized shared-memory multiprocessors. SMP is also sometimes called uniform memory access (UMA).
     - Distributed shared memory (DSM). It is also called nonuniform memory access (NUMA).
   - Multiprocessor architecture shares address space, which is different from the cluster and warehouse-scale computer where the memory of one processor cannot be accessed by another processor without the assistance of software protocols running on both processors.

### 3 Challenges of parallel processing
  - Limited parallelism available in programs. Example:
    - For achieving a speedup of 80 with 100 processors, what fraction of the original computation can be sequential: Based on the Amdahl's law, the sequential part could only be 0.25%.
    - Large latency of remote access in a parallel processor.

### 4 Cache coherence issue
  - Originally processors were all single-core and often took an entire board, and memory was located on a shared bus. Now microprocessors directly connect memory to a single chip, which is sometimes called a backside or memory bus to distinguish it from the bus used to connect to I/O.
  - Multiprocessor usually supports the caching of both shared and private data:
    - For private data, the program behavior is identical with that in a uniprocessor.
    - For shared data, the shared value may be replicated in multiple caches.
  - Cache coherence problem: A memory system is coherent if
    - 1. A read by processor P to location X that follows a write by P to X, with no writes of X by another processor occurring between the write and the read by P, always returns the value written by P.
    - 2. A read by a processor to location X that follows a write by another processor to X returns the written value if the read and write are sufficiently separated in time and no other writes to X occur between the two accesses.
    - 3. Writes to the same location are serialized; that is, two writes to the same location by any two processor are seen in the same order by all processors. For example, if the values 1 and then 2 are written to a location, processors can never read the value of the location as 2 and then later read it as 1. (also called write serialization issue)
  - Cache data migration and replication.
  - Cache coherence protocol:
    - Directory based
    - Snooping

### 5 Snooping Coherence Protocol
  - Category:
    - Write invalidate protocol: exclusive access ensures that no other readable or writable copies of an item exist when the write occurs and all other cached copies of the item are invalidated.
    - Write update protocol: update all of copies when write occurs.
  - For writes, we would like to know whether any other copies of the block are cached because, if there are no other cached copies, then the write need not be placed on the bus in a write-back cache.
  - Each bus transaction must check the cache-address tags, which could potentially interfere with processor cache accesses. Two solutions:
    - Duplicate the tags and have snoop accesses directed to the duplicate tags.
    - Use a directory at the shared L3 cache; the directory indicates whether a given block is shared and possibly which cores have copies. It requires that L3 must always have a copy of any data item in L1 or L2, a property called inclusion.
  - Simple protocol (also called MSI protocol):
    - Three states: invalid, shared, modified.
    - Protocol details:

    Request | Source | State of addressed cache block | Type of cache action | Function and explanation
    --- | --- | --- | --- | --- |
    Read hit | Processor | Shared or modified | Normal hit | Read data in local cache
    Read miss | Processor | Invalid | Normal miss | Place read miss on bus
    Read miss | Processor | Shared | Replacement | Address conflict miss: place read miss on bus
    Read miss | Processor | Modified | Replacement | Address conflict miss: write-back block, then please read miss on bus
    Write hit | Processor | Modified |  Normal hit | Write data in local cache
    Write hit | Processor | Shared | Coherence | Place invalidate on bus. These operations are often called upgrade or ownership misses, since they do not fetch the data but only change the state.
    Write miss | Processor | Invalid | Normal miss | Place write miss on bus
    Write miss | Processor | Shared | Replacement | Address conflict miss: place write miss on bus
    Write miss | Processor | Modified | Replacement | Address conflict miss: write-back bock, then place write miss on bus
    Read miss | Bus | Shared | No action | Allow shared cache or memory to service read miss.
    Read miss | Bus | Modified | Coherence | Attempt to share data: Place cache block on bus and change state to shared.
    Invalidate | Bus | Shared | Coherence | Attempt to write shared block; invalidate the block.
    Write miss | Bus | Shared | Coherence | Attempt to write shared block; invalidate the cache block
    Write miss | Bus | Modified | Coherence | Attempt to write block that is exclusive elsewhere; write-back the cache block and make its state invalid in the local cache.
  - Drawback of the simple protocol: The protocol assumes that operations are atomic,
  which means that an operation can be done in such a way that no intervening operation can occur.
  - Many dual-processor chips, including the Intel Xeon and AMD Opteron, supported multichip multiprocessors that could be built by connecting a high-speed interface (called Quickpath or Hypertransport).
  - A multiprocessor built with multiple multicore chips will have a distributed memory architecture and will need an interchip coherency mechanism above the one within the chip.
  - Extensions to the basic coherence protocol
    - MESI protocol & MESIF protocol (used by Intel i7)
    - MOESI protocol
  - How to increase the snooping bandwidth
    - multiple buses as well as interconnection networks
    - memory is directly connected to each multicore chip. Meanwhile, using the point-to-point links to broadcast up to three other chips. Used by AMD Opteron.

### 6 Implementing snooping cache coherence
  - Challenge: Steps including detecting a write or upgrade miss, communicating with the other processors and memory, cannot be done in single cycle.
  - In a single multicore chip, these steps can be made effectively atomic by arbitrating for the us to the shared cache or memory first and not releasing the bus until all actions are complete.
  - In a system without a bus, there must be methods of making the steps in a miss atomic, such as serializing writes.

### 7 Performance of symmetric shared-memory multiprocessors
 - Coherence miss [section 5.3]:
   - True sharing misses
   - False sharing misses:
     - It occurs when a block is invalidated because some word in the block, other than the one being read, is written into. Miss would not occur if the block size were a single word.
 - Experiment on OLTP benchmark and AlphaServer 4100
   - The execution time is improved as the L3 cache grows due to the reduction in L3 misses. Almost all of the gain occurs in going from 1 to 2 MB, with little additional gain beyond that. Because the further growth in cache size fails to reduce the L3 cache misses introduced by coherence misses.
   - Increasing the processor count also escalates the coherence misses.
   - Increasing the block size leads to:
     - The true sharing miss rate decreases
     - The compulsory miss rate significantly decreases.
     - The conflict/capacity misses show a small decrease, indicating that the spatial locality is not high in the uniprocessor misses.
     - False sharing miss rate nearly doubles.
  - Experiment on a multiprogramming and OS workload
    - Why OS has a higher cache miss rate:
      - The kernel initializes all pages before allocating them to a user.
      - Kernel usually shares data and thus has a nontrivial coherence miss rate.
    - Increasing L1 data cache causes the user miss rate to decrease proportionately more than the kernel miss rate.
    - Increasing the block size improves the kernel miss rate more significantly than the user miss rate.
    - If examining the number of bytes needed per data reference, we see that the kernel has a higher traffic ration that grows with block size. For example, when going from a 16-byte block to a 128-byte block, the miss rate drops by about 3.7, but the number of bytes transferred per miss increase by 8.

### 8 Distributed shared-memory and director-based coherence
  - A directory keeps the state of every block that may be cached. The information includes which caches have copies of the block, whether it is dirty, and so on.
  - Simplest implementation: It associates an entry in the directory with each memory block. The amount of information is proportional to the product of the number of memory blocks (where each block is the same size as the L2 or L3 cache block) times the number of nodes, where a node is a single multicore processor or a small collection of processors that implements coherence internally.
  - Downside of the simple implementation: Not work for supercomputer-sized systems.

### 9 Directory-based cache coherence protocols
  - Track the state of each potentially shared memory block:
    - Shared - One or more nodes have the block cached, and the value in memory is up to date.
    - Uncached - No node has a copy of the cache block.
    - Modified - Exactly one node has a copy of the cache block, and it has written the block. Memory copy is out of data. The processor is called the owner of the block.
  - Track which nodes have copies of that block
    - Simplest way is to keep a bit vector for each memory block.
    - Local node: the node where a request originates
    - Home node: the node where the memory location and the directory entry of an address reside.
    - Messages between the processors and the directory:

    |Message Type | Source | Destination | Message contents | Function of this message
    | --- | --- | --- | --- | --- |
    Read miss | Local cache | Home directory | P, A | Node P has a read miss at address A; request data and make P a reader sharer.
    Write miss | Local cache | Home directory | P, A | Node P has a write miss at address A; request data and make P the exclusive owner.
    Invalidate | Local cache | Home directory | A | Request to send invalidates to all remote caches that are caching the block at address A.
    Invalidate | Home directory | Remote cache | A | Invalidate a shared copy of data at address A.
    Fetch | Home directory | Remote cache | A | Fetch the block at address A and sent it to its home directory; change the state of A in the remote cache to shared.
    Fetch / Invalidate | Home directory | Remote cache | A | Fetch the block at address A and send it to its home directory; invalidate the block in the cache.
    Data value reply | Home directory | Local cache | D | Return a data value from the home memory.
    Data write-back | Remote cache | Home directory | A, D | Write-back a data value for address A.

    - Note that the directory must be accessed when the home node is the local node, since copies may exist in yet a third node, called a remote node.
    - The write miss operation, which was broadcast on the bus in the snooping scheme, is replaced by the data fetch and invalidate operations that are selectively sent by the director controller.
    - The state transition of directory:
      - Two requests for uncached block:
        - Read miss
        - Write miss
      - Two requests for shared block:
        - Read miss
        - Write miss
      - Three requests for exclusive block:
        - Read miss
        - Data write-back
        - Write miss
    - Note that the protocol above is simplified: nonatomic memory transaction should be dealt with.
    - Optimization in commercial CPU: when a read or write miss occurs for a exclusive block, instead of sending the block to the directory at the home node then storing into the home memory and sending to the original requesting node, the data is forwarded from the owner node the requesting node directly.

### 10 Synchronization: the basics
  - Basic hardware primitives:
    - It is the key ability of synchronization.
    - Example atomic primitives:
      - Atomic exchange: interchanges a value in a register for a value in memory
      - Fetch and increment
    - Implementation challenge: Require both a memory read and a write in a single, uninterruptible instruction. It is harder when dealing with cache coherence issue.
    - An alternative way: Have a pair of instructions where the second instruction returns a value which it can be deduced whether the pair of instructions was executed as if the instructions were atomic.
      - load linked (link register)
      - store conditional
      - Example: LL and SC implement the exchange operation:
        ~~~
        try: MOV R3, R4    ;mov exchange value
             LL  R2, 0(R1) ;load linked
             SC  R3, 0(R1) ;store conditional
             BEQZ R3, try  ;branch store fails
             MOV  R4, R2   ;put load value in R4
        ~~~
    - Implementing locks using coherence:
      - Spin lock:
        - Simplest implementation (without cache coherence): keep the lock variables in memory. Then it can use the atomic exchange.
        - With cache coherence in consideration: Cache the locks. However, it cannot use atomic exchange directly since each attempt to exchange the lock variable requires a write operation. New method should be: the spin lock procedure spins by doing reads on a local copy of the lock until it successfully sees that the lock is available; then it attempts to acquire the lock by doing a swap operation.

        ~~~
        lockit:  LDR2,    0(R1)      ;load of lock
                 BNEZR2,  lockit     ;not available-spin
                 DADDIUR2,R0,#1      ;load locked value
                 EXCHR2,  0(R1)      ;swap
                 BNEZR2,  lockit     ;branch if lock wasn't 0
        ~~~
       - An example to explain why the spin lock code above works:
       
        | Step| P0  | p1  | p2  |  Coherence state of lock at end of step | Bus/directory activity|
         | --- | --- | --- | --- | --- | --- |
         | 1   | Has lock | Begins spin, testing if lock = 0 | Begins spin, testing if lock = 0 | Shared | Cache misses for P1 and P2 satisfied in either order. Lock state becomes shared |
         | 2   | Set lock to 0 | Invalidated received | Invalidated received | Exclusive (p0) | Write invalidate of lock variable from P0
         | 3 | | Cache miss | Cache miss | Shared | Bus/directory services P2 cache miss; write-back from P0; state shared
         | 4 | | Waits while bus/directory busy | Lock = 0 test succeeds | shared | Cache miss for P2 satisfied  
         | 5 || Lock = 0 | Execute swap, gets cache miss | shared | Cache miss for P1 satisfied
         | 6 ||Execute swap, gets cache miss | completes swap; return 0 and sets lock = 1 | Exclusive(P2) | Bus/directory services P1 cache miss; send invalidate and generates write-back from P2
         |7||Swap completes and return 1, and sets lock = 1|Enter critical section | Exclusive P1 | Bus/directory services P1 cache miss; sends invalidate and generate write-back from P2
         |8||Spins, testing if lock = 0 | | |None

       - How the new spin lock code utilizes the cache coherence:
         - When multiple processes try to lock a variable using an atomic swap, if one process with the lock stores a 0 into the lock, all other caches are invalidated and must fetch the new value to update their copy of lock. After update, other processes will find that lock is already taken.

### 11 Models of memory consistency: an introduction
  - What properties must be enforced among reads and writes to different locations by different processors?
  - Sequential consistency: The result of any execution is the same as if the memory accesses executed by each processor were kept in order and the accesses among different processors were arbitrarily interleaved. However, it hurts the performance.
  - Two ways to improve the performance:
    - Develop ambitious implementations that preserve sequential consistency but use latency-hiding techniques to reduce the penalty.
    - Develop less restrictive memory consistency models that allow for faster hardware.
  - The use of standard synchronization primitives ensures that even if the architecture implements a more relaxed consistency model than sequential consistency, a synchronized program will behave as if the hardware implemented sequential consistency.
  - Relaxed consistency models: basics
    - Relaxing W->R ordering yields a processor consistency model.
    - Relaxing W->W ordering yields a model known as partial store order.
    - Relaxing the R->W and R->R yields a variety of models including weak-ordering, the PowerPC consistency model, and release consistency model.

### 12 Crosscutting issues
  - Compiler optimization the consistency model: One reason for defining a model for memory consistency is to specify the range of legal compiler optimizations that can be performed on shared data.
  - Using speculation to hide latency in strict consistency models. The state of optimization technology and the fact that shared data are often accessed via pointers or array indexing have limited the use of such optimizations.
  - Cache inclusion. Different cache blocks in different cache levels complicates the maintenance of cache inclusion. To achieve it, we must probe the higher levels of the hierarchy when a replacement is done at the lower level to ensure that any words replaced in the lower level are invalidated in the higher level.

### 13 Fallacies and pitfalls
- Pitfall: Measuring performance of multiprocessors by linear speedup versus execution time.
  - First question: the power of the processor being scaled. A program that linearly improves performance to equal 100 Intel Atom processors may be slower than the version run on an eight-core Xeon.
  - Comparing execution time is fair only if you are comparing the best algorithms on each computer.
  - Conclusion: comparing performance by comparing speedups is at best tricky and at worst misleading.
- Fallacy: Linear speedups are needed to make multiprocessors cost effective
  - Cost is not only a function of processors count but also depends on memory, I/O, and the overhead of the system (box, power supply, interconnect, and so on).
- Pitfall: not developing the software to take advantage of, or optimize for, a multiprocessor architecture.
