# Day 6: A Peek at Recon & Enumeration

## What You'll Learn Today

Why professionals look before they touch. Recon comes before any testing. You'll run your first port scan with Nmap and do passive recon with crt.sh.

## Core Concepts

### Why Recon First

Before testing anything, you need to know what's there. What services are running? What ports are open? What's the attack surface? Recon answers these questions without touching the target aggressively.

### Port Scanning with Nmap

A **port scan** checks which ports on a machine are open and what services are listening. An open port means a service is running and accepting connections.

### Common Ports

| Port | Service | What it usually means |
|---|---|---|
| 22 | SSH | Remote shell access |
| 80 | HTTP | Web server (unencrypted) |
| 443 | HTTPS | Web server (encrypted) |
| 21 | FTP | File transfer |
| 8080 | HTTP alt | Often a dev/proxy server |

### Passive Recon with crt.sh

[crt.sh](https://crt.sh/) is a public log of issued TLS certificates. You can look up any domain and see what certificates have been issued for it, revealing subdomains, infrastructure, and history. This is **passive** recon: you're reading public records, not touching the target.

## Hands-On

### 1. Nmap Scan

Run one scan against the provided practice VM only:
```bash
nmap -sV <practice-VM-IP>
```

Read through the output line by line. For each open port:
- What service is running?
- What version did Nmap detect?
- What do you think it's used for?

### 2. Certificate Lookup

Go to [crt.sh](https://crt.sh/) and look up a domain you own (or one provided for the course).

The results table shows every TLS certificate ever issued for that domain. The "Common Name" and "Matching Identities" columns are where subdomains show up. Look for entries like `staging.example.com`, `api.example.com`, or anything you didn't know existed. Each one is a piece of infrastructure someone set up and possibly forgot about.

**Important:** Only look up domains you own or that are provided for the course.

## Resources

- [Nmap Getting Started Guide](https://nmap.org/book/man.html)
- [crt.sh](https://crt.sh/)
