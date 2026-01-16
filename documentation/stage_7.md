# ABI and XEXE Format

## Start

## Assignment

1.  Change the user program to compute cubes of the first five numbers.

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
JZ R2, 2076
MOV R1, R0
MUL R1, R0
MUL R1, R0
BRKP
ADD R0, 1
JMP 2058
INT 10
```