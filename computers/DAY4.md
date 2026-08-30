# Day 4: The Toolkit

## What You'll Learn Today

The two tools you'll use for the rest of the week (and beyond): **Ghidra** (disassembler/decompiler) and **gdb** (debugger). You'll also learn to recognize common C control-flow patterns in assembly (`if`, `for`, `while`) so disassembly stops looking like random instructions.

**Setup check:** You should have Ghidra and pwndbg installed before today. If not, see the track README for install instructions. Do this first.

## Core Concepts

### Disassemblers vs. Debuggers

- A **disassembler** (`objdump`, Ghidra) shows you the assembly from a binary statically, without running it
- A **debugger** (`gdb`) lets you run the binary and stop it mid-execution to inspect registers, memory, and the stack

Ghidra goes further: it has a **decompiler** that tries to turn assembly back into C-like pseudocode. It's not always perfect, but it's usually good enough to understand what a function does.

### Recognizing ASM Patterns

These three shapes cover most of what you'll see:

**if-else**: a comparison (`cmp`) followed by a conditional jump (`je`, `jne`, `jg`, etc.) that skips one block

**for loop**: an initialization, a comparison, a conditional jump to exit, a body, an increment, and an unconditional jump back to the comparison

**while loop**: similar to `for`, but no separate initialization/increment. Just a comparison and a jump-back

### gdb Essentials

| Command | What it does |
|---|---|
| `break main` | Set a breakpoint at main |
| `run` | Start the program |
| `si` / `ni` | Step one instruction (into calls / over calls) |
| `info registers` | Show all register values |
| `x/20xg $rsp` | Examine 20 quad-words at the stack pointer |
| `disassemble` | Show disassembly of the current function |

## Hands-On

### 1. Write Three Functions

Create `patterns.c`:
```c
#include <stdio.h>

int check_positive(int x) {
    if (x > 0) return 1;
    else return 0;
}

int sum_to_n(int n) {
    int total = 0;
    for (int i = 1; i <= n; i++) total += i;
    return total;
}

int count_down(int n) {
    int steps = 0;
    while (n > 0) { n--; steps++; }
    return steps;
}

int main() {
    printf("%d\n", check_positive(-3));
    printf("%d\n", sum_to_n(5));
    printf("%d\n", count_down(4));
    return 0;
}
```

### 2. Disassemble and Match

```bash
gcc -O0 -o patterns patterns.c
objdump -d patterns
```

For each of the three functions, find:
- The comparison instruction (`cmp`)
- The conditional jump (`je`, `jne`, `jle`, etc.)
- The jump-back that makes a loop (for `sum_to_n` and `count_down`)

### 3. Open in Ghidra

Open `patterns` in Ghidra. Navigate to `check_positive`. Compare the decompiler's output to your original source. It should be recognizable, even if the variable names are gone.

### 4. Step Through in gdb

```bash
gdb ./patterns
break check_positive
run
disassemble
si
info registers
```

Watch `rip` advance instruction by instruction. See the comparison happen and the jump taken (or not).

## Mini-Challenge: Spot the Pattern

You'll be given the disassembly of these three functions with the function names removed. Identify which block is the `if-else`, which is the `for` loop, and which is the `while` loop. Point to the instruction(s) that gave it away.

Submit on the CTFd scoreboard.

## Resources

- [Compiler Explorer (godbolt.org)](https://godbolt.org/): paste C, see assembly instantly
- LiveOverflow, "Reverse Engineering" playlist
