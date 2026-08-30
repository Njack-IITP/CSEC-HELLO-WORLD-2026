# Computers Track

## What You'll Learn

How computers work under the hood, from source code to binary, memory layout to OS fundamentals, and your first look at reverse engineering and binary exploitation. This track feeds into the pwn, bin, and rev roles.

By the end of this track, you'll be able to:

- Explain how C becomes a running program: the compilation pipeline, ELF structure, and how to read a binary without running it
- Understand memory and the stack: how function calls work at the register level, what sits on the stack and why
- Navigate a Linux system: processes, syscalls, permissions, `strace`
- Read assembly: recognize `if`, `for`, `while` patterns in disassembly; use Ghidra and gdb
- Reverse a simple binary: static and dynamic analysis to crack a password check
- Exploit a buffer overflow: redirect execution by overwriting a return address

## Schedule

| Day | Theme |
|---|---|
| [1](DAY1.md) | Source to Binary |
| [2](DAY2.md) | Memory and the Stack |
| [3](DAY3.md) | OS Fundamentals |
| [4](DAY4.md) | The Toolkit (Ghidra + gdb) |
| [5](DAY5.md) | Reverse Engineering |
| [6](DAY6.md) | Pwn / Binary Exploitation |
| [7](DAY7.md) | Joint Capstone |

Content is released one day at a time.

## Setup

Install these **before Day 1**. Some (especially Ghidra) take a while:

- A Linux environment: Kali, Ubuntu, or WSL2. Needs `gcc`, `gdb`, and standard binutils (`objdump`, `readelf`, `file`, `strace`)
- pwndbg, a gdb plugin that makes gdb usable: [install guide](https://github.com/pwndbg/pwndbg)
- Ghidra, free disassembler/decompiler (needed from Day 4): [download](https://ghidra-sre.org/)
- Python 3 + pwntools (`pip install pwntools`), needed for Day 6

Give your VM at least 2 CPU cores and 4GB RAM. Ghidra wants headroom.

### On macOS?

This track works natively on Mac. The concepts are identical, a few tool names differ:

| Linux tool | macOS equivalent | Install |
|---|---|---|
| `gcc` | `clang` (the `gcc` command works too) | `xcode-select --install` |
| `objdump -d` | `otool -tV` | already installed |
| `readelf -S` | `otool -l` | already installed |
| `file`, `strings` | same | already installed |
| `strace` | `dtruss` (needs `sudo`) | already installed |
| `gdb` | `lldb` | already installed |
| pwndbg (gdb plugin) | not needed, `lldb` has built-in equivalents | — |

macOS compiles to **Mach-O** format instead of ELF. The sections have different names (e.g. `__TEXT` instead of `.text`) but the concepts are the same. Each day file has a "macOS?" note where commands differ.

**Ghidra** and **pwntools** install and run natively on macOS.

For Days 5-6, the challenge binaries you're given are Linux (ELF) format. To run them, use Docker:
```bash
docker run -it --cap-add=SYS_PTRACE -v $(pwd):/work ubuntu bash
# inside the container: /work/crackme1, /work/overflow1, etc.
```
Ghidra can analyze these ELF binaries on your Mac without Docker.

## Resources

- [Compiler Explorer (godbolt.org)](https://godbolt.org/): instant assembly from C
- [OverTheWire Bandit](https://overthewire.org/wargames/bandit/): CLI fluency practice
- [pwn.college](https://pwn.college/): deeper binary exploitation after this track
- [LiveOverflow](https://www.youtube.com/c/LiveOverflow): Binary Exploitation & Reverse Engineering playlists
