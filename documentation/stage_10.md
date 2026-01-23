# Console Output

- The aim is to modify the program such that the result is printed directly to the terminal.
- This is done by issuing a write system call from user program, which is serviced by interrupt routine 7.

## Start

### Changes to user program

- A system call is an OS routine that can be invoked from a user program.
- Write call of eXpOS (system call number 5) is coded inside the INT 7 handler, create and delete in INT 4, and so on.

- We're doing a modified one here that prints the contents of register R1 to the terminal.

1. Save the registers in use to user stack. Since user program calls system call routine, the OS expects it to save its own context.

2. Push system call number and arguments to stack.
    - System call number is 5.
    - Argument 1 is file descriptor, which is -2 for terminal.
    - Argument 2 is the word to be written to the terminal.
    - By convention, all system calls have 3 arguments. Since we don't have one here, push some random register, maybe R0, to stack. Here, the last argument will be ignored ny system call handler.

3. Push any register, maybe R0, to allocate space for return value.

4. Invoke interrupt by "INT 7" instruction.

> The following code will be executed after return from system call.

5. Pop out the return value, the system call number and arguments which were pushed on the stack prior to the system call.

6. Restore register context from stack.

The final program is as follows:-
```
0
2056
0
0
0
0
0
0
MOV R0, 1
MOV R2, 5
GE R2, R0
JZ R2, 2110
MOV R1, R0
MUL R1, R0

// saving register context
PUSH R0
PUSH R1
PUSH R2

// pushing system call number and arguments
MOV R0, 5
MOV R2, -2
PUSH R0
PUSH R2
PUSH R1
PUSH R0

//  pushing space for return value
PUSH R0
INT 7

// popping out return value and ignore
POP R1

// pop out arguments and system call number and ignore
POP R1
POP R1
POP R1
POP R1

//  restoring the register context
POP R2
POP R1
POP R0

ADD R0, 1
JMP 2058

INT 10
```

> Note: Remove comments and spaces when you're actually loading it.

### INT 7 Module

1. Set the MODE FLAG field in process table to system call number, which is 5.
```
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 9] = 5;
```

2. Store the value of user SP in a register.
```
alias userSP R0;
userSP = SP;
```

3. Switch from user stack to kernel stack. First, save the value of SP is user SP field of Process Table, and then set value of SP to beginning of kernel stack;
```
[PROCESS_TABLE + 13 + [SYSTEM_STATUS_TABLE + 1]*16] = SP;
SP = [PROCESS_TABLE + 11 + [SYSTEM_STATUS_TABLE + 1]*16] * 512 - 1;
```

4. We have to access argument 1, which is file descriptor, to check if it's valid. Since interrupts are executed in kernel mode, we'll have to calculate the physical address of the memory location. Acc. to convention, `userSP - 4` is the location of argument 1.
```
alias physicalPageNum R1;
alias offset R2;
alias fileDescPhysicalAddr R3;

physicalPageNum = [PTBR + 2 * ((userSP - 4)/ 512)];
offset = (userSP - 4) % 512;
fileDescPhysicalAddr = (physicalPageNum * 512) + offset;
alias fileDescriptor R4;
fileDescriptor=[fileDescPhysicalAddr];
```

5. Check if it's valid or not. Here, it should be -2 since we're writing to the terminal. If it's not -2, we store -1 as a return value at userSP-1.
```
if (fileDescriptor != -2)
then
	alias physicalAddrRetVal R5;
	physicalAddrRetVal = ([PTBR + 2 * ((userSP - 1) / 512)] * 512) + ((userSP - 1) % 512);
	[physicalAddrRetVal] = -1;
else
	 //code when argument 1 is valid
endif;
```

6. If it's valid, we calculate the physical address of argument, extract the value from it, and print it to terminal. We then set the return value to 0, which is stored memory location userSP-1.
```
if (fileDescriptor != -2)
then
	alias physicalAddrRetVal R5;
	physicalAddrRetVal = ([PTBR + 2 * ((userSP - 1) / 512)] * 512) + ((userSP - 1) % 512);
	[physicalAddrRetVal] = -1;
else
	alias word R5;
    word = [[PTBR + 2 * ((userSP - 3) / 512)] * 512 + ((userSP - 3) % 512)];
    print word;
    alias physicalAddrRetVal R6;
    physicalAddrRetVal = ([PTBR + 2 * (userSP - 1)/ 512] * 512) + ((userSP - 1) % 512);
    [physicalAddrRetVal] = 0;
endif;
```

7. We then set back the value of SP to top of user stack.
```
SP = userSP;
```

8. Reset MODE FLAG to 0, which indicates that the process is in user mode now.
```
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 9] = 0;
```

9. Return to user program.
```
ireturn;
```

### Changes to OS Startup Code

1. Add code to load INT7 from disk to memory.
```
loadi(16,29);
loadi(17,30);
```

- Now we just compile everything and load to XSM disk.

## Questions

1. Why should we calculate the physical address of userSP-3 and userSP-1 separately instead of calculating one and adding/subtracting the difference from the calculated value?

Just because two logical addresses are close together does not mean that their corresponding physical addresses are similarly close by.
The stack of a process is spread over 2 pages, and these 2 pages need not be contiguous. 

## Assignment

1.  Write a program to print the first 20 numbers and run the system with timer enabled.

```
0
2056
0
0
0
0
0
0
MOV R0, 1
MOV R2, 20
GE R2, R0
JZ R2, 2108
MOV R1, R0
PUSH R0
PUSH R1
PUSH R2
MOV R0, 5
MOV R2, -2
PUSH R0
PUSH R2
PUSH R1
PUSH R0
PUSH R0
INT 7
POP R1
POP R1
POP R1
POP R1
POP R1
POP R2
POP R1
POP R0
ADD R0, 1
JMP 2058
INT 10
```