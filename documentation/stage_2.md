# Stage 2: Understanding the Filesystem

## **eXpFS:** eXperimental File System

- Assumes the disk is a sequence of blocks, with each block storing a sequence of words.
- **Attributes:** `name`, `size`, `type`, `username`, and `permission`
> Note: `username` and `permission` are part of extended eXpOS
- Each attribute is one word long.
- File name must be a unique string.
- Size of file = total number of words stored in the file.
- Each data/executable file can span across four data blocks at most.
- Index to these blocks and file name and size stored in Inode table (blocks 3 and 4).

### Root File

- Has the name `root` and contains the metadata for each file stored in the system.
- Root entry for a file = 5-tuple of attributes
- First root entry is for the root itself.

### Data Files

- Sequence of words.
- Recommended to use `.dat` extension.
- Every file other than `root` and executable files are treated as data files.

### Executable Files

- Contains executable code for programs that can be loaded and run by the operating system.
- Can only be created externally and loaded using XFS Interface.
- File format recognized here = `XEXE` or eXperimental EXEcutable File
- 2 sections: first is header, and second is code section.

## **XFS Interface:** eXperimental File System Interface

- External interface to access eXpFS from host system.
- Simulated on a binary file `disk.xfs`.
- Can format the disk, dump disk data structures, load and remove files, list files, transfer data and executable files b/w eXpOS and host, and copy specified blocks of XFS disk to file on host system.
- Owner of files loaded through XFS is `root` and `userid` is 1.

## **XSM:** eXperimental String Machine

- Machine simulator consisting of processor, memory, and disk.
- Completely empty except for boot ROM, so we'll have to make code in host system and insert into machine.
- Disk contains 512 blocks, each having 512 words.

## Start

Creating a text file and loading to XFS disk with interface.

1. Starting XFS Interface
```bash
z xfs-interface
./xfs-interface
```

2. Formatting raw disk to eXpFS. This creates a `disk.xfs` file in the folder, and initialises disk data structures.
```bash
fdisk
exit
```

3. `Disk Free List` keeps track of used and unused blocks in disk. Unused = 0, and used = 1. Use `df` to view it. The first 69 blocks (0 to 68) are reserved, and remaining 443 are free.

4. Created a file `sample.dat` in host and loading it to XFS disk.
```bash
load --data ../sample.dat
```

5. Copying Inode Table (blocks 3 and 4) to file on host system.
```bash
copy 3 4 ../inode_table.txt
```
We can also use the `dump` command to put contents of inode table in the `xfs-interface` folder.
```bash
dump --inodeusertable
```

6. Checking disk free list, 69th block is used. We'll copy that to a file in host. This shows each word in a line, since a word is 16 characters long.
```bash
copy 69 69 ../data.txt
```

7. `xfs-interface` provides the `export` command to export files for XSM to host. Exporting `sample.dat` to host system.
```bash
export sample.dat ../data.txt
```

## Question

When a file is created entries are made in the Inode table as well as the Root file. What is the need for this duplication?

**Answer**: Inode table is accessible only in Kernel mode, while Root file is accessible both in Kernel and User mode. This allows the user to view the files from an application program by reading the Root file.

## Assignments

1. Copy the contents of Root File (from Block 5 of XFS disk) to a UNIX file `$HOME/myexpos/root_file.txt` and verify that an entry for `sample.dat` is made in it also.

```bash
./xfs-interface
copy 5 5 ../root_file.txt
exit
z ..
cat root_file.txt
```

2.  Delete the `sample.dat` from the XSM machine using `xfs-interface` and note the changes for the entries for this file in inode table, root file and disk free list .
```bash
./xfs-interface
rm sample.dat
df
exit
```