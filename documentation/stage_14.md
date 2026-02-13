# Round Robin Scheduler

- We're implementing the scheduler here.

## Start

1. Load the odd number printing program as init code, and the even printing program as an executable.
```
load --init ../expl/expl_progs/oddnum.xsm
load --exec ../expl/expl_progs/evennum.xsm
```

The code blocks of evennum.xsm is `83` and `84`.

### Modifying Boot Module Code

1. Loading even to disk.

2. Setting up the Process Table entry and Page Table entries.

3. Setting IP to top of user stack.

4. Initialise every other process to TERMINATED.

- The final code:
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

// Module 5: Scheduler
loadi(50,63);
loadi(51,64);

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

alias i R6;
i=3;
while(i < 16) do
    [PROCESS_TABLE + 16*i + 4] = TERMINATED;
    i = i+1;
endwhile;

return;
```

### Modifying Timer Interrupt

- The final code:
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

call MOD_5;

// Restoring user context and setting SP back to user SP
restore;
SP = [PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1]*16) + 13];
[PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1]*16) + 9] = 0;
[PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1]*16) + 12] = 0;

ireturn;
```

### Module 5: Scheduler

- The final code:
```
alias currentPID R0;
currentPID = [SYSTEM_STATUS_TABLE+1];

SP=SP+1;
[SP]=BP;

alias process_table_entry R1;
process_table_entry = PROCESS_TABLE + currentPID * 16;

[process_table_entry + 12] = SP % 512;
[process_table_entry + 14] = PTBR;
[process_table_entry + 15] = PTLR;

alias nextPID R2;
nextPID = (currentPID + 1) % 16;
while (nextPID != currentPID) do
    if ([PROCESS_TABLE + 16*nextPID + 4] == READY || [PROCESS_TABLE + 16*nextPID + 4] == CREATED) then
        break;
    endif;
    nextPID = (nextPID + 1) % 16;
endwhile;

alias next_process_table R3;
next_process_table = PROCESS_TABLE + nextPID * 16;

SP =  [next_process_table + 11] * 512 + [next_process_table + 12] ;
PTBR = [next_process_table + 14];
PTLR = [next_process_table + 15];

[SYSTEM_STATUS_TABLE + 1] = nextPID;

if([next_process_table + 4] == CREATED) then
    [next_process_table + 4] = RUNNING;
	[next_process_table + 9] = 0;
	SP = [next_process_table + 13];
	ireturn;
endif;

[next_process_table + 4] = RUNNING;
BP = [SP];
SP = SP - 1;
return;
```

### Modifying INT 10

- The final code:
```
alias currentPID R0;
currentPID = [SYSTEM_STATUS_TABLE+1];

alias process_table_entry R1;
process_table_entry = PROCESS_TABLE + 16*currentPID;

[process_table_entry + 4] = TERMINATED;
alias i R2;
i=1;
while ((i<16) && ([PROCESS_TABLE + 16*i + 4] == TERMINATED)) do
    i = i+1;
endwhile;

if (i>=16) then
    halt;
endif;

call MOD_5;
```

- Now compile and load all of this.

## Question

1. When does the OS kernel invoke the scheduler from some routine other than the timer interrupt handler?

- If a process gets blocked inside a kernel module because it's waiting for something, the process will set its state to WAITING and call the scheduler. When the process gets its resources and is back in READY state and is selected by the scheduler, execution return to the instruction following the call to scheduler.

## Assignment

1. Write ExpL programs to print odd numbers, even numbers and prime numbers between 1 and 100. Modify the boot module code accordingly and run the machine with these 3 processes along with idle process. 

- The result:
```
// Library
loadi(63,13);
loadi(64,14);

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

// Module 5: Scheduler
loadi(50,63);
loadi(51,64);

// This is the INIT pr0cess

// Code: INIT Program
loadi(65,7);
loadi(66,8);

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
[PTBR+10] = -1;
[PTBR+11] = "0100";
[PTBR+12] = -1;
[PTBR+13] = "0000";
[PTBR+14] = -1;
[PTBR+15] = "0000";

//Stack
[PTBR+16] = 86;
[PTBR+17] = "0110";
[PTBR+18] = -1;
[PTBR+19] = "0110";

[PROCESS_TABLE + 32 + 1] = 2; // process PID
[PROCESS_TABLE + 32 + 4] = CREATED; // State
[PROCESS_TABLE + 32 + 11] = 87; //User Area Page Number
[PROCESS_TABLE + 32 + 12] = 0; // KPTR
[PROCESS_TABLE + 32 + 13] = 8*512; // UPTR
[PROCESS_TABLE + 32 + 14] = PAGE_TABLE_BASE + 40;
[PROCESS_TABLE + 32 + 15] = 10;

[86*512] = [83*512 + 1];

// Primes
loadi(84, 70);

PTBR = PAGE_TABLE_BASE + 60;
PTLR=10;

//Library
[PTBR+0] = 63;
[PTBR+1] = "0100";
[PTBR+2] = 64;
[PTBR+3] = "0100";

//Heap
[PTBR+4] = 92;
[PTBR+5] = "0110";
[PTBR+6] = 93;
[PTBR+7] = "0110";

//Code
[PTBR+8] = 84;
[PTBR+9] = "0100";
[PTBR+10] = -1;
[PTBR+11] = "0000";
[PTBR+12] = -1;
[PTBR+13] = "0000";
[PTBR+14] = -1;
[PTBR+15] = "0000";

//Stack
[PTBR+16] = 90;
[PTBR+17] = "0110";
[PTBR+18] = 91;
[PTBR+19] = "0110";

[PROCESS_TABLE + 48 + 1] = 3;
[PROCESS_TABLE + 48 + 4] = CREATED;
[PROCESS_TABLE + 48 + 11] = 94;
[PROCESS_TABLE + 48 + 12] = 0;
[PROCESS_TABLE + 48 + 13] = 8 * 512;
[PROCESS_TABLE + 48 + 14] = PAGE_TABLE_BASE + 60;
[PROCESS_TABLE + 48 + 15] = 10;
[90 * 512] = [84 * 512 + 1];

alias i R6;
i=4;
while(i < 16) do
    [PROCESS_TABLE + 16*i + 4] = TERMINATED;
    i = i+1;
endwhile;

return;
```