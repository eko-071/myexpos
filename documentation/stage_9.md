# Handling Kernel Stack

## Why two stacks?

- When the OS enters an interrupt handler that runs in kernel mode, eXpOS requires that the interrupt handler must switch to a different stack.
- This is to prevent user level hacks into the kernel through the stack.
- To isolate the kernel from user stack, the OS kernel must maintain 2 stacks for a program: a **user stack** and a **kernel stack**.

## How do we implement it?

- In eXpOS, one page called the **user area page** is allocated for each process, of which a part will be used for the kernel stack.
- Whenever there is a transfer of program control from user mode to kernel mode during interrupts or exceptions, the interrupt handler will change the stack to kernel stack.
- This means that the SP register will point to the top of the kernel stack of the program.
- We also need some way to store the SP values of the 2 stacks, since the SP register can only store one value.
- eXpOS requires us to maintain a **Process Table**, where data such as value of kernel stack pointer and user stack pointer and other stuff are stored.
- For now, we're only interested in storing the **user stack pointer** and the **memory page allocated as user area** for the program.
- The process table starts at **page number 56** (address 28762).
- It has space for **16 entries**, each having **16 words**.
- In the first entry, we'll be updating only the entries for user stack pointer **(word 13)** and user area page number **(word 11)** for now.

## System Status Table

- Stored starting from **memory address 29560**.
- Keeps information about:-
    - Number of free pages in memory
    - Number of process blocked because memory is unavailable
    - Number of processes in swapped state
    - The PID of the process to be scheduled next
- Size of the table is 8 words, out of which 2 are unused.
- The format is as follows:

| CURRENT_USER_ID | CURRENT_PID | MEM_FREE_COUNT | WAIT_MEM_COUNT | SWAPPED_COUNT | PAGING_STATUS |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
|User ID of currently logged in user|PID of currently running process|Number of free pages available in memory|Number of process waiting/blocked for memory|Number of processes which are swapped. A process is swapped if any of its user stack pages or its kernel stack page is swapped out.|Specifies whether swapping is initiated. Swap Out/In represented by 0 and 1 respectively. 0 if paging is not in progress.|

## Start

### Modifications to OS Startup Code

1. Set the User Area page number in the Process Table entry of the current process. We're allocating it at the first available free page, which is **page 80**. The SPL constant `PROCESS_TABLE` points to the starting address of the Process Table (28762).
```
[PROCESS_TABLE + 11] = 80;
```

2. Since we're using the first Process Table entry, the PID will be 0 and we'll store it in the PID field.
```
[PROCESS_TABLE + 1] = 0;
```

3. We set the second field of System Status table to the process which is going to be run in user mode. `SYSTEM_STATUS_TABLE` points to starting address of the table.
```
[SYSTEM_STATUS_TABLE + 1] = 0;
```

4. No need to set kernel stack pointer now as all interrupt handlers assume kernel stack is empty when handler is entered from user mode.

### Timer Interrupt

1. Save current value of User SP into Process Table entry. Get PID of currently executing process from System Status Table. 

```
[PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1] * 16) + 13] = SP;
```

2. Set SP to beginning of kernel stack. Initial value of SP must be set to this (User Area Page Number)*512 - 1.
```
SP = [PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1] * 16) + 11] * 512 - 1;
```

3. Save the user context to kernel stack using the `backup` instruction.
```
backup;
```

4. Print "TIMER".
```
print "TIMER";
```

5. Restore user context from kernel stack and set SP to user SP saved in Process Table.
```
restore;
SP = [PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1]*16) + 13];
```

6. Using `ireturn` to switch to user mode.
```
ireturn;
```

## Assignment

1. Print the process id of currently executing process in timer interrupt before returning to user mode. You can look up this value from the System Status Table.

- Add this line to `sample_timer.spl`.
```
print [SYSTEM_STATUS_TABLE + 1];
```

- Now load OS startup code and timer.
```
Unix-XFS Interace Version 2.0. 
Type "help" for getting a list of commands.
# load --os ../spl/spl_progs/os_startup.xsm
# load --timer ../spl/spl_progs/sample_timer.xsm
```

- Now check if it works by running XSM.
```bash
./xsm --timer 5
```