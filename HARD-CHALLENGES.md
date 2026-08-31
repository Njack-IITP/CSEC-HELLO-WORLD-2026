# Hard mode

The daily walkthroughs teach one technique at a time on a challenge built to isolate it. The hard challenges do the opposite. They combine techniques, hide the obvious approach, and expect you to recognize what you are looking at. Nothing here needs a skill the two tracks did not touch, but you will have to put pieces together yourself and read code the decompiler will not hand you cleanly.

This page is a map, not a walkthrough. Each entry names the idea, says why the first thing you try will not work, and points at the technique that does. It does not give away answers. If you have not finished the daily content for a day, do that first. The hard challenge for a day assumes you already have that day's tool in hand.

Flags still look like `NJACK{...}`. Submit them the same way, on the CTFd scoreboard.

## Track A (Networks)

### The code is already on your machine (Day 1)

A page can lock a value behind a password box and still be giving that value away, because everything the page needs to unlock it was sent to your browser. The password check is theater. Open the Sources tab in DevTools, read the JavaScript, and notice that the secret is built from an array right there in the script. You can call the page's own functions from the Console to have it decode the value for you, or work the small transform (usually an XOR) by hand. The lesson is the one in the flag: client-side is not a security boundary.

### Read the token, then break its signature (Day 2)

A JWT is three Base64 chunks split by dots. Decoding the middle chunk to read the payload is a Day 2 skill. The harder version gives you a token whose signature was made with a weak secret. Because the signing input (`header.payload`) is public, you can take a wordlist, compute the HMAC-SHA256 of the signing input under each candidate secret, and stop when one reproduces the signature on the token. Tools like `jwt_tool` or `hashcat` automate this, but a short Python loop with `hmac` does it too. Once you hold the secret, anything else that was derived from it (a keystream, a re-signed token) is yours to reproduce.

You need a wordlist for this. A small [starter list](./wordlists/starter-wordlist.txt) is in this repo and is enough for the JWT challenge. For real cracking, use rockyou.txt: it ships with Kali at `/usr/share/wordlists/rockyou.txt.gz` (run `gunzip` on it once), or download it from [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Leaked-Databases/rockyou.txt.tar.gz). It is about 14 million passwords, too large to keep in this repo.

### One-time pads are only safe once (Day 3)

A single-byte XOR you brute force. A long random keystream you cannot, unless the author reused it. If two messages are XOR'd with the same keystream, then XORing the two ciphertexts together cancels the key and leaves you `plaintext1 XOR plaintext2`. From there you guess a word likely to appear in one message (a "crib"), slide it along, and read the other message where it lines up. This is called crib dragging. The mistake, not the math, is what breaks it.

### Rebuild what left in pieces (Days 4 and 5)

Some captures do not hand you a clean HTTP stream. Data can leave a network one fragment at a time, smuggled inside the names of DNS queries, or arrive split across many small packets that showed up out of order. Wireshark still shows you every packet. Your job is to pull the piece out of each one (a chunk of a hostname, a numbered fragment in a payload), put them back in order, and then decode the result. It is usually hex or Base64 once reassembled. `tshark -r file.pcap -T fields -e ...` can dump one field from every packet so you are not copying by hand.

### Read the whole certificate (Day 6)

A full service scan can look like a dead end where nothing is exploitable. That is the point: the leak is passive. Read every field of the TLS certificate in the scan output, including the Subject Alternative Names. A name that shows up nowhere else is the answer. This is the same idea as the crt.sh exercise, applied to a cert in front of you.

### Peel the layers in order (Day 7)

When a blob has been encoded several times, you undo the encodings in reverse. Recognize each layer by its shape: hex is `0-9a-f`, Base64 ends in `=` and uses mixed case, Base32 is uppercase and digits `2-7`, and gzip starts with the bytes `1f 8b`. CyberChef's "Magic" operation can detect and suggest layers, but doing it by hand teaches you to read an alphabet and know what it is. Reversing a string is also an encoding step, so watch for one.

## Track B (Computers)

### The flag is never a string (Day 1)

`strings` finds text. It finds nothing here because the flag is never stored as text. It exists only as the combination of two byte arrays that live in different sections, one writable (`.data`) and one read-only (`.rodata`). Pull both arrays out of the binary and XOR them together. You can read the bytes straight out of the disassembly or Ghidra, or let the program compute the value and read it from memory in gdb rather than reimplementing the loop.

### Invert the check instead of guessing it (Days 2, 5, 7)

A whole family of crackmes works the same way: the program transforms your input byte by byte and compares the result to a stored target. Guessing is hopeless, but the transform is usually invertible. XOR is its own inverse. Addition undoes subtraction. A bit rotation left by `n` is undone by a rotation right by `n`. If you can read the operations in the decompiler and read the stored target bytes, you run the transform backward on the target and recover the input the program wants. When the transform runs on a small hand-rolled interpreter (a "bytecode VM": a loop that reads an opcode and a number and applies one operation per step), reverse-engineer the opcode table first, then invert the program it runs. When passing the check also decrypts a second payload, the input that satisfies the check is the same key that decrypts the payload, so solving the first stage hands you the second.

### Bit rotations

Rotations show up constantly in these transforms and are worth naming, since the daily content only covers XOR. A rotate-left by 3 (`rol`, or in C `(c << 3) | (c >> 5)` for a byte) shifts bits off the top and wraps them to the bottom. It looks like a shift-and-or pair in the disassembly. To undo a `rol` by `n`, rotate right by `n` (`ror`), which is `(c >> n) | (c << (8 - n))` for a byte. Recognizing this pair in a loop tells you the byte is being rotated, not just shifted.

### When the program knows it is being watched (Day 3)

A binary can detect a debugger by calling `ptrace(PTRACE_TRACEME)` on itself. A process can only be traced once, so if your debugger already attached, the program's own call fails, and it uses that result to take a wrong branch or a wrong key. Three ways past it: reverse the logic statically so you never run it under a debugger, run it normally and only compute the answer by hand, or neutralize the check (patch the branch, or set the return value of the `ptrace` call so the program believes it is alone). Recognize the tell first: a `ptrace` call early in `main` whose result feeds a comparison.

### Recognize the function, do not brute force it (Day 4)

Some checks run your input through a well-known mixing function, the kind used inside hash functions and fast random-number generators (splitmix64 and MurmurHash finalizers are common). You will recognize them by their magic constants (large hex numbers like `0xff51afd7ed558ccd`) and their shape: xor-shift, multiply, xor-shift, multiply, xor-shift. These functions are bijections, which means they are invertible. Each step reverses: an xor-shift undoes itself, and a multiply by a constant undoes with a multiply by that constant's inverse modulo 2^64. Look the function up by its constants, find the published inverse, and run the target backward. Brute force is not the intended path.

### No single win (Day 6)

The daily overflow gives you one `win()` to jump to. The hard version does not. One function flips an "authorized" flag, a second prints the flag but only when authorized. Overwriting the return address with one of them is not enough, because you need both to run, in order. After the first return address you place a second one, so that when the first function returns it lands in the next. This is the first step toward return-oriented programming: the stack becomes a list of addresses to visit. Get the offset the same way as Day 6 (a cyclic pattern), then stack the addresses instead of using just one.

## When you are stuck

- Re-read the daily walkthrough for that day. The hard challenge extends it; it does not replace the fundamentals.
- Name what you are looking at before you attack it. "This is a reused keystream" or "this is a bit rotation" is most of the work.
- Ask in the CSEC Discord. Hints get released for the hard challenges too, on a longer cooldown than the daily ones.
