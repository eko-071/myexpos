# Console Input

- This stage is about XSM console interrupt handling.
- A process must use the XSM instruction IN to read data from console into the input port P0.
- This is a privileged instruction and can be executed only inside a system call/module, so a user process will have to invoke the read system call.
- This call invokes the Terminal Read function from the Device Manager module.
- The instruction will not wait for the data to arrive in P0, so there must be some hardware mechanism to detect arrival of data.
- Data arrives in P0 when something is entered from keyboard and ENTER is pressed, which is when the XSM machine will raise the console interrupt.
- Since the process that invoked the Terminal Read function doesn't need to continue execution, it will set its state to WAIT_TERMINAL, invoke the scheduler, and wait for the console interrupt.

## Start

### INT 6

- The final c0de:
```
// Set mode flag to system call number
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 9] = 7;

alias userSP R0;
userSP = SP;

// save SP to UPTR
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 13] = userSP;

SP = [PROCESS_TABLE + ([SYSTEM_STATUS_TABLE + 1] * 16) + 11] * 512 - 1;

alias physicalPageNumber R1;
alias offset R2;
alias physicalAddress R3;
alias page_table R4;

page_table = PAGE_TABLE_BASE + [SYSTEM_STATUS_TABLE + 1] * 20;

physicalPageNumber = [page_table + 2 * ((userSP - 4) / 512)];
offset = (userSP - 4) % 512;
physicalAddress = physicalPageNumber * 512 + offset;

alias fileDescriptor R5;
fileDescriptor = [physicalAddress];

alias retAddr R6;
retAddr = ([page_table + 2 * ((userSP-1) / 512)] * 512) + ((userSP-1) % 512);

if (fileDescriptor != -1) then
    [retAddr] = -1;
else
    physicalPageNumber = [page_table + 2 * ((userSP - 3) / 512)];
    offset = (userSP - 3) % 512;
    physicalAddress = physicalPageNumber * 512 + offset;

    multipush(R0, R1, R2, R3, R4, R5);

    alias arg1 R1;
    alias arg2 R2;
    alias arg3 R3;
    arg1 = 4; // terminal read
    arg2 = [SYSTEM_STATUS_TABLE + 1];
    arg3 = [physicalAddress];

    call MOD_4;

    multipop(R0, R1, R2, R3, R4, R5);

    [retAddr] = 0;
endif;

SP = userSP;
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 9] = 0;

ireturn;
```

### Device Manager

- The final c0de:
```
alias functionNum R1;
alias currentPID R2;
alias word R3;

if (functionNum == 3) then
    // terminal write
    multipush(R1, R2, R3);
    functionNum = 8; // acquire terminal
    call MOD_0;
    multipop(R1, R2, R3);
    print word;

    multipush(R1, R2, R3);
    functionNum = 9; // release terminal
    call MOD_0;
    multipop(R1, R2, R3);

    return;
endif;

if (functionNum == 4) then
    // terminal read
    multipush(R1, R2, R3);
    functionNum = 8; // acquire terminal
    call MOD_0;
    multipop(R1, R2, R3);
    read;

    [PROCESS_TABLE + currentPID * 16 + 4] = WAIT_TERMINAL;
    multipush(R1, R2, R3);
    call MOD_5; // scheduler
    multipop(R1, R2, R3);
    
    alias wordAddr R4;
    wordAddr = [PTBR + 2 * (word / 512)] * 512 + (word % 512);
    [wordAddr] = [PROCESS_TABLE + currentPID*16 + 8]; 

    return;
endif;
```

### Console Interrupt

- The final c0de:
```
[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 13] = SP;
SP = [PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 11] * 512 - 1;

backup;

alias reqPID R0;
alias process_table_entry R1;

reqPID = [TERMINAL_STATUS_TABLE + 1];
process_table_entry = PROCESS_TABLE + reqPID * 16;

[process_table_entry + 8] = P0;

multipush(R0, R1);

alias arg1 R1;
alias arg2 R2;

arg1 = 9;
arg2 = reqPID;

call MOD_0;

multipop(R0, R1);

restore;
SP = [PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 13];
ireturn;
```

### Boot Module

- The final c0de:
```
// Library
loadi(63,13);
loadi(64,14);

// Code: INIT Program
loadi(65,7);
loadi(66,8);

// INT6 module
loadi(14,27);
loadi(15,28);

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

// Console Interrupt
loadi(8, 21);
loadi(9, 22);

// Module 0: Device Manager
loadi(40, 53);
loadi(41, 54);

// Module 4: Terminal Handler
loadi(48, 61);
loadi(49, 62);

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

alias i R6;
i=2;
while(i < 16) do
    [PROCESS_TABLE + 16*i + 4] = TERMINATED;
    i = i+1;
endwhile;

[TERMINAL_STATUS_TABLE] = 0;

return;
```

### Init pr0gram

```
decl
	int gcd(int a, int b);
enddecl

int gcd(int a, int b){
	decl
	    int temp;
	enddecl
	begin
        if(a == 0) then
            temp = b;	
        else
            temp = gcd(b % a, a);
        endif;
        return temp;
	end
}

int main(){
	decl
	    int temp, a, b, result;
	enddecl
	begin
        temp = exposcall("Read", -1, a);
        temp = exposcall("Read", -1, b);
        result = gcd(a, b);
        temp = exposcall("Write", -2, result);
        return 0;
	end
}
```