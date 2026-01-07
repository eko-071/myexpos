# Stage 4: Learning the SPL Language

## Start

1. Creating a program to print odd numbers from 1 to 20 using SPL in `spl/spl_progs/oddnos.spl`.
```
alias counter R0;
counter = 0;
while(counter <= 20) do
  if(counter%2 != 0) then
    print counter;
  endif;
  counter = counter + 1;
endwhile; 
```

2. Compiling it.
```bash
./spl spl_progs/oddnos.spl
```
A `oddnos.xsm` file is compiled.
```
MOV R0, 0
_L1:
MOV R16, 20
GE R16, R0
JZ R16, _L2
MOV R16, R0
MOD R16, 2
MOV R17, 0
NE R16, R17
JZ R16, _L3
MOV R16, R0
PORT P1, R16
OUT
JMP _L4
_L3:
_L4:
MOV R16, R0
ADD R16, 1
MOV R0, R16
JMP _L1
_L2:
HALT
```

4. Loading it as OS startup code.
```bash
cd ../xfs-interface
./xfs-interface
```

```
Unix-XFS Interace Version 2.0. 
Type "help" for getting a list of commands.
# load --os ../spl/spl_progs/oddnos.xsm
# exit
```

5. Running the machine
```bash
cd ../xsm
./xsm
```
Output:
```
1
3
5
7
9
11
13
15
17
19
Machine is halting.
```

## Assignments

1.  Write the spl program to print sum of squares of the first 20 natural numbers. Load it using xfs interface and run the in the machine.

SPL Code:
```bash
alias counter R0;
alias sum R1;
counter = 0;
sum = 0;
while(counter < 21) do
    sum = sum + counter*counter;
    counter = counter + 1;
endwhile;
print sum;
```

Compiling, loading, and running:
```bash
./spl spl_progs/sum_squares.spl
cd ../xfs-interface
./xfs-interface
```

```
Unix-XFS Interace Version 2.0. 
Type "help" for getting a list of commands.
# load --os ../spl/spl_progs/sum_squares.xsm
# exit
```

```bash
cd ../xsm
./xsm
```

Output:
```bash
2870
Machine is halting.
```