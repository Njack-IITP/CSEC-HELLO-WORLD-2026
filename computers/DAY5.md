# Day 5: Reverse Engineering

## What Today Is

The first real payoff day. You're going to crack a password-protected binary. Figure out the secret password without ever being told it, using only the tools from this week.

Two challenges, increasing difficulty:
1. **crackme1**, solvable in under 5 minutes with a single command
2. **crackme2** (stretch), the password is hidden at runtime, requiring dynamic analysis or manual deobfuscation

## Challenge 1: crackme1

You'll be given a compiled binary called `crackme1`. It asks for a password and says "Correct!" or "Nope."

Your job: find the password without guessing.

### Approach: Static Analysis

Before running it under gdb or opening Ghidra, try the simplest thing first:
```bash
strings crackme1
```

`strings` dumps every printable string embedded in the binary. If the password is stored as a plaintext string literal... it'll be right there.

Try the string you found as the password. Done.

### Why This Worked

The developer stored the password as a string constant. The compiler put it in the `.rodata` section (remember Day 1?). `strings` reads that section. No encryption, no obfuscation. The secret was sitting in the binary the whole time.

Real challenges avoid this rookie mistake. Which brings us to...

## Challenge 2: crackme2 (Stretch)

Same idea, binary asks for a password, but `strings` won't help this time. The password is obfuscated and only decoded at runtime.

### Approach: Dynamic Analysis

1. Open in gdb: `gdb ./crackme2`
2. Disassemble `main`. Look for a function call that transforms data before the comparison
3. Set a breakpoint **after** that function returns
4. Run the program, hit the breakpoint
5. Examine the buffer it wrote to: `x/s <address>`
6. The decoded password is in that buffer

### Alternative: Static Analysis

Read the disassembly (or Ghidra's decompiler output) and identify the transformation. It's a simple XOR loop. If you can read the XOR key and the encrypted data, you can undo it by hand without ever running the binary.

## Mini-Challenge

Submit the password for crackme1 (required) and crackme2 (stretch) on the CTFd scoreboard.

## Hints

If you're stuck, ask in Discord. Mentors will release hints progressively. Don't worry about needing them, that's what they're for.

## Resources

- `man strings`
- LiveOverflow, "Reverse Engineering" playlist
