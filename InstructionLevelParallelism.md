<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [Instruction Level Parallelism](#instruction-level-parallelism)   
      - [1 Two ways of exploiting ILP:](#1-two-ways-of-exploiting-ilp)   
      - [2 CPI for a pipelined processor](#2-cpi-for-a-pipelined-processor)   
      - [3 Major techniques utilizing ILP](#3-major-techniques-utilizing-ilp)   
      - [4 Type of dependences](#4-type-of-dependences)   
      - [5 Data hazards](#5-data-hazards)   
      - [6 Basic compiler techniques for exposing ILP](#6-basic-compiler-techniques-for-exposing-ilp)   
      - [7 Reducing branch costs with advanced branch prediction](#7-reducing-branch-costs-with-advanced-branch-prediction)   
      - [8 Dynamic scheduling](#8-dynamic-scheduling)   
      - [9 Tomasulo Algorithm:](#9-tomasulo-algorithm)   
      - [10 Hardware based speculation:](#10-hardware-based-speculation)   

<!-- /MDTOC -->

## Instruction Level Parallelism
### 1 Two ways of exploiting ILP:
  - Relying on hardware to discover and exploit the parallelism dynamically
  - Relying on software to find parallelism statically at compile time

### 2 CPI for a pipelined processor
![alt text](data/ILP/equation1.png)  

### 3 Major techniques utilizing ILP
Technique | Reduces
--- | --- |
Forwarding and bypassing | Potential data hazard stalls
Delayed branches and simple branch scheduling | Control hazard stalls
Basic compiler pipeline scheduling | Data hazard stalls
Basic dynamic scheduling (scoreboarding) | Data hazard stalls from true dependencies
Loop unrolling | Control hazard stalls
Branch prediction | Control stalls
Dynamic scheduling with renaming | Stalls from data hazards, output dependencies, and anti-dependences
Hardware speculation | Data hazard and control hazard stalls
Dynamic memory disambiguation | Data hazard stalls with memory
Issuing multiple instructions per cycle | Ideal CPI
Compiler dependence analysis, software pipelining, trace scheduling | Ideal CPI, data hazard stalls
Hardware support for compiler speculation | Ideal CPI, data hazard stalls, branch hazard stalls

### 4 Type of dependences
  - Data dependency
    - Coming from data flow through registers
    - Coming from data flow through memory locations. It is hard to detect since two addresses which seem different can actually refer to the same hardware address.
  -  Name dependency. It can be solved by register renaming.
    - Antidependence
    - Output dependence
  - Control dependency

### 5 Data hazards
  - RAW
  - WAW
  - WAR

### 6 Basic compiler techniques for exposing ILP
  - Basic pipeline scheduling and loop unrolling
  - Three different effects limit the gains from loop unrolling:
    - A decrease in the amount of overhead amortized with each unroll
    - Code size limitations
    - Compiler limitations

### 7 Reducing branch costs with advanced branch prediction
  - Correlating branch predictors
    - (m, n) predictor uses the behavior of the last m branches to choose from 2^m branch predictors, each of which is an n-bit predictor for a single branch
  - Tournament predictors: adaptively combining local and global predictors

### 8 Dynamic scheduling
  - Hardware rearranges the instruction execution to reduce the stalls while maintaining data flow and exception behavior.
  - Advantages:
    - It allows code that was compiled with one pipeline in mind to run efficiently on a different pipeline, eliminating the need to have multiple binaries and recompile for a different microarchitecture.
    - It enables handling some cases when dependences are unknown at compile time. For example, they may result from a modern programming environment that uses dynamic linking or dispatching.
    - It allows the processor to tolerate unpredictable delays, such as cache misses, by executing other code while waiting for the miss to resolve.
  - In the classic five-stage pipeline, both structural and data hazards could be checked during instruction decode (ID).
  - Out-of-order execution introduces the possibility of WAR and WAW hazards.
  - Dynamically scheduled processors could generate imprecise exception. An exception is imprecise if the processor state when an exception is raised does not look exactly as if the instructions were executed sequentially in strict program order. Imprecise exceptions make it difficult to restart execution after an exception.
  - To allow out-of-order execution, we essentially split the ID pipe stage of the classic five-stage pipeline into two stages:
    - Issue-Decode instructions, check for structural hazards.
    - Read operands-Wait until no data hazards, then read operations.
  - Multiple functional units and multiple functional units are required.

### 9 Tomasulo Algorithm:
  - Register renaming is provided by reservation stations, which buffer the operands of instructions waiting to issue.
  - The use of reservation stations, rather than a centralized register file, leads to two other important properties. First, hazard detection and execution control are distributed. Second, results are passed directly to functional units from the reservation stations where they are buffered, rather than going through the registers. The bypassing is done with a common result bus that allows all units waiting for an operand to be loaded simultaneously.
  - Data structures that detect and eliminate hazards:
    - Each reservation station has:
      - Op: The operation to perform on source operands S1 and S2
      - Qj,Qk: The reservation stations that will produce the corresponding source operand; a value of zero indicates that the source operand is already available in Vj or Vk, or is unnecessary.
      - Vj,Vk: The value of the source operands.
      - AL Used to hold information for the memory address calculation for a load or store.
      - Busy: Indicates that this reservation station and its accompanying functional unit are occupied.
    - The register file has:
      - Qi: The number of the reservation station that contains the operation whose result should be stored into this register. If the Qi is zero, it means that the data is ready.

### 10 Hardware based speculation:
  - Three key ideas:
    - Dynamic branch prediction to choose which instructions to execute.
    - Speculation to allow the execution of instructions before the control dependences are resolved
    - Dynamic scheduling to deal with the scheduling of different combinations of basic blocks.
  - Reorder buffer (ROB): It holds the result of an instruction between the time the operation associated with the instruction completes and the time the instruction commits.
    - Each entry in ROB holds four fields: the instruction type, the destination field, the value field and the ready field.
    - ROB becomes the source of operands. The ROB entry index is used to tag the result.
    - When a branch with incorrect prediction reaches the head of the ROB, it indicates that the speculation was wrong. The ROB is flushed and execution is restarted at the correct branch.
  - In-order commit can ensure the precise exception which the basic Tomasulo algorithm cannot achieve. However, imprecise floating-point exception is deemed acceptable.
