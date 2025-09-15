# Demonstration Scenarios: Fusion + Redirection

This folder contains tested examples demonstrating how the Binary Fusion tool fuses multiple ELF executables and handles output redirection.

This document demonstrates how to:
- Fuse 2 or more ELF binaries into a single executable
- Capture their individual outputs via redirection
- Use compression to reduce fused size

---

## 1. Demo: Fusing `jules` and `vincent`

### Source Files
- `build/jules`: prints "What do they call it?"
- `build/vincent`: prints "They call it a Royale with Cheese."

### Fused Binary
- `fused_jules_vincent`: Created by running:
  ```bash
  ./fuser build/jules build/vincent -o demo/fused_jules_vincent
  ```

### Redirected Output
- `output_jules.txt`: contains output from jules
- `output_vincent.txt`: contains output from vincent

### Example Test Run
```bash
./demo/fused_jules_vincent > demo/output_jules.txt 2> demo/output_vincent.txt
```
---

## 2. Demo: Fusing Copies of System Executables `ls` and `whoami`

### Source Files

- To safely test with real Linux executables, create copies of system executables (if not done already): 
  ```bash
  cp /usr/bin/ls build/ls-copy
  cp /usr/bin/whoami build/whoami-copy
  chmod +x build/ls-copy
  chmod +x build/whoami-copy
  ```

- `build/ls-copy`: prints a list of directory contents, displaying files and subdirectories
- `build/whoami-copy`: prints the username of the effective user in the current shell

### Fused Binary
- `fused_ls_whoami`: Created by running:
  ```bash
  ./fuser build/ls-copy build/whoami-copy -o demo/fused_ls_whoami
  ```

### Redirected Output
- `output_ls.txt`: contains output from ls
- `output_whoami.txt`: contains output from whoami

### Example Test Run
```bash
./demo/fused_ls_whoami > demo/output_ls.txt 2> demo/output_whoami.txt
```
---

## 3. Demo: Fusing All Four Executables with Compression Enabled

### Source Files
- `build/jules`: prints "What do they call it?"
- `build/vincent`: prints "They call it a Royale with Cheese."
- `build/ls-copy`: prints a list of directory contents, displaying files and subdirectories
- `build/whoami-copy`: prints the username of the effective user in the current shell

### Fused Binary
- `fused_all`: Created by running the program with the `-c` flag:
  ```bash
  ./fuser build/jules build/vincent build/ls-copy build/whoami-copy -o demo/fused_all -c 
  ```
- Note the compressed size of the `fused_all` file (check with e.g. `ls -la demo/`)

### Example Test Run of Redirected Output
- The output `.txt` files are not provided in the repo for the second time, but you can test that they are created correctly with e.g.:
    ```bash
    ./demo/fused_all > demo/output_all_jules.txt 2> demo/output_all_vincent.txt 3> demo/output_all_ls.txt 4> demo/output_all_whoami.txt
    ```

### Redirected Output Created with Above Command
- `output_all_jules.txt`: contains output from jules
- `output_all_vincent.txt`: contains output from vincent
- `output_all_ls.txt`: contains output from ls
- `output_all_whoami.txt`: contains output from whoami
