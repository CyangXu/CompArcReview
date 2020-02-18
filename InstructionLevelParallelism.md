<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [Instruction Level Parallelism](#instruction-level-parallelism)   
      - [1 Two ways of exploiting ILP:](#1-two-ways-of-exploiting-ilp)   
      - [2 CPI for a pipelined processor](#2-cpi-for-a-pipelined-processor)   
      - [3 Major techniques utilizing ILP](#3-major-techniques-utilizing-ilp)   
      - [4 Type of dependences](#4-type-of-dependences)   
      - [5 Data hazards](#5-data-hazards)   
      - [6 Basic compiler techniques for exposing ILP](#6-basic-compiler-techniques-for-exposing-ilp)   
      - [7 Reducing branch costs with advanced branch prediction](#7-reducing-branch-costs-with-advanced-branch-prediction)   

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
