# Instruction set principles
<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [Instruction set principles](#instruction-set-principles)
  - [1. Introduction](#1-introduction)
  - [2. Classify instruction set architectures](#2-classify-instruction-set-architectures)
  - [3 Memory addressing](#3-memory-addressing)
  - [4 Operations in the instruction set.](#4-operations-in-the-instruction-set)
  - [5 Instructions for control flow](#5-instructions-for-control-flow)
  - [6 Encoding an instruction set](#6-encoding-an-instruction-set)
  - [7 Crosscutting Issues: The Role of Compilers](#7-crosscutting-issues-the-role-of-compilers)
  - [8 The MIPS architecture](#8-the-mips-architecture)

<!-- /MDTOC -->
## 1. Introduction

- General-purpose RISC architectures: MIPS, PowerPC, Precision Architecture, SPARC
- Embedded RISC processors: ARM, Hitachi SH, MIPS 16, Thumb

## 2. Classify instruction set architectures

- type of internal storage:
  - stack
  - accumulator
  - set of registers: register-memory or register-register
- Although most early computers used stack or accumulator-style architectures, virtually every architecture designed after 1980 uses a load-store register architecture because:
  - Registers are faster than memory.
  - Registers are more efficient for a compiler to use.
- Modern compiler technology and its ability to effectively use larger numbers of registers has led to an increase in register counts in more recent architectures.
- Two major instruction set characteristics divide the general purpose register (GPR) architectures:
  - Whether an ALU instruction has two or three operands
  - How many of the operands may be memory addresses in ALU instructions
  - Comparison:

    Type | Advantages | Disadvantages
    --- | --- | --- |
    Register-register (0, 3) | Simple, fixed-length instruction encoding. Simple code generation model. Instructions take similar numbers of clocks to execute | Higher instruction count than architectures with memory references in instructions. More instructions and lower instruction density lead to larger programs.
    Register-memory (1, 2) | Data can be accessed without a separate load instruction first. Instruction format tends to be easy to encode and yields good density. | Operands are not equivalent since a source operand in a binary operation is destroyed. Encoding a register number and a memory address in each instruction may restrict the number of registers. Clocks per instruction vary by operand location.

## 3 Memory addressing

- Big endian and little endian
- Whether accesses to objects larger than a byte must be aligned.
- Addressing modes:

  Addressing mode | Example instruction | Meaning | When used
  --- | --- | --- | --- |
  Register | Add R4, R3 | Regs[R4] <- Regs[R4] + Regs[R3] | When a value is in a register
  Immediate | Add R4, \#3 | Regs[R4] <- Regs[R4] + 3 | For constants
  Displacement | Add R4, 100(R1) | Regs[R4] <- Regs[R4] + Mem[100 + Regs[R1]] | Accessing local variables (+ simulates register indirect, direct addressing modes)
  Register indirect | Add R4, (R1) | Regs[R4] <- Regs[R4] + Mems[Regs[R1]] | Accessing using a pointer or a computed address.
  Indexed | Add R3, (R1 + R2) | Regs[R3] <- Regs[R3] + Mem[Regs[R1] + Regs[R2]] | Sometimes useful in array addressing: R1 = base of array; R2 = index amount.
  Direct or absolute | Add R1, (1001) | Regs[R1] <- Regs[R1] + Mem[1001] | Sometimes useful for accessing static data; address constant may need to large.
  Memory indirect | Add R1, @(R3) | Regs[R1] <- Regs[R1] + Mem[Mem[Regs[R3]]] | If R3 is the address of a pointer p, then mode yeilds \*p.
  Autoincrement | Add R1, (R2)+ | Regs[R1] <- Regs[R1] + Mem[Regs[R2]]    Regs[R2] <- Regs[R2] + d | Useful for stepping through arrays within a loop. R2 points to start of array; each reference increments R2 by size of an element d.
  Autodecrement | Add R1, -(R2) | Regs[R2] <- Regs[R2] - d Regs[R1] <- Regs[R1] + Mem[Regs[R2]] | Same as Autoincrement.
  Scaled | Add R1, 100[R2](R3) | Regs[R1] <- Regs[R1] + Mem[100 + Regs[R2] + Regs[R3] * d] | used to index arrays. May be applied to any indexed addressing mode in some computers.

- Type and size of operands
  - Basic types: word, single-precision floating point, and double-precision.
  - character string
  - decimal, used specially for financial transaction.

## 4 Operations in the instruction set

  Operator type | Examples |
  --- | --- |
  Arithmetic and logical | Integer arithmetic and logical operations: add, subtract, and, or, multiply, divide
  Data transfer | Loads-stores
  Control | Branch, jump, procedure call and return, traps
  System | Operating system call, virtual memory management instructions
  Floating point | floating-point operation: add, multiply, divide, compare
  Decimal | Decimal add, decimal multiply, decimal-to-character conversions
  String | String move, string compare, string search
  Graphics | Pixel and vertex operations, compression/decompression operations

## 5 Instructions for control flow

- Basic types of control flow change:
    conditional branches, jumps, procedure calls, procedure returns
- addressing modes:
  - PC-relative
  - register-indirect jump, usually used in four cases:
    - Case or switch statement
    - Virtual functions or methods
    - High-order functions or function pointers
    - Dynamically shared libraries
- Conditional branch options:
    Name | Examples | How condition is tested | Advantages | Disadvantages |
    --- | --- | --- | --- | --- |
    Condition code (CC) | 80x86, ARM, PowerPC, SPARC, SuperH | Test special bits set by ALU operations, possibly under program control | Sometimes condition is set for free | CC is extra state. Condition codes constrain the ordering of instructions since they pass information from one instruction to a branch.
    Condition register | Alpha, MIPS | Tests arbitrary register with the result of a comparison | Simple | Uses up a register
    Compare and branch | PA-RISC, VAX | Compare is part of the branch. Often compare is limited to subset. | One instruction rather than two for a branch. | May be too much work per instruction for pipelined execution.
- Procedure invocation options
  - Two basic conventions in use to save registers:
    - Caller saving and callee saving
    - The convention is specified in an application binary interface (ABI) that sets down the basic rules as to which registers should be caller saved and which should be callee saved.

## 6 Encoding an instruction set

- The operation is typically specified in one field, called the opcode.
- Address specifier: the address specifier tells what addressing mode is used to access the opcode.
- Several competing forces when encoding the instruction set:
  - the desire to have as many registers and addressing modes as possible
  - the impact of the size of the register and addressing mode fields on the average instruction size and hence on the average program size.
  - A desire to have instructions encoded into lengths that will be easy to handle in a pipelined implementation.
- __*variable*__ vs __*fixed*__
- Reduced code size in RISCs
  - Smaller code are important in embedded applications. So several manufacturers offered a new hybrid version of their RISC instruction sets, such as ARM Thumb and MIPS MIPS16.
  - IBM simply compresses its standard instruction set and then adds hardware to decompress instructions as they are fetched from memory. To handle branches, which are no longer to an aligned word boundary, the PowerPC creates a hash tablet in memory that maps between compressed and uncompressed addresses.
  - Hitachi simply invented a RISC instruction set with a fixed 16-bit format, called SuperH.

## 7 Crosscutting Issues: The Role of Compilers

- Architectural choices affect the quality of the code that can be generated for a computer and the complexity of building a good compiler for it, for better or for worse.
- Optimization performed by modern compilers:
  - __High-level optimizations__ are often done on the source with output fed to later optimization passes.
  - __Local optimizations__ optimize code only within a straight-line code fragment.
  - __Global optimizations__ extend the local optimizations across branches and introduce a set of transformations aimed at optimizing loops.
  - __Register allocation__ associates registers with operands.
  - __Processor-dependent optimizations__ attempt to take advantage of specific architectural knowledge.
- Register allocation
  - __Graph coloring__.It is NP-complete problem but there are heuristic algorithms. However, these heuristic algorithms do not work well when the number of registers is small.

## 8 The MIPS architecture

- Registers for MIPS:  
  - 32 64-bit general-purpose registers
  - 32 floating-point registers
- Data types:
  - 8-bit bytes, 16-bit half words, 32-bit words, and 64-bit double words for integer data and 32-bit single precision and 64-bit double precision for floating point.
- Addressing modes for MIPS data transfers:
  - Immediate and displacement
- MIPS instruction format
  - I-type instruction: 6-bit opcode, 5-bit rs, 5-bit rt and 16-bit immediate
  - R-type instruction: 6-bit opcode, 5-bit rs, 5-bit rt, 5-bit rd, 5-bit shamt and 6-bit funct
  - J-type instruction: 6-bit opcode and 26-bit offset added to PC.
- MIPS control flow instructions
    Example instruction | instruction name | Meaning
    --- | --- | --- |
    J name | Jump | PC[36, 63]<-name
    JAL name | Jump and link | Regs[R31]<-PC+8; PC[36...63]<-name;((PC + 4)-2^27) <= name < ((PC+4) + 2^27)
    JALR R2 | Jump and link register | Regs[R31]<-PC+8; PC<-Regs[R2]
    JR R3 | Jump register | PC<-Regs[R3]
    BEQZ R4, name | Branch equal zero | if (Regs[R3] == 0) PC<-name;
    ((PC+4)-2^17)<=name<((PC+4)+2^17)
    BNE R3, R4,name | Branch not equal zero | if (Regs[R3] != Regs[R4]) PC<-name; ((PC+4)-2^17)<=name<((PC+4)+2^17)
    MOVZ R1,R2,R3 | Conditional move if zero | if(Regs[R3] == 0) Regs[R1]<-Regs[R2]
- MIPS floating-point operation
  - MOVS MOVD
  - MFC1, MTC1, DMFC1, DMTC1
  - ADDD, ADDS, SUBD, SUBS, MULD, MULS, DIVD, DIVS
