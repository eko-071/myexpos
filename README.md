# eXperimental Operating System

This is an elementary operating system built from scratch on a virtual machine, following the eXpOS NITC [Roadmap](https://exposnitc.github.io/Roadmap.html).

## Stages

The roadmap is split into 28 stages.

### Preparatory Stages

- [x] [Stage 1 : Setting up the System](documentation/stage_1.md)
- [x] [Stage 2 : Understanding the Filesystem](documentation/stage_2.md)
- [x] [Stage 3 : Bootstrap Loader](documentation/stage_3.md)
- [x] [Stage 4 : Learning the SPL Language](documentation/stage_4.md)
- [ ] [Stage 5 : XSM Debugging](documentation/stage_5.md)
- [ ] [Stage 6 : Running a user program](documentation/stage_6.md)
- [ ] [Stage 7 : ABI and XEXE format](documentation/stage_7.md)
- [ ] [Stage 8 : Handling Timer Interrupt](documentation/stage_8.md)
- [ ] [Stage 9 : Handling kernel stack](documentation/stage_9.md)
- [ ] [Stage 10 : Console output](documentation/stage_10.md)
- [ ] [Stage 11 : Introduction to ExpL](documentation/stage_11.md)
- [ ] [Stage 12 : Introduction to Multiprogramming](documentation/stage_12.md)

### Intermediate Stages

- [ ] [Stage 13 : Boot Module](documentation/stage_13.md)
- [ ] [Stage 14 : Round Robin Scheduler](documentation/stage_14.md)
- [ ] [Stage 15 : Resource Manager Module](documentation/stage_15.md)
- [ ] [Stage 16 : Console Input](documentation/stage_16.md)
- [ ] [Stage 17 : Program Loader](documentation/stage_17.md)
- [ ] [Stage 18 : Disk Interrupt Loader](documentation/stage_18.md)
- [ ] [Stage 19 : Exception Handler](documentation/stage_19.md)

### Advanced Stages

- [ ] [Stage 20 : Process Creation and Termination](documentation/stage_20.md)
- [ ] [Stage 21 : Process Synchronization](documentation/stage_21.md)
- [ ] [Stage 22 : Semaphores](documentation/stage_22.md)
- [ ] [Stage 23 : File Creation and Deletion](documentation/stage_23.md)
- [ ] [Stage 24 : File Read](documentation/stage_24.md)
- [ ] [Stage 25 : File Write](documentation/stage_25.md)
- [ ] [Stage 26 : User Management](documentation/stage_26.md)
- [ ] [Stage 27 : Pager Module](documentation/stage_27.md)
- [ ] [Stage 28 : Multi-Core Extension](documentation/stage_28.md)

## Notes

When you run `make`, on systems with versions of `gcc` newer than 13 (I think), it just fails.
You can run Docker and still do the project, but if you prefer not to do that, you can download an older version of `gcc`, most preferable `gcc-13`, and use that. Here's how to do it.

### Changing Makefiles

This is how your project structure will look after you run `curl`.
> For some reason, the test/ directory is missing. Will have to ask lab faculty what the exact issue is. Not necessary for now.
```
.
├── expl/
├── Makefile
├── spl/
├── xfs-interface/
└── xsm/
```
In each of the four subfolders, there will be a Makefile. This is what the top of each Makefile will look like.
```
CC = gcc
CFLAGS = -g -fcommon
.
.
.
```
Change the `CC = gcc` to `CC = gcc-13`. Now all that's left to do is to actually install `gcc-13`.

### Installing `gcc-13`

#### Debian-based

Download it using `snap` or `apt` or something else.
```bash
sudo snap install gcc-13
```

#### Arch-based

There are 2 options here. 

1. The first one is to compile `gcc-13` from scratch from the Arch User Repository.
```bash
yay -S gcc13
```
Keep in mind one thing though, this is compiling from source code, and thus, will be slow.
Took me around 9 hours on my machine, can take more or less depending on your specs.

2. The alternative is to download the `.pkg.tar.zst` files and use that to directly download `gcc-13`.
You can find these packages floating the Internet, or in the `packages` folder in my repo.
Download the two packages, `gcc13` and `gcc13-libs`. Now open the terminal where the files are, and run these commands:
```bash
sudo pacman -U gcc13-13.4.1+r80+gd6ebfe4-1-x86_64.pkg.tar.zst
sudo pacman -U gcc13-libs-13.4.1+r80+gd6ebfe4-1-x86_64.pkg.tar.zst
```
> I would not recommend installing things this way, because you don't know what exactly is there inside the package. So be careful and verify the package before you set it up.

#### Fedora-based

If you're on Fedora version 41 or older, you should be able to just install it.
```bash
sudo dnf install gcc13
```
For newer versions above 42, compile from source code.
I'm not familiar with Fedora, so I'm leaving this aside.

#### Windows

I don't know, man... Just use Docker.

After this, you should be able to run the `make` without issues.
A few warnings will pop up, but no need to worry about that.
Run any of the executables to confirm it worked.