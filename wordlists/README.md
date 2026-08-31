# Wordlists

`starter-wordlist.txt` is a short list of common passwords, enough for the JWT-cracking hard challenge (Track A, A2). Point a cracking tool at it:

```bash
# example: crack a weak JWT secret
jwt_tool <token> -C -d wordlists/starter-wordlist.txt
```

For anything real, use rockyou.txt. It is not kept here because it is about 133 MB, over GitHub's file-size limit. Get it from:

- Kali: `/usr/share/wordlists/rockyou.txt.gz`, then `gunzip rockyou.txt.gz`
- [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Leaked-Databases/rockyou.txt.tar.gz)
