# Virtual Memory

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [Virtual Memory](#virtual-memory)
  - [1. Difference between cache and virtual memory](#1-difference-between-cache-and-virtual-memory)
  - [2. Paging versus segmentation](#2-paging-versus-segmentation)
  - [3. Four memory hierarchy questions revisited](#3-four-memory-hierarchy-questions-revisited)
  - [4. Techniques for fast address translation](#4-techniques-for-fast-address-translation)

<!-- /MDTOC -->

## 1. Difference between cache and virtual memory

  Parameter | First-level cache | Virtual memory
  --- | --- | --- |
  Block(page) size | 16-128 bytes | 4096-65536 bytes
  Hit time | 1-3 clock cycles | 100-200 clock cycles
  Miss penalty | 8-200 clock cycles | 1 million - 10 million clock cycles
  access time | 6-160 clock cycles | 0.8 million - 8 million clock cycles
  transfer time | 2-40 clock cycles | 0.2 million - 2 million clock cycles
  Miss rate | 0.1-10% | 0.00001-0.001%
  Address mapping | 25-45-bit physical address to 14-20 cache address | 32-64-bit virtual address to 25-45-bit physical address

- Replacement on cache misses is primarily controlled by hardware, while virtual memory replacement is primarily controlled by the operating system.
- The size of the processor address determines the size of virtual memory, but the cache size is independent of the processor address size.
- In addition to acting as the lower-level backing store for main memory in the hierarchy, secondary storage is also used for the file system.

## 2. Paging versus segmentation

  |Metric|Page|Segment|
  ---|---|---|
  Words per address |  One | Two (segment and offset)|
  Programmer visible? | Invisible to application programmer | May be visible to application programmer |
  Replacing a block | Trivial (all blocks are the same size) | Difficult (Must find contiguous, variable-size, unused portion of main memory) |
  Memory use inefficiency | Internal fragmentation (unused portion of page) | External fragmentation (unused pieces of main memory) |
  Efficient disk traffic | Yes (adjust page size to balance access time and transfer time) | Not always (small segments may transfer just a few bytes) |

## 3. Four memory hierarchy questions revisited

- Where can a block be placed in main memory
  - Answer: Fully-associative
- How is a block found if it is in main memory
  - Page tablet and the inverted page table.
- Which block should be replaced on a virtual memory miss?
  - LRU
- What happens on a write
  - Always write-back.

## 4. Techniques for fast address translation
