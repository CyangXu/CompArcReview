## Thread-level parallelism (TLP)
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
 - Coherence miss:
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
