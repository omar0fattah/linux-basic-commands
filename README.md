# Cybersecurity Learning Notes

Personal notes and beginner-friendly guide as I work through cybersecurity fundamentals — from networking basics all the way to actual exploitation and privilege escalation. Written so I can reference specific details quickly, and structured so anyone starting from zero could follow along too.

Each file below covers one stage of the roadmap. Quizzes are included at the end of each stage as self-checks — cover the answer, try to answer yourself first, then check.

## 📂 Stages

- **[Stage 1 — Networking & Linux Fundamentals](./stage1-networking-linux.md)**
  OSI/TCP-IP model, IP addressing & subnetting, TCP vs UDP, the 3-way handshake, common ports, DNS, Linux file system & permissions, essential commands, basic bash scripting, and netcat/reverse shell fundamentals.

- **[Stage 2 — Web Fundamentals, Vulnerabilities & Enumeration](./stage2-web-fundamentals.md)**
  How HTTP/cookies/sessions work, the three core vuln classes (SQL injection, XSS, command injection), enumeration with nmap/gobuster, privilege escalation, two full worked attack-chain walkthroughs, and a legal/authorization primer.

*(More stages will be added as I progress — specialization tracks, certifications, etc.)*

## 🗺️ The Bigger Roadmap

```
1. Foundations           → networking, Linux, scripting          ✅
2. Core security concepts → web fundamentals, vuln classes, enum ✅
3. Offensive practice     → TryHackMe / HackTheBox boxes          🔄 in progress
4. Specialization         → pentest / OSINT / blue team / AppSec  ⏳
5. Certifications         → Security+ → eJPT → OSCP                ⏳
```

## ⚖️ A Note on Ethics

Everything here is for learning purposes on authorized targets only — TryHackMe, HackTheBox, or systems I own outright. If you're using this as a reference too: never run any of this against a system without explicit, documented permission from its actual owner. No exceptions.

## 🛠️ Setup

Practicing primarily on Kali Linux (attacker side) with a Windows VM (target side) for cross-platform practice. Hands-on lab work paused temporarily due to limited device access — theory-first learning in the meantime.

---

*Started: [11 aug 2026] — updated as I go.*
