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
  - In practice, processors that speculate try to recover as early as possible. This recovery can be done by clearing the ROB for all entries that appear after the mispredicted branch.

### 11 Exploiting ILP using multiple issue and static scheduling
 - Multiple-issue processors come in three major flavors:
   - Statically scheduled superscalar processors
   - VLIW(very long instruction word) processors
   - Dynamically scheduled superscalar processors
 - Because of the diminishing advantages of a statically scheduled superscalar as the issue width grows, statically scheduled superscalars are used primarily for narrow issue widths, normally just two instructions. Beyond that width, most designers chose to implement either a VLIW or a dynamically scheduled superscalar.
 - A VLIW packages the multiple operations into one very long instruction, or requires that the instructions in the issue packet satisfy the same constraints.
 - Cos of VLIW:
   - Generating enough operations in a straight-line code fragment requires ambitiously unrolling loops, thereby increasing code size
   - Whenever instructions are not full, the unused functional units translate to wasted bits in the instruction encoding.
  - To combat the code size increase:
    - using clever encoding: for example, there may be only one large immediate field for use by any function unit.
    - Compressing the instructions in main memory and expanding them when they are read into cache or memory.
  - Binary code compatibility is a major logistical problem

### 11 Exploiting ILP using dynamic scheduling, multiple issue, and speculation
- Reason why processor becomes complex in this way:
- Two approaches to issue multiple instructions per clock in a dynamically scheduled processor:
  - Run this step in half a clock cycle. But it cannot extended to four instructions per clock easily.
  - Build the logic necessary to handle two or more instructions at once, including any possible dependences between the instructions.
- Example of ILP execution without branch speculation
  - Assume that there are sperate integer functional units for effective address calculation, ALU operation, branch condition evaluation. Two instructions can be issued in one cycle.
  ```
  Loop: LD     R2,0(R1)   ;R2=array element
        DADDIU R2,R2,#1   ;increment R2
        SD     R2,0(R1)   ;store result
        DADDIU R1,R1,#8   ;increment pointer
        BNE    R2,R3,Loop ;branch if not last element
  ```
  - Result of the first three iterations

  Iteration Number | Instructions | Issues at clock cycle number | Executes at clock cycle number | Memory access at clock cycle number | Write CDB at clock cycle number | Comment
   ---| --- | --- | --- | --- | --- | --- |
   1 | LD R2,0(R1) | 1 | 2 | 3 | 4 | First issue
   1 |DADDIU R2,R2,#1 | 1 | 5 ||6|Wait for LW
   1 |SD R2,0(R1)|2|3|7||Wait for DADDIU
   1 |DADDIU R1,R1,#8|2|3||4|Execute directly
   1 |BNE R2,R3,Loop|3|7|||Wait for DADDIU
   2 | LD R2,0(R1) | 4 | 8 | 9 | 10 | Wait for BNE
   2 |DADDIU R2,R2,#1 |4| 11 ||12|Wait for LW
   2 |SD R2,0(R1)|5|9|13||Wait for DADDIU
   2 |DADDIU R1,R1,#8|5|8||9|Wait for SD
   2 |BNE R2,R3,Loop|6|13|||Wait for DADDIU
   3 | LD R2,0(R1) | 7 | 14 | 15 | 16 | Wait for BNE
   3 |DADDIU R2,R2,#1 | 7 | 17 ||18|Wait for LW
   3 |SD R2,0(R1)|8|14|19|||Wait for DADDIU
   3 |DADDIU R1,R1,#8|8|14||15|Wait for BNE
   3 |BNE R2,R3,Loop|9|19|||Wait for DADDIU
- Example of ILP without branch speculation
- Result of the first three iterations

Iteration Number | Instructions | Issues at clock cycle number | Executes at clock cycle number | Read access at clock cycle number | Write CDB at clock cycle number | Commits at clock numner | Comment
 ---| --- | --- | --- | --- | --- | --- |--- |
 1 | LD R2,0(R1) | 1 | 2 | 3 | 4 | 5 | First issue
 1 |DADDIU R2,R2,#1 | 1 | 5 ||6|7|Wait for LW
 1 |SD R2,0(R1)|2|3|||7|Wait for DADDIU
 1 |DADDIU R1,R1,#8|2|3||4|8|Commit in order
 1 |BNE R2,R3,Loop|3|7|||8|Wait for DADDIU
 2 | LD R2,0(R1) | 4 | 5 |6| 7 | 9 | First issue
 2 |DADDIU R2,R2,#1 | 4 | 8 ||9|10|Wait for LW
 2 |SD R2,0(R1)|5|6|||10|Wait for DADDIU
 2 |DADDIU R1,R1,#8|5|6||7|11|Execute directly
 2 |BNE R2,R3,Loop|6|10|||11|Wait for DADDIU
 3 | LD R2,0(R1) |7|8|9|10 | 12 | Wait for commit
 3 |DADDIU R2,R2,#1 |7|11||12|13|Wait for LW
 3 |SD R2,0(R1)|8|9|||13|Wait for DADDIU
 3 |DADDIU R1,R1,#8|8|9||10|14|Wait for commit
 3 |BNE R2,R3,Loop|9|13|||14|Wait for DADDIU

### 14 Advanced techniques for instruction delivery and speculation
- Increasing instruction fetch bandwidth
  - Branch-target buffer
    - A branch-target buffer predicts the next instruction address and will send it out before decoding the instruction.
    - Note that unlike a branch-prediction buffer, the predictive entry must be matched to this instruction because the predicted PC will be sent out before it is known whether this instruction is even a branch.
  - Return address predictors
    - Incentive: The challenge of predicting indirect jumps, that is, jumps whose destination address varies at runtime. Although procedure returns can be predicted with a branch-target-buffer, the accuracy of such a prediction technique can be low if the procedure is called from multiple sites and the calls from one site are not clustered in time.
  - Integrated instruction fetch units
    - Integrated branch prediction
    - Instruction prefetch
    -  Instruction memory access and buffering

### 15 Studies of the limitations of ILP
  - History: ILP is utilized since 1960s. Engineers were enforced to change their persistence in ILP around 2005 because of the power inefficiency and too high use of silicon brought by aggressive ILP strategies.
  -  

### 16 Multithreading: exploiting thread-level parallelism to improve uniprocessor throughput
  - Incentive: ILP can be quite limited to utilize. For example, with reasonable instruction issue rates, cache misses that go to memory or off-chip caches are unlikely to be hidden by available ILP. In addition, when the processor is stalled, the utilization of the functional units drops dramatically.
  - Duplicating the per-thread state of a processor core means creating a separate register file, a separate PC, and a separate page table for each thread.
  - Three hardware ways to multithreading:
    - Fine grained multi-threading
      - pros: Hide the throughput losses that arise from both short and long stalls.
      - cons: Slow down the execution of an individual thread
      - applications: Sun Niagara processor, Nvidia GPUs
    - Coarse grained multithreading
      - pros: less likely to slow down the individual thread speed.
      - cons: Limited to overcome throughput losses.
    - Simultaneous multithreading (SMT)

### 12 Summary
Common name | issue structure | hazard detection | scheduling | distinguishing characteristic | example
 --- | --- | --- | --- | --- | ---
 Superscalar (static) | dynamic | hardware | static | in-order execution | mostly in the embedded space: MIPS and ARM, including the ARM Cortex-A8
 Superscalar (dynamic) | dynamic | hardware | dynamic | some out-of-order execution, but no speculation | none at the present
 Superscalar (speculative) | dynamic | hardware | Dynamic with speculation | out-of-order execution with speculation | Intel Core i3, i5, i7; AMD Phenom; IBM Power 7
 VLIW/LIW | static | Primarily software | Static | All hazards determined and indicated by compiler | Most examples are in signal processing, such as the TIC6x
 EPIC | Primarily static | Primarily software | Mostly static | All hazards determined and indicated explicitly by the compiler | Itanium
