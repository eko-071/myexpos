# Boot Module

- Modules are used to perform logical tasks that are performed frequently.
- Run and invoked only in kernel mode.
- User programs can never invoke modules directly.
- Have to use interrupt routines, other modules, or startup code.
- Kernel stack of currently scheduled process used as caller stack for invocation.
- XSM supports 8 modules, invoked using the `CALL MOD_n` or `CALL <MODULE_NAME>` instruction.
- The `CALL` instruction pushes the IP address of the next instruction on the top of kernel stack and starts execution of the module.
- It returns to caller using `RET` instruction which restores IP present on top of kernel stack.
- Different from IRET; IRET changes mode from kernel to user as it assumes that SP contains a logical address, RET just returns to caller in kernel mode using IP value pointed by SP.
- Code for OS startup may exceed one page due to initialization of several OS data structures, so we design **boot module** for initialization.
- What code does now:-
    - Creates idle process
    - Initializes SP register to kernel stack of idle process
    - Loads module 7 in memory and invokes it
    - After return, initiates user mode execution of idle process.

## Start

- Startup Code
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

// Module 7
loadi(67, 54);
loadi(68, 55);

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

// Calling boot module
SP = 82*512 - 1;
call BOOT_MODULE;

PTBR = PAGE_TABLE_BASE;
PTLR = 10;

// Library
[PTBR+0] = 63;
[PTBR+1] = "0100";
[PTBR+2] = 64;
[PTBR+3] = "0100";

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

[PROCESS_TABLE + 1] = 0; // PID
[PROCESS_TABLE + 4] = RUNNING; // State
[PROCESS_TABLE + 11] = 82; // User Area Page Number
[PROCESS_TABLE + 12] = 0; // Kernel Stack P0inter
[PROCESS_TABLE + 13] = 8*512; // User Stack P0inter
[PROCESS_TABLE + 14] = PTBR;
[PROCESS_TABLE + 15] = PTLR;

[81 * 512] = [69*512 + 1];

//Current PID
[SYSTEM_STATUS_TABLE + 1] = 0;

SP = 8*512;

ireturn;
```

- Boot module
```
// Library
loadi(63,13);
loadi(64,14);

// Code: INIT Program
loadi(65,7);
loadi(66,8);

// Module 7
loadi(67, 54);
loadi(68, 55);

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
[PROCESS_TABLE + 16 + 4] = CREATED; // State
[PROCESS_TABLE + 16 + 11] = 80; //User Area Page Number
[PROCESS_TABLE + 16 + 12] = 0; // KPTR
[PROCESS_TABLE + 16 + 13] = 8*512; // UPTR
[PROCESS_TABLE + 16 + 14] = PTBR;
[PROCESS_TABLE + 16 + 15] = PTLR;

[76*512] = [65*512 + 1];

return;
```

## Assignments

1. Write ExpL programs to print even and odd numbers below 100. Modify the boot module code and the timer interrupt handler to schedule the two processes along with the idle process concurrently using the Round Robin scheduling algorithm.

- Boot Module
```
// Library
loadi(63,13);
loadi(64,14);

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

// This is the INIT pr0cess

PTBR = PAGE_TABLE_BASE + 20;
PTLR = 10;

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
[PROCESS_TABLE + 16 + 4] = CREATED; // State
[PROCESS_TABLE + 16 + 11] = 80; //User Area Page Number
[PROCESS_TABLE + 16 + 12] = 0; // KPTR
[PROCESS_TABLE + 16 + 13] = 8*512; // UPTR
[PROCESS_TABLE + 16 + 14] = PTBR;
[PROCESS_TABLE + 16 + 15] = PTLR;

[76*512] = [65*512 + 1];

// 76 - 80 is used by init, and 81 - 82 by idle

// Code: Even Program
loadi(83, 69);
loadi(84, 70);

PTBR = PAGE_TABLE_BASE + 40;
PTLR = 10;

//Library
[PTBR+0] = 63;
[PTBR+1] = "0100";
[PTBR+2] = 64;
[PTBR+3] = "0100";

//Heap
[PTBR+4] = 87;
[PTBR+5] = "0110";
[PTBR+6] = 88;
[PTBR+7] = "0110";

//Code
[PTBR+8] = 83;
[PTBR+9] = "0100";
[PTBR+10] = 84;
[PTBR+11] = "0100";
[PTBR+12] = -1;
[PTBR+13] = "0000";
[PTBR+14] = -1;
[PTBR+15] = "0000";

//Stack
[PTBR+16] = 85;
[PTBR+17] = "0110";
[PTBR+18] = 86;
[PTBR+19] = "0110";

[PROCESS_TABLE + 32 + 1] = 2; // process PID
[PROCESS_TABLE + 32 + 4] = CREATED; // State
[PROCESS_TABLE + 32 + 11] = 89; //User Area Page Number
[PROCESS_TABLE + 32 + 12] = 0; // KPTR
[PROCESS_TABLE + 32 + 13] = 8*512; // UPTR
[PROCESS_TABLE + 32 + 14] = PAGE_TABLE_BASE + 40;
[PROCESS_TABLE + 32 + 15] = 10;

[85*512] = [83*512 + 1];

return;
```

- Timer Interrupt
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

// Round Robin
alias newPID R2;
newPID = currentPID + 1;
if (newPID == 3) then
	newPID = 0;
endif;

//Set back Kernel SP, PTBR, PTLR
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

// Restoring user context and setting SP back to user SP
restore;
SP = [PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1]*16) + 13];

ireturn;
```

2.  In the program of the previous assignment, add a breakpoint immediately upon entering the timer interrupt handler and print out in debug mode the contents of the page table entry and the process table entry of the current process (that is, the process from which timer was entered). You need to use p and pt options of xsm debugger. Add another breakpoint just before return from the timer interrupt handler to print out the same contents. 

- Just add the breakpoints.