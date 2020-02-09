# Memory Hierarchy
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
