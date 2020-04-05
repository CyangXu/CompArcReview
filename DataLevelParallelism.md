## Data-Level Parallelism

### 1. Introduction
  - How wide a set of applications has significant data-level parallelism:
    - Matrix-oriented computations
    - Media-oriented image and sound processing
  - SIMD is usually more energy efficient than MIMD
  - Variations of SIMD:
    - Vector architectures
      - Oldest compared to other two.
      - Expensive due to DRAM bandwidth.
    - Multimedia SIMD instruction set extensions
    - graphics processing units (GPU)
      - heterogeneous

### 2. Vector architecture
  - Vector architectures grab sets of data elements scattered about memory, place them into large, sequential register files, operate on data in those register files, and then disperse the results back into memory.
  - VMIPS architecture components:
    - vector registers: A fixed-length bank holding a single vector.
    - vector functional units: Each unit is fully pipelined, and it can start a new operation on every clock cycle.
    - vector load/store unit: The vector memory unit loads or stores a vector to or from memory.
    - A set of scalar registers.
  - How vector processors work
    - The vector processor greatly reduces the dynamic instruction bandwidth.
    - VMIPS has less interlocks.
  - Vector execution time:
    - Depends on three things:
      - The length of the operand vector
      - Structural hazards among the operations
      - The data dependence
    - Convoy: the set of instructions that could potentially execute together.
    - Chaining: Allow a vector operation to start as soon as the individual elements of its vector source operand become available. In practice, we often implement chaining by allowing the processor to read and write a particular vector register at the same time.
    - Flexible chaining: Allow a vector instruction to chain to essentially any other active vector instruction, assuming there is no structural hazard.
    - Chime: unit of time taken to execute one convoy. The chime approximation ignores some processor-specific overheads. Another source of overhead ignored by the chime model is vector start-up time.
  - Multiple Lanes: beyond one element per clock cycle
    - Each lane contains one portion of the vector register file and one execution pipeline from each vector functional unit.
  - Vector-length registers (VLRs): handling loops not equal to 64
    - Strip mining: handle the situation that n may be greater than the maximum vector length (MVL)
