# Assembly for CTFs: the parts you actually need

You do not need to learn assembly the way a compiler engineer does. For reverse engineering and pwn challenges you need to read it, not write it, and the same dozen or so instructions show up over and over. This page covers that dozen, plus the handful of registers and the one calling convention you keep running into. Read it once before Day 4, then come back to it whenever a disassembly stops making sense.

If you have never opened a disassembler, that is fine. This assumes you know what a variable and a function are in C, and nothing past that.

## Why you end up reading assembly at all

Ghidra's decompiler turns a binary back into C-like code, and most of the time that is enough. But the decompiler guesses, and sometimes it guesses wrong or gives up and leaves you a block of assembly. gdb also works at the instruction level: when you step through a program to watch a password get built in memory, what you step through is assembly. So you will read it whether you plan to or not. The goal here is for those instructions to look like something you can follow instead of a wall of hex.

## Two syntaxes, same instructions

x86-64 assembly gets written two ways, and this trips people up before anything else does.

- **Intel syntax** puts the destination first: `mov rax, 5` means "put 5 into rax."
- **AT&T syntax** puts the destination last and prefixes registers with `%`: `mov $5, %rax` means the same thing.

On Linux, `objdump -d` gives you AT&T by default. Ghidra and most CTF writeups use Intel, which most people find easier to read. Make objdump match by adding `-M intel`:

```bash
objdump -M intel -d ./program
```

In gdb, run `set disassembly-flavor intel` once (pwndbg already does this). The rest of this page uses Intel syntax.

## Registers: the short list

A register is a small, fast slot inside the CPU that holds one value. There are many. You need these:

| Register | What it holds |
|---|---|
| `rax` | Scratch, and where a function leaves its return value |
| `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9` | The first six arguments passed to a function, in that order |
| `rsp` | Stack pointer: the current top of the stack |
| `rbp` | Base pointer: the bottom of the current function's stack frame |
| `rip` | Instruction pointer: the address of the next instruction to run |

One register, several names by size. `rax` is the full 64-bit register. `eax` is its low 32 bits, `ax` the low 16, `al` the low 8. They are the same storage, just different windows onto it. When you see `eax` and `rax` near each other, they are the same place. This matters because a comparison written as `cmp eax, 0` is checking a 32-bit `int`.

## The instructions you will actually see

**mov** copies a value. `mov rax, rbx` puts the contents of rbx into rax. `mov rax, [rbx]` puts the value stored *at the address in rbx* into rax. The square brackets mean "the memory at this address," the same idea as dereferencing a pointer in C.

**lea** ("load effective address") computes an address without reading memory. `lea rax, [rbp-0x20]` puts the address `rbp-0x20` into rax. You will see it a lot when a function takes the address of a local buffer, for example right before a `gets` or `scanf` call.

**push** and **pop** move things on and off the stack. `push rax` writes rax to the top of the stack and moves rsp down. `pop rax` does the reverse.

**call** and **ret** are how functions work. `call foo` pushes the address of the next instruction (the return address) onto the stack, then jumps to `foo`. `ret` pops that address back off and jumps to it. This is the mechanism Day 6 abuses: overwrite the saved return address on the stack, and `ret` sends execution wherever you pointed it.

**cmp** and **test** compare two values and set flags (see below). `cmp rax, rbx` computes `rax - rbx` and throws away the result, keeping only whether it was zero, negative, and so on. `test rax, rax` is the common way to check whether rax is zero.

**add**, **sub**, **xor**, **and**, **or** do arithmetic and bit operations, destination first. One idiom worth memorizing: `xor eax, eax` sets eax to zero. Anything XOR'd with itself is zero, and this is shorter than `mov eax, 0`, so compilers use it constantly. When you see it, read "eax = 0."

**jmp** and the conditional jumps are the branches. `jmp` always jumps. The conditional ones jump only if the flags from the last `cmp` or `test` say so:

| Jump | Taken when |
|---|---|
| `je` / `jz` | Equal / result was zero |
| `jne` / `jnz` | Not equal / not zero |
| `jg`, `jl` | Greater, less (signed) |
| `ja`, `jb` | Above, below (unsigned) |

You do not need the full list. When you hit one you do not know, look it up. The pattern to recognize is `cmp` (or `test`) immediately followed by a conditional jump. That pair is an `if` in the source.

## Flags, briefly

A `cmp` or `test` does not produce a value you can see. It sets a few status bits called flags, and the next conditional jump reads them. You do not need to track the flags by hand. Read `cmp a, b` then `je somewhere` as one unit: "if a equals b, jump to somewhere." That is enough to follow the logic.

## How arguments get passed

When one function calls another, the first six integer or pointer arguments go into `rdi, rsi, rdx, rcx, r8, r9`, in that order, and the return value comes back in `rax`. So a call that looks like this in C:

```c
strcmp(user_input, "letmein");
```

shows up in assembly as: load `user_input`'s address into rdi, load the address of the string `"letmein"` into rsi, then `call strcmp`. Reading the two `lea`/`mov` instructions right before a `call` tells you what the function is being handed. This one trick, "look at rdi and rsi before the call," solves a large share of beginner reversing challenges.

## Shapes you will keep seeing in CTF binaries

**A password compared to a hardcoded string.** The program loads your input into one register and a constant string into another, then calls `strcmp` or `memcmp` and branches on the result. If the constant is a plain string, `strings` finds it and you are done without reading any assembly at all. Always run `strings` first.

**A loop that transforms data.** A short block that reads a byte, does an `xor` or `add` on it, stores it back, increments a counter, and jumps back to the top. This is usually obfuscation: the real password is built at runtime so `strings` cannot see it. Set a breakpoint after the loop finishes and read the buffer out of memory, or work the transform backward by hand.

**A length check before a copy.** A `cmp` against a fixed number guarding a `call` to something like `memcpy`. When the check is missing or wrong, that is often the overflow.

**A function that is never called.** Ghidra's function list shows a `win` or `print_flag` function, but nothing in the program calls it. On Day 6 your job is to reach it anyway by overwriting a return address with its start address, which you get from `objdump -d | grep win`.

## Making it readable while you work

- Get Intel syntax everywhere (`objdump -M intel -d`, `set disassembly-flavor intel` in gdb).
- Use Ghidra's decompiler for the shape of a function, then drop to the assembly listing for the parts it gets wrong.
- When an instruction is unfamiliar, search "x86 <instruction>" and read one sentence. You are not memorizing the manual, just looking up the word you are missing.
- In gdb, `x/s <address>` prints the string at an address and `x/20xg $rsp` prints twenty stack slots. Those two cover most of what you need to see while stepping.

## Where to go deeper

- [Compiler Explorer (godbolt.org)](https://godbolt.org/): write a line of C, watch the assembly appear next to it. The fastest way to build intuition is to change the C and see which instructions change.
- [x86-64 reference](https://www.felixcloutier.com/x86/): one page per instruction when you need the exact behavior.
- LiveOverflow's reverse engineering and binary exploitation playlists work through real binaries at this level.
