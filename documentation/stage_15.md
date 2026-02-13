# Resource Manager Module

- Before the use of a resource, a process has to invoke a **resource manager** to acquire the required resource.
- When a process releases a resource, the state of other processes waiting for the resource must be set to READY.
- There are 2 functions in the module here: one to acquire the resource, and the other to release the resource.
- Right now, we'll learn how the terminal is shared by processes for writing.
- We have a data structure called the **Terminal Status Table** to keep track of the process that has acquired the terminal.
- The 2 functions here are **Acquire Terminal** (8) and **Release Terminal** (9).

## Terminal Status Table

| Status | PID | Unused |
| ----- | ----- | ----- |
| Specifies whether the terminal is free (0) or is being used by a process to read or write (1). Default is 0.  | Specifies the PID of the process currently using the terminal. Invalid when STATUS is 0. | Not used. |

## Start

### INT 7 

- The final c0de:
```
// Set mode flag to system call number
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 9] = 5;

alias userSP R0;
userSP = SP;

// store user stack pointer in UPTR
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 13] = SP;

// Set SP to start of kernel stack
SP = [PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 11] * 512 - 1;

alias physicalPageNumber R1;
alias offset R2;
alias physicalAddress R3;

physicalPageNumber = [PTBR + 2 * ((userSP - 4) / 512)];
offset = (userSP - 4) % 512;
physicalAddress = physicalPageNumber * 512 + offset;

alias fileDescriptor R4;
fileDescriptor = [physicalAddress];
if (fileDescriptor != -2) then
    physicalPageNumber = [PTBR + 2 * ((userSP - 1) / 512)];
    offset = (userSP - 1) % 512;
    [physicalPageNumber * 512 + offset] = -1;
else 
    physicalPageNumber = [PTBR + 2 * ((userSP - 3) / 512)];
    offset = (userSP - 3) % 512;
    alias word R5;
    word = [physicalPageNumber * 512 + offset];

    multipush(R0, R1, R2, R3, R4, R5);

    alias arg1 R1;
    alias arg2 R2;
    alias arg3 R3;

    arg1 = 3;
    arg2 = [SYSTEM_STATUS_TABLE  + 1];
    arg3 = word;

    call MOD_4;

    multipop(R0, R1, R2, R3, R4, R5);

    physicalPageNumber = [PTBR + 2 * ((userSP - 1) / 512)];
    offset = (userSP - 1) % 512;
    [physicalPageNumber * 512 + offset] = 0;
endif;

SP = userSP;
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 9] = 0;

ireturn;
```

### Device Manager (Module 4)

- The final code:
```
alias functionNum R1;
alias currentPID R2;
alias word R3;

if (functionNum != 3) then
    return;
else
    multipush(R1, R2, R3);
    functionNum = 8;

    call MOD_0;

    multipop(R1, R2, R3);
    print word;
    multipush(R1, R2, R3);

    functionNum = 9;
    call MOD_0;
    multipop(R1, R2, R3);

    return;
endif;
```

### Terminal Handler (Module 0)

- The final code:
```
alias functionNum R1;
alias currentPID R2;

if (functionNum == 8) then // Acquire Terminal
    while ([TERMINAL_STATUS_TABLE] != 0) do
        [PROCESS_TABLE + currentPID * 16 + 4] = WAIT_TERMINAL;
        multipush(R1, R2);

        call MOD_5;

        multipop(R1, R2);
    endwhile;

    [TERMINAL_STATUS_TABLE] = 1;
    [TERMINAL_STATUS_TABLE + 1] = currentPID;
    return;
else
if (functionNum == 9) then // Release Terminal
    if (currentPID != [TERMINAL_STATUS_TABLE + 1]) then
        alias returnValue R0;
        returnValue = -1;
        return; 
    else
        [TERMINAL_STATUS_TABLE] = 0;
        alias i R3;
        i = 0;
        while (i < 16) do
            if ([PROCESS_TABLE + i * 16 + 4] == WAIT_TERMINAL) then
                [PROCESS_TABLE + 16 * i + 4] = READY;
            endif;
            i = i + 1;
        endwhile;
        alias returnValue R0;
        returnValue = 0;
        return;
    endif;
else
    alias returnValue R0;
    returnValue = -1;
    return;
endif;
endif;
```

## Questions

1. According to eXpOS resource management system introduced here, will Deadlock occur? If yes, explain it with a situation. If no, which of the four conditions of Deadlock are not satisfied?

- No, I don't think deadlock will happen here as we only have one resource here. The conditions left unsatisfied here are **hold and wait** and **circular wait**.

## Assignment

1. Set a breakpoint (see SPL breakpoint instruction) just before return from the Acquire Terminal and the Release Terminal functions in the Resource Manager module to dump the Terminal Status Table (see XSM debugger for various printing options).

- Just set a breakpoint before return, not much to do here.