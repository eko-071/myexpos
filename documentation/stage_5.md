# XSM Debugging

## XSM Simulator

- Used to simulate the XSM hardware.
- Running it within the `myexpos/xsm` directory:
```
./xsm [--timer #1] [--disk #2] [--console #3] [--debug]
```
1. --timer value:
- Sets number of user mode instructions after which timer interrupt is triggered.
- 0 $\leq$ value $\leq$ 1024
- Default: 20
2. --disk value
- Sets number of user mode instructions after which disk interrupt is triggered.
- Count begins after LOAD or STORE instruction is executed.
- 20 $\leq$ value $\leq$ 1024
- Default: 20
3. --console value
- Sets number of user mode instructions after which console interrupt is triggered.
- Count begins after IN instruction is executed.
- 20 $\leq$ value $\leq$ 1024
- Default: 20
4. --debug
- Sets the machine into DEBUG mode when it encounters a BRKP machine instruction.

## Start

1. Writing code to generate odd numbers from 1 to 10, and adding a debug instruction in between.
```
alias counter R0;
counter = 0;
while(counter <= 10) do
  if(counter%2 != 0) then
    breakpoint;
  endif;
  counter = counter + 1;
endwhile; 
```

2. Compiling and loading it as OS Startup code.
```bash
./spl spl_progs/oddnos_debug.spl
cd ../xfs-interface
./xfs-interface
```

```
Unix-XFS Interace Version 2.0. 
Type "help" for getting a list of commands.
# load --os ../spl/spl_progs/oddnos_debug.xsm
# exit
```

3. Running XSM machine in `debug` mode.
```bash
cd ../xsm
./xsm --debug
```

4. The machine pauses after execution of the first `BRKP` command. We can use `reg` to view the contents of registers, and `mem 1` which writes the contents of memory page 1 to the file mem inside the `xsm/` folder. `s` steps to the next instruction, and `c` continues execution till the next `BRKP` instruction. We can see that the content of `R0` register changes during each iteration.
```
Previous instruction at IP = 530: BRKP
Mode: KERNEL     PID: -1
Next instruction at IP = 532, Page No. = 1: JMP 534
debug> reg
R0: 1   R1:     R2:     R3:     R4: 
R5:     R6:     R7:     R8:     R9: 
R10:    R11:    R12:    R13:    R14: 
R15:    R16: 1  R17: 0  R18:    R19: 
P0:     P1:     P2:     P3: 
BP:     SP:     IP: 532 PTBR:   PTLR: 
EIP:    EC:     EPN:    EMA: 
debug> mem 1
Page: 1
Written to file mem
debug> s
Previous instruction at IP = 532: JMP 534
Mode: KERNEL     PID: -1
Next instruction at IP = 534, Page No. = 1: MOV R16,R0
debug> c
Previous instruction at IP = 530: BRKP
Mode: KERNEL     PID: -1
Next instruction at IP = 532, Page No. = 1: JMP 534
debug> reg R0
R0: 3
debug> c
Previous instruction at IP = 530: BRKP
Mode: KERNEL     PID: -1
Next instruction at IP = 532, Page No. = 1: JMP 534
debug> reg R0
R0: 5
debug> c           
Previous instruction at IP = 530: BRKP
Mode: KERNEL     PID: -1
Next instruction at IP = 532, Page No. = 1: JMP 534
debug> reg R0
R0: 7
debug> exit
Killing the machine
```