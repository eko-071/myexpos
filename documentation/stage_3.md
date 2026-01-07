# Stage 3: Bootstrap Loader

## XSM Machine Organization

- The processor consists of a set of registers and ports along with the hardware.
- Each register/port can store a string.
- Two contiguous memory words to store an XSM instruction.
- Two fundamental modes of execution:-
    - Privileged: Can execute any instruction and has full view of the memory and disk.
    - Unprivileged: Only has access to a restricted machine model called the **XSM virtual machine**. Instruction set is a subset of privileged.

### Registers/Ports

- 29 registers which can each hold a single word, & 4 ports, of which one (P0) is used for Standard Input and one (P1) for Standard Output (Remaining 2 are unused).
- General Purpose User Registers are R0-R19, of which R16-R19 are reserved for the SPL compiler.

### Memory

- Organized as a sequence of 128 pages.
- Pages are a sequence of 512 words.
- Word addressable.
- Page 0 for ROM code, and Page 1 for loading the BOOT block from disk.

### Disk

- Organized as a sequence of 512 blocks.
- Blocks are a sequence of 512 words.
- Block addressable.
- Block 0 for BOOT block. 

### Boot ROM and Boot block

- Program execution starts at first word of first page (Page 0) of memory.
- **Bootstrap Loader** is the pre-loaded ROM code in Page 0 that loads block 0 (Boot block) from disk to page 1 of memory, and then transfers control to first instruction in page 1 using the jump instruction.

## Start

1. Create an assembly program to print "Hello World" and save it in `myexpos/spl/spl_progs/helloworld.xsm`. The contents of the file are as such:
```bash
MOV R16, "Hello World"
PORT P1, R16
OUT
HALT 
```

2. Load the file as OS Startup code.
```bash
./xfs-interface
load --os ../spl/spl_progs/helloworld.xsm
exit
```

3. Running the machine
```bash
./xsm
```
Output:
```bash
Hello World
Machine is halting.
```

## Question

If the OS Startup Code is loaded to some other page other than Page 1, will XSM work fine?

**Answer:** No, it won't work since after ROM code is executed, Instruction Pointer will point to 512, the first instruction of Page 1. So if there's no OS Startup code there, the system crashes.

## Assignments

1. Write an assembly program to print numbers from 1 to 20 and run it as the OS Startup code.

Assembly code:
```bash
MOV R0, 1
MOV R1, 20

LOOP:
PORT P1, R0
OUT
ADD R0, 1
MOV R2, R0
GT R2, R1
JNZ R2, LOOP_END
JMP LOOP

LOOP_END:
HALT
```

Output:
```bash
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
Machine is halting.
```