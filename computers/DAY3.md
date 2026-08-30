# Day 3: OS Fundamentals

## What You'll Learn Today

How the operating system manages programs: processes, syscalls, permissions, and how to trace what a program actually does under the hood with `strace`.

## Core Concepts

### Processes and Threads

A **process** is a running program. It gets its own memory space, its own file descriptors, its own identity. A **thread** is a lightweight unit of execution inside a process. Threads share memory but run independently.

### Syscalls

Programs can't directly touch hardware, open files, or send network traffic. They ask the kernel to do it via **system calls** (syscalls). `read`, `write`, `open`, `mmap`, `execve`: everything a program does to interact with the outside world goes through a syscall.

### Privilege Rings (Conceptual)

The CPU has privilege levels. User programs run in **ring 3** (restricted). The kernel runs in **ring 0** (full access). A syscall is the controlled gateway between the two. You don't need the ring math. Just the concept: your program can't do anything the kernel doesn't allow.

### Virtual Memory

Every process thinks it owns all of memory. The OS and CPU translate virtual addresses to physical addresses behind the scenes. Process A's address `0x400000` and Process B's address `0x400000` point to different physical memory. This is why one process can't accidentally (or intentionally) read another's memory.

## Hands-On

### 1. strace: See the Syscalls

Run `strace` on your hello program from Day 1:
```bash
strace ./hello
```

You'll see every syscall: `execve`, `mmap`, `write`, and more. The `write(1, "Hello from NJACK!\n", 18)` line is your `printf`. It became a `write` syscall to file descriptor 1 (stdout).

### 2. Compare with a File-Opening Program

Create `readfile.c`:
```c
#include <stdio.h>
int main() {
    FILE *f = fopen("/etc/hostname", "r");
    char buf[256];
    if (f) { fgets(buf, sizeof(buf), f); printf("%s", buf); fclose(f); }
    return 0;
}
```

```bash
gcc -o readfile readfile.c
strace ./readfile
```

Compare the two `strace` outputs. The second one has `openat`, `read`, `close`, syscalls for file I/O that the first program never needed.

### 3. Permissions and ps

```bash
ls -la /etc/shadow          # you can't read this (permissions)
ps aux | head -20            # running processes
ps aux | grep <your-user>   # just your processes
```

## Practice: OverTheWire Bandit

[OverTheWire Bandit](https://overthewire.org/wargames/bandit/) is a free wargame where each level requires a different Linux command to get the password for the next level. Levels 0 through 5 cover `ssh`, `cat`, `find`, `grep`, and file permissions, all things you've seen today.

SSH into the first level:
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# password: bandit0
```

Work through levels 0-5. Each password you find is also the login for the next level.

## Resources

- [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)
- `man strace`
