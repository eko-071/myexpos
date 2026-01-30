# Introduction to Multiprogramming

- Purpose: to load two processes into memory and put them on concurrent execution.
- We modify the timer interrupt to switch between the two processes to do this.

## Start

- We'll be using the same init process as before, and a second process called an **idle process** with an infinite loop will also be setup.

### Idle Program

- Stored in disk blocks 11 and 12.
- Loaded to memory pages 69 and 70.
- PID is fixed to be 0.
- It's a simple infinite loop.
```
int main()
{
    decl
        int a;
    enddecl
    begin
        while(1 == 1) do
            a=1;
        endwhile;
        return 0;
    end
}
```
- Load into disk blocks 11 and 12 using XFS interface.
```
# load --idle ../expl/expl_progs/idle.xsm
```

### OS Startup Code

- Since design stipulates the PID of idle to be 0, INIT is assigned 0.

1. Load idle code
```
loadi(69,11);
loadi(70,12);
```

- Final OS Startup Code
```
// Library
loadi(63,13);
loadi(64,14);

// Idle Code
loadi(69,11);
loadi(70,12);

// Code: INIT Program
loadi(65,7);
loadi(66,8);

// INT7 module
loadi(16,29);
loadi(17,30);

// INT10 module
loadi(22,35);
loadi(23,36);

// Exception handler
loadi(2,15);
loadi(3,16);

// Timer Interrupt
loadi(4, 17);
loadi(5, 18);

// Starting with IDLE since PID is 0

PTBR = PAGE_TABLE_BASE;
PTLR = 10;

// Library
[PTBR+0] = -1;
[PTBR+1] = "0000";
[PTBR+2] = -1;
[PTBR+3] = "0000";

//Heap
[PTBR+4] = -1;
[PTBR+5] = "0000";
[PTBR+6] = -1;
[PTBR+7] = "0000";

//Code
[PTBR+8] = 69;
[PTBR+9] = "0100";
[PTBR+10] = 70;
[PTBR+11] = "0100";
[PTBR+12] = -1;
[PTBR+13] = "0000";
[PTBR+14] = -1;
[PTBR+15] = "0000";

//Stack
[PTBR+16] = 81;
[PTBR+17] = "0110";
[PTBR+18] = -1;
[PTBR+19] = "0000";

[PROCESS_TABLE + 1] = 0;
[PROCESS_TABLE + 4] = CREATED;
[PROCESS_TABLE + 11] = 82;
[PROCESS_TABLE + 12] = 0;
[PROCESS_TABLE + 13] = 8*512;
[PROCESS_TABLE + 14] = PTBR;
[PROCESS_TABLE + 15] = PTLR;

[81 * 512] = [69*512 + 1];

// This is the INIT pr0cess

PTBR = PAGE_TABLE_BASE + 20;

//Library
[PTBR+0] = 63;
[PTBR+1] = "0100";
[PTBR+2] = 64;
[PTBR+3] = "0100";

//Heap
[PTBR+4] = 78;
[PTBR+5] = "0110";
[PTBR+6] = 79;
[PTBR+7] = "0110";

//Code
[PTBR+8] = 65;
[PTBR+9] = "0100";
[PTBR+10] = 66;
[PTBR+11] = "0100";
[PTBR+12] = -1;
[PTBR+13] = "0000";
[PTBR+14] = -1;
[PTBR+15] = "0000";

//Stack
[PTBR+16] = 76;
[PTBR+17] = "0110";
[PTBR+18] = 77;
[PTBR+19] = "0110";

[PROCESS_TABLE + 16 + 1] = 1; // process PID
[PROCESS_TABLE + 16 + 4] = RUNNING;
[PROCESS_TABLE + 16 + 11] = 80; //User Area Page Number
[PROCESS_TABLE + 16 + 12] = 0;
[PROCESS_TABLE + 16 + 13] = 8*512;
[PROCESS_TABLE + 16 + 14] = PTBR;
[PROCESS_TABLE + 16 + 15] = PTLR;

[76*512] = [65*512 + 1];
SP = 8*512;

//Current PID
[SYSTEM_STATUS_TABLE + 1] = 1;

ireturn;
```

### Timer Interrupt

- The final timer interrupt code
```
//Saving current value of User SP into Process Table
[PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1] * 16) + 13] = SP;

//Setting SP to beginning of kernel stack
SP = [PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1] * 16) + 11] * 512 - 1;

//Saving user context to kernel stack
backup;

// 0btaining PID of currently executing process 
alias currentPID R0;
currentPID = [SYSTEM_STATUS_TABLE+1];

alias process_table_entry R1;
process_table_entry = PROCESS_TABLE + currentPID * 16;

[process_table_entry + 4] = READY;
[process_table_entry + 12] = SP % 512;
[process_table_entry + 14] = PTBR;
[process_table_entry + 15] = PTLR;

// Toggling between the processes
alias newPID R2;
if(currentPID == 0) then
	newPID = 1;
else
	newPID = 0;
endif;

//Set back Kernel SP, PTBR , PTLR
alias new_process_table R3;
new_process_table = PROCESS_TABLE + newPID * 16;
SP =  [new_process_table + 11] * 512 + [new_process_table + 12] ;
PTBR = [new_process_table + 14];
PTLR = [new_process_table + 15];

[SYSTEM_STATUS_TABLE + 1] = newPID;

// Check f0r CREATED
if([new_process_table + 4] == CREATED) then
	[new_process_table + 4] = RUNNING;
	SP = [new_process_table + 13];
	ireturn;
endif;
[new_process_table + 4] = RUNNING;

print "TIMER";
print [SYSTEM_STATUS_TABLE + 1];

// Restoring user context and setting SP back to user SP
restore;
SP = [PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1]*16) + 13];

ireturn;

```

- Now compile and load everything.

## Questions

1. What is the significance of the idle process?

The purpose of this is to run as a background process in an infinite loop.  This is demanded by the OS so that the scheduler will always have at least one "READY" process to schedule. 

## Assignments

 1. Load a program to print numbers from 1-100 as the INIT program, and modify IDLE to print numbers from 101-200. (You will have to link the library to address space of IDLE for the Write function call to work.)

 - In Startup Code, you modify it so that library is linked to IDLE.
```
// Library
[PTBR+0] = 63;
[PTBR+1] = "0100";
[PTBR+2] = 64;
[PTBR+3] = "0100";
```

- It's then pretty simple to modify the IDLE and INIT code.
```
int main()
{
    decl
        int a, temp;
    enddecl
    begin
        a=101;
        while(a <= 200) do
            temp = exposcall("Write", -2, a);
            a = a+1;
        endwhile;
        return 0;
    end
}
```

2.  Set two breakpoints in the timer interrupt routine, the first one immediately upon entering the timer routine and the second one just before return from the timer routine. Dump the process table entry and page table entries of current process (see XSM debugger for various printing options).