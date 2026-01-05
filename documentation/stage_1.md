# Stage 1: Setting up the System

1. Installed needed prerequisites.
```bash
sudo pacman -S readline flex bison make gcc wget curl
```

2. Used `curl` to download the files.
```bash
curl -sSf https://raw.githubusercontent.com/eXpOSNitc/expos-bootstrap/main/download.sh | sh
```

3. Changed to the directory.
```bash
z myexpos
```

4. Run 'make' to build the code.
```bash
make
```

The project structure looks like this now.

```
.
├── expl
├── Makefile
├── spl
├── xfs-interface
└── xsm
```