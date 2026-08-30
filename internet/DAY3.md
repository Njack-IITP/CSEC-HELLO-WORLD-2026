# Day 3: Cryptography & the XOR Atom

## What You'll Learn Today

The difference between encoding, hashing, and encryption, and why it matters. You'll break a weak cipher by brute force today, which is the clearest way to understand why real encryption uses massive key spaces.

## Core Concepts

### Encoding vs. Hashing vs. Encryption

| | Reversible? | Needs a key? | Example |
|---|---|---|---|
| **Encoding** (Base64, hex) | Yes | No | Representation, not protection |
| **Hashing** (MD5, SHA-256) | No (one-way) | No | Fingerprinting: same input always gives same output |
| **Encryption** (AES, RSA) | Yes | Yes | Actual protection, unreadable without the key |

### Symmetric vs. Asymmetric

- **Symmetric**: same key encrypts and decrypts (AES). Fast, but you need a way to share the key safely.
- **Asymmetric**: public key encrypts, private key decrypts (RSA). Slower, but you can share the public key openly.

### Why Websites Never Store Your Password

They store a **hash** of it. When you log in, they hash what you typed and compare the hashes. Even if the database leaks, the original passwords aren't there, just fingerprints.

### Why Tiny Key Spaces Die

A single-byte XOR cipher has 256 possible keys. A computer can try all 256 in milliseconds. AES-256 has 2^256 possible keys. Brute force isn't an option.

## Hands-On (CyberChef)

Open [CyberChef](https://gchq.github.io/CyberChef/) for all of these.

### 1. Encoding vs. Hashing

- Base64-encode a word, then decode it. It reverses perfectly
- Hash the same word with MD5. Do it twice, get identical output
- Change one letter, hash again. The output is completely different

### 2. XOR Break

You'll be given a hex string that was XOR'd with a single byte. Your job: find the key.

**Using CyberChef:**
- Input: the hex string
- Recipe: `From Hex` → `XOR Brute Force`
- Scan the output for the key that produces readable text containing `NJACK{...}`

**Or with Python (5 lines):**
```python
cipher = bytes.fromhex("YOUR_HEX_HERE")
for key in range(256):
    result = bytes([b ^ key for b in cipher])
    if b"NJACK" in result:
        print(f"Key: {key:#04x} → {result.decode()}")
```

## Resources

- [CyberChef](https://gchq.github.io/CyberChef/)
- [Cloudflare: What is encryption?](https://www.cloudflare.com/learning/ssl/what-is-encryption/)
- [Cloudflare: What is hashing?](https://www.cloudflare.com/learning/ssl/what-is-hashing/)
- [CryptoHack](https://cryptohack.org/), Intro/General sets for more practice
