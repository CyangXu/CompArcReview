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
  - Vector Mask Registers: handling IF statements in vector loops
    - One difference between vector processors and GPUs: how they handle conditional statements:
      - vector processor make the mask registers part of the architectural state and rely on compilers to manipulate mask registers explicitly; GPUs utilize the hardware to manipulate internal mask registers that are invisible to GPU software.
  - Memory banks: supplying bandwidth for vector load/store units
    - Reasons:
      - Many vector computers support multiple loads or stores per clock, and the memory bank cycle time is usually several times larger than the processor cycle time.
      - Most vector processors support the ability to load or store data words that not sequential.
      - Most vector computers support multiple processors sharing the same memory system, so each processor will be generating its own independent stream of addresses.
  - Gather-scatter:
    - gather: Taking an index vector and fetching the vector whose elements are at the addresses given by adding a base address to the offsets given in the index vector.
    - scatter
    - Gather-scatter runs more slowly than the non-indexed loads or stores: each element has an individual address so they cannot be handled in groups.

### 3. SIMD instruction set extensions for multimedia
  - Background: SIMD extensions started with the simple observation that many media applications operate on narrower data types than the 32-bit processors were optimized for.
  - Difference between vector processors and SIMD extensions:
    - SIMD extensions fix the number of data operands in the opcode, which has led to the addition of hundreds of instructions in the MMX, SSE and AVX extensions of the x86 architecture.
    - SIMD does not address the issue like strided access or gather-scatter access.
    - SIMD does not offer the mask registers to support conditional execution.
  - Advantages of SIMD extensions:
    - Cost little to add to the standard arithmetic unit.
    - Don't have to rely on a lot of memory bandwidth.
    - Do not need to deal with page fault. SIMD extensions use separate data transfers per SIMD group of operands that are aligned in memory, and so they cannot cross page boundaries.
  - Programming multimedia SIMD architectures:
    - The easiest way to use SIMD instructions is through libraries or by writing in assembly language.
    - Some compilers are able to generate SIMD instruction automatically. But there is some restrictions, such as data alignment in memory.
  - Roofline visual performance model
    - arithmetic intensity: the ratio of floating-point operations per byte transferred to main memory during program execution.
    - Roofline model: the vertical Y-axis is achievable floating-point performance (GFLOP/sec); the horizontal X-axis arithmetic intensity.
### 4. Graphics processing units
  - Basic CUDA programming:
    - \__device\__
    - \__global\__
    - \__host\__
    - name<<<dimGrid, dimBlock>>>(...parameter list...)
    - identifiers: blockIdx, threadIdx, blockDim
  - Example CUDA code:
    - DAXPY
    ~~~
    __host__
    int nblocks = (n + 255) / 256;
    daxpy<<<nblocks, 256>>>(n, 2.0, x, y);
    __device__
    void daxpy(int n, double a, double *x, double *y) {
      int i = blockIdx.x*blockDim.x + threadIdx.x;
      if(i < n) y[i] = a*x[i] + y[i];
    }
    ~~~
  - NVIDIA GPU Computational Structures
    - Terms

    More descriptive name | Closest old term outside of GPUs | CUDA term
    --- | --- | --- |
    Vectorizable loop | Vectorizable loop |Grid
    Body of vectorizable loop |  Body of a vectorizable loop | Thread block
    Sequence of SIMD lane operations | One iteration of a scalar loop | CUDA thread
    A thread of SIMD instructions | Thread of vector instructions | Warp
    SIMD instruction | vector instruction | PTX instruction
    Multithreaded SIMD processor | Vector processor | Streaming multiprocessor
    Thread block scheduler | Scalar processor | Giga thread engine
    SIMD thread scheduler | Thread scheduler in a multithreaded CPU | Warp scheduler
    SIMD lane | vector lane | thread processor
    GPU memory | main memory | global memory
    private memory | stack or thread local storage | local memory
    local memory | local memory | shared memory
    SIMD lane registers | vector lane registers | thread processor registers

    - An example to explain how GPU works.
      - Multiply two vectors together, each 8192 elements long. The GPU code that works on the whole 8192 element multiply is called a Grid. A grid can be composed of thread blocks, each with up to 512 elements. So thread block size is 8192 / 512. A thread block is assigned to a processor that executes that code (or called multithreaded SIMD processor) by the thread block scheduler. SIMD processors are full processors with separate PCs and are programmed using threads.
      - GPU is a multipleprocessor composed of multithreaded SIMD processors.
      - Machine object that the hardware creates, manages, schedules, and executes is a thread of SIMD instructions. These SIMD instruction threads run on a multithreaded SIMD processor. The SIMD scheduler arranges SIMD instruction threads on a multithreaded SIMD processor (with help from a scoreboard).
      - SIMD instructions of SIMD instruction thread are 32 wide. So one thread block in the example contains 512/32 = 16 SIMD threads.
      - SIMD processor have parallel functional units to perform the operation. These units are called SIMD lanes.
      - Threads of SIMD instructions should be independent. The SIMD thread scheduler can pick whatever thread of SIMD instructions is ready and need not wait for the next SIMD instruction within a thread.
      - In the example, each multithreaded SIMD processor must load 32 elements of two vectors from memory into registers, perform the multiply by reading and writing registers and store the product back from register into memory.
      - NVIDIA GPU instruction set architecture
        - PTX (parallel thread execution) uses virtual registers, so the compiler figures out how many physical vector registers a SIMD thread needs.
        - Different from x86 microarchitectures, PTX is translated in software and load time on a GPU.
        - format of a PTX instruction:
          opcode.type d, a, b, c
        - Type | .type specifier
          --- | --- |
          untyped bits 8, 16, 32, and 64 bits | .b8 .b16 .b32 .b64
          unsigned integer 8, 16, 32, and 64 bits | .u8 .u16 .u32 .u64
          signed integer 8, 16, 32, 64 | .s8 .s16 .s32 .s64
          floating point 16, 32, 64 | .f16 .f32 .f64
        - Unlike vector architectures, GPUs don't have separate instructions for sequential data transfers, strided data transfers, and gather-scatter data transfers. All data transfers are gather-scatter.
        - Address coalescing hardware to recognize when the SIMD lanes within a thread of SIMD instructions are collectively issuing sequential addresses.
    - Conditional branching in GPUs
      - GPU hardware provides each SIMD thread with its own stack; a stack entry contains an identifier token, a target instruction address and a target thread-active-mask.
      - There are special instructions that push stack entries for a SIMD thread and special instructions that pop a stack entry or unwind the stack to a specified entry.
      - GPU hardware instructions also have individual per-lane predication specified with a 1-bit predicate register for each lane.
      - __branch diverge__: A more complex control flow results in a mixture of predication and GPU branch instructions with special instructions and markers that use the branch synchronization stack to push a stack entry when some lanes branch to the target address, while others fall through.
      - If the PTX assembler generates predicated instructions with no GPU branch instructions, it uses a per-lane predicate register to enable or disable each SIMD Lane for each instruction.
      - The PTX assembler sets a "branch synchronization" marker on appropriate conditional branch instructions that pushes the current acive mask on a stack inside each SIMD thread.
   - NVIDIA GPU memory structures
     - Each SIMD lane in a SIMD processor is provided with __private memory__.
     - The on-chip memory that is local to each SIMD processor is called __local memory__.
     - The off-chip DRAM shared by the whole GPU and all thread blocks is called __GPU memory__.
   - Comparison between vector architectures and GPUs
     - Terms
       vector term | closest CUDA/NVIDIA GPU term
       --- | --- |
       vectorized loop | Grid
       Chime | NA
       Vector instruction | PTX instruction
       Gather / scatter | global load/stored
       Mark register | predicate registers and internal mask register
       vector processor | multithreaded SIMD processor
       control processor | thread block scheduler
       scalar processor | system processor
       vector lane | SIMD lane
       vector register | SIMD lane register
       main memory | GPU memory

     - Different ways to hide memory latency:
       - vector architecture: deeply pipelined access
       - GPU: multithreaded
     - Handle condition:
       - vector architecture: by software
       - GPU: by hardware 
