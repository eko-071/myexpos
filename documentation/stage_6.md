# Running a user program

## Start

```
Unix-XFS Interace Version 2.0. 
Type "help" for getting a list of commands.
# load --init ../expl/expl_progs/squares.xsm
# load --int=10 ../spl/spl_progs/haltprog.xsm
# load --exhandler ../spl/spl_progs/haltprog.xsm
# load --os ../spl/spl_progs/os_startup.xsm
# exit
```

```bash
./xsm --debug --timer 0
```

```
Previous instruction at IP = 12: BRKP
Mode: USER 	PID: 0
Next instruction at IP = 14, Page No. = 0: ADD R0,1
debug> reg 
R0: 1	R1: 1	R2: 1	R3: 	R4: 	
R5: 	R6: 	R7: 	R8: 	R9: 	
R10: 	R11: 	R12: 	R13: 	R14: 	
R15: 	R16: 1024	R17: 	R18: 	R19: 	
P0: 	P1: 	P2: 	P3: 	
BP: 	SP: 1023	IP: 14	PTBR: 29696	PTLR: 3	
EIP: 	EC: 	EPN: 	EMA: 	
debug> c
Previous instruction at IP = 12: BRKP
Mode: USER 	PID: 0
Next instruction at IP = 14, Page No. = 0: ADD R0,1
debug> reg
R0: 2	R1: 4	R2: 1	R3: 	R4: 	
R5: 	R6: 	R7: 	R8: 	R9: 	
R10: 	R11: 	R12: 	R13: 	R14: 	
R15: 	R16: 1024	R17: 	R18: 	R19: 	
P0: 	P1: 	P2: 	P3: 	
BP: 	SP: 1023	IP: 14	PTBR: 29696	PTLR: 3	
EIP: 	EC: 	EPN: 	EMA: 	
debug> c
Previous instruction at IP = 12: BRKP
Mode: USER 	PID: 0
Next instruction at IP = 14, Page No. = 0: ADD R0,1
debug> reg
R0: 3	R1: 9	R2: 1	R3: 	R4: 	
R5: 	R6: 	R7: 	R8: 	R9: 	
R10: 	R11: 	R12: 	R13: 	R14: 	
R15: 	R16: 1024	R17: 	R18: 	R19: 	
P0: 	P1: 	P2: 	P3: 	
BP: 	SP: 1023	IP: 14	PTBR: 29696	PTLR: 3	
EIP: 	EC: 	EPN: 	EMA: 	
debug> c
Previous instruction at IP = 12: BRKP
Mode: USER 	PID: 0
Next instruction at IP = 14, Page No. = 0: ADD R0,1
debug> reg
R0: 4	R1: 16	R2: 1	R3: 	R4: 	
R5: 	R6: 	R7: 	R8: 	R9: 	
R10: 	R11: 	R12: 	R13: 	R14: 	
R15: 	R16: 1024	R17: 	R18: 	R19: 	
P0: 	P1: 	P2: 	P3: 	
BP: 	SP: 1023	IP: 14	PTBR: 29696	PTLR: 3	
EIP: 	EC: 	EPN: 	EMA: 	
debug> c
Previous instruction at IP = 12: BRKP
Mode: USER 	PID: 0
Next instruction at IP = 14, Page No. = 0: ADD R0,1
debug> reg
R0: 5	R1: 25	R2: 1	R3: 	R4: 	
R5: 	R6: 	R7: 	R8: 	R9: 	
R10: 	R11: 	R12: 	R13: 	R14: 	
R15: 	R16: 1024	R17: 	R18: 	R19: 	
P0: 	P1: 	P2: 	P3: 	
BP: 	SP: 1023	IP: 14	PTBR: 29696	PTLR: 3	
EIP: 	EC: 	EPN: 	EMA: 	
debug> c
Machine is halting.
```

## Assignment

1. Change virtual memory model such that code occupies logical pages 4 and 5 and the stack lies in logical page 8. You will have to modify the user program as well as the os startup code. 

The user program:
```
MOV R0, 1 
MOV R2, 5
GE R2, R0
JZ R2, 2066
MOV R1, R0
MUL R1, R0
BRKP
ADD R0, 1
JMP 2050
INT 10
```

The OS startup code:
```
loadi(65,7);
loadi(66,8);

loadi(22,35);
loadi(23,36);

loadi(2,15);
loadi(3,16);

PTBR = PAGE_TABLE_BASE;
PTLR = 9;

[PTBR+8] = 65;
[PTBR+9] = "0100";
[PTBR+10] = 66;
[PTBR+11] = "0100";
[PTBR+16] = 76;
[PTBR+17] = "0110";

[76*512] = 4*512;
SP = 8*512;

ireturn;
```