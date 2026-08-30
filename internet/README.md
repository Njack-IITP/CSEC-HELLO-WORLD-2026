# Internet & Networking Track

## What You'll Learn

By the end of this track, you'll be able to:

- Explain how a webpage loads, and read requests, headers, and cookies in DevTools
- Recognize common web vulnerabilities (XSS, SQL injection, broken access control) when you see them
- Break a weak cipher: understand encoding vs. hashing vs. encryption, and brute-force a single-byte XOR
- Do basic forensics: open a packet capture in Wireshark and extract cleartext credentials
- Understand core networking: DNS, TCP/UDP, firewalls, VPNs, proxies
- Run basic recon: port scanning with Nmap, passive certificate lookups

## Schedule

| Day | Theme |
|---|---|
| [1](DAY1.md) | How the Web Actually Works |
| [2](DAY2.md) | Web App Security Basics |
| [3](DAY3.md) | Cryptography & the XOR Atom |
| [4](DAY4.md) | Taste: Forensics |
| [5](DAY5.md) | Networking & DNS Fundamentals |
| [6](DAY6.md) | A Peek at Recon & Enumeration |
| [7](DAY7.md) | Capstone, putting it all together |

Content is released one day at a time.

## Setup

Install these **before Day 1**:

- VirtualBox or VMware with a [Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines) or Ubuntu VM (2+ CPU cores, 4GB RAM, 25GB disk)
- Wireshark (needed from Day 4, worth installing now): `sudo apt install wireshark -y`
- Bookmark [CyberChef](https://gchq.github.io/CyberChef/), no install needed
- A terminal with `curl`, `dig`, `ping` available (Kali has all of these)
- Docker (optional, for running practice apps locally): [install guide](https://docs.docker.com/engine/install/)

If using Kali, update after first boot:
```bash
sudo apt update && sudo apt full-upgrade -y
```

### On macOS?

Most of this track works natively on your Mac without a VM:

- **Wireshark**: download from [wireshark.org](https://www.wireshark.org/download.html) or `brew install --cask wireshark`
- **Nmap**: `brew install nmap`
- **curl, dig, ping**: already installed on macOS
- **Docker**: [Docker Desktop for Mac](https://docs.docker.com/desktop/install/mac-install/) (for running Juice Shop locally)
- **CyberChef**: browser-based, works everywhere
- **Python 3**: `brew install python3` if not already installed

No VM needed for this track.

## Resources

These are good companions throughout the week:

- [PortSwigger Web Security Academy](https://portswigger.net/web-security): free beginner labs
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/): short, clear explainers
- [CryptoHack](https://cryptohack.org/): crypto practice
