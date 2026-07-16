# Program Loader

- Implementation of exec system call, the program loader.
- Takes a filename as input.
- First checks wheter the file is a valid executable acc. to XEXE format.
- If it is, exec destroys the invoking process, loads the executable file from disk, and sets up the program for execution as a process.
- Inode index of the file can be obtained by going through the memory copy of the inode table.
- Exec first deallocates all pages the invoking process is using, and invalidates its page table.
- The newly scheduled process will have the same PID as of the invoking process, and the same process table entry and page table will be used.
- Exec does this by calling the `Exit Process` function in the process manager module.