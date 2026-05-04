# An Unusual Sighting – Automated Solver

**Category** Forensics / Scripting  
**Platform** Hack The Box  
**Scenario** After manually extracting the six answers from the SSH logs and bash history, a bash script with an embedded Python solver automates the interaction with the remote service and retrieves the flag.

---

## Overview

The challenge provides a remote service on `154.57.164.68:31649` that asks six questions about an intrusion. Answering all correctly returns the flag. The forensic analysis (detailed in a separate walkthrough) yields these answers

1. `100.107.36.130:2221`
2. `2024-02-13 11:29:50`
3. `2024-02-19 04:00:14`
4. `OPkBSs6okUKraq8pYo4XwwBg55QSo210F09FCe1-yj4`
5. `whoami`
6. `./setup`

Instead of answering manually, a combined bash/Python script connects to the service, sends the answers sequentially, and prints the flag.

---

## Script Breakdown

### Bash Wrapper

The outer script is a standard bash script with strict error handling (`set -euo pipefail`). It accepts the target host and port as arguments (defaulting to `154.57.164.68` and `31649`). First, it performs a preflight check using `nc -z` to verify connectivity, then launches an inline Python script using a heredoc (`<<'PY' ... PY`), passing the host and port as arguments.

```bash
#!/usr/bin/env bash
set -euo pipefail

host="${1:-154.57.164.68}"
port="${2:-31649}"

echo "[*] Target: $host:$port"
echo "[*] Preflight check with netcat..."

if ! nc -z -w 3 "$host" "$port"; then
    echo "[-] Connection refused or port closed."
    exit 1
fi

echo "[+] Netcat check passed. Service is reachable."
echo "[*] Launching solver..."

python3 - "$host" "$port" <<'PY'
...
PY
```

### Python Solver

The Python code performs the actual interaction. Key components

**Answer list** Pre‑determined answers from the forensic investigation.

```python
answers = [
    "100.107.36.130:2221",
    "2024-02-13 11:29:50",
    "2024-02-19 04:00:14",
    "OPkBSs6okUKraq8pYo4XwwBg55QSo210F09FCe1-yj4",
    "whoami",
    "./setup",
]
```

**`recv()` helper** Reads data from the socket until the prompt character `>` appears or a timeout occurs. This waits for the question prompt before sending the answer.

```python
def recv(sock):
    data = b""
    sock.settimeout(5)
    try:
        while True:
            chunk = sock.recv(4096)
            if not chunk:
                break
            data += chunk
            if b">" in data:
                break
    except socket.timeout:
        pass
    return data
```

**Main socket logic** Creates a TCP connection, then loops over the answers. For each answer it first calls `recv()` to consume the prompt, then sends the answer followed by a newline.

```python
with socket.create_connection((host, port), timeout=5) as sock:
    for ans in answers:
        recv(sock)
        sock.sendall(ans.encode() + b"\n")
```

**Flag extraction** After sending all answers, the script reads any remaining data until the socket closes or times out. It decodes the output and searches for the `HTB{` marker, then extracts all characters up to the closing `}`.

```python
out = b""
sock.settimeout(3)
while True:
    try:
        chunk = sock.recv(4096)
        if not chunk:
            break
        out += chunk
    except socket.timeout:
        break

text = out.decode(errors="ignore")

if "HTB{" not in text:
    print("[-] Flag not found.")
    sys.exit(1)

start = text.index("HTB{")
end = text.index("}", start)
print(text[start:end+1])
```

If the flag is missing the script exits with an error message; otherwise it prints the flag string.

---

## Execution

Running the script produces the following output

```
[*] Target: 154.57.164.68:31649
[*] Preflight check with netcat...
[+] Netcat check passed. Service is reachable.
[*] Launching solver...
HTB{4n_unusual_s1ght1ng_1n_SSH_l0gs!}
```

---

## Flag

`HTB{4n_unusual_s1ght1ng_1n_SSH_l0gs!}`
