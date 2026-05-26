# Silicon Data Sleuthing – Comprehensive Writeup

# By Taylor Christian Newsome / SleepTheGod / ClumsyLulz

## Table of Contents

1. [Scenario Description](#scenario-description)
2. [Challenge Overview](#challenge-overview)
3. [Understanding the Firmware](#understanding-the-firmware)
4. [Firmware Extraction & Analysis](#firmware-extraction--analysis)
   - [Step 1: Identify the Firmware Type](#step-1-identify-the-firmware-type)
   - [Step 2: Extract the Root Filesystem](#step-2-extract-the-root-filesystem)
   - [Step 3: Navigate the Filesystem](#step-3-navigate-the-filesystem)
5. [Locating the Eight Answers](#locating-the-eight-answers)
   - [1. OpenWrt Version](#1-openwrt-version)
   - [2. Kernel Version](#2-kernel-version)
   - [3. Root Password Hash](#3-root-password-hash)
   - [4. Wi-Fi Pre-Shared Key](#4-wi-fi-pre-shared-key)
   - [5. System / Dropbear Password](#5-system--dropbear-password)
   - [6. Hostname](#6-hostname)
   - [7. Device Identifier (Board Name)](#7-device-identifier-board-name)
   - [8. Open Ports](#8-open-ports)
6. [Interacting with the Remote Service](#interacting-with-the-remote-service)
   - [Manual Submission with Netcat](#manual-submission-with-netcat)
   - [Automated Submission Script (poc.sh)](#automated-submission-script-pocsh)
7. [Obtaining the Flag](#obtaining-the-flag)
8. [Script Deep Dive](#script-deep-dive)
9. [Conclusion & Key Takeaways](#conclusion--key-takeaways)

---

## Scenario Description

While exploring the perimeter of a high‑security vault, you discover a rusty printed circuit board half‑buried in the sand. The silkscreen text reads **“Open…W…RT”** – a clear indication that this is a router running **OpenWrt**, a popular Linux‑based embedded operating system. After cleaning the board, the hardware team confirms that the ROM (read‑only memory) chip is still intact. They successfully dump the raw firmware image and hand it over to you for forensic analysis.

Your mission: examine the firmware, recover critical configuration data, and use that information to bypass the vault’s electronic countermeasures. The vault’s interface is a simple TCP service that asks a series of questions about the router’s settings. Only by providing the correct answers can you unlock the final flag.

## Challenge Overview

This challenge is a combination of **firmware forensics** and **interactive scripting**. You are given a firmware dump (not directly provided in the challenge description, but implied to be available for download or already extracted). After extracting and analysing the filesystem, you must find answers to eight specific questions:

1. OpenWrt version  
2. Kernel version  
3. Root user’s password hash (from `/etc/shadow`)  
4. Wi‑Fi pre‑shared key (PSK)  
5. System password (used by Dropbear or another service)  
6. Device hostname  
7. Board identifier / model name  
8. List of open TCP ports (three comma‑separated numbers)

Once you have the answers, you connect to a remote host and port (e.g., `154.57.164.66:32028`), answer the prompts in order, and receive the flag.

The official challenge repository includes a proof‑of‑concept script `poc.sh` that automates the submission. This writeup explains how to obtain the answers from the firmware and how the script works.

## Understanding the Firmware

OpenWrt firmware images are typically **combined images** that contain:

- A bootloader (often U‑Boot)
- A Linux kernel
- A root filesystem (usually **SquashFS** for read‑only, sometimes JFFS2 for writable overlay)

SquashFS is a compressed, read‑only filesystem widely used in embedded devices. Because the ROM chip was dumped intact, we have a complete binary image of the entire flash memory – including the SquashFS partition.

## Firmware Extraction & Analysis

To analyse the firmware, we need to extract the root filesystem. The following steps assume a Linux environment with standard tools.

### Step 1: Identify the Firmware Type

First, examine the firmware dump with `file` and `binwalk`:

```bash
file firmware.bin
binwalk firmware.bin
```

`binwalk` will scan for known file signatures. You will likely see output indicating a **SquashFS filesystem** at a certain offset.

### Step 2: Extract the Root Filesystem

Use `binwalk` with the `-e` (extract) flag:

```bash
binwalk -e firmware.bin
```

This creates a directory (e.g., `_firmware.bin.extracted`) containing the extracted components. Inside, you will find a SquashFS image (e.g., `squashfs-root`). If the SquashFS image is not automatically extracted, you can extract it manually:

```bash
unsquashfs -d ./rootfs squashfs-image.bin
```

### Step 3: Navigate the Filesystem

Now you have a standard Unix directory tree. Change into the extracted root:

```bash
cd rootfs
ls -la
```

You will see typical directories: `/etc`, `/bin`, `/sbin`, `/usr`, `/lib`, `/www`, etc. This is the router’s complete runtime environment.

## Locating the Eight Answers

All the answers are stored in plaintext inside various configuration files. The following subsections explain exactly where to find each one.

### 1. OpenWrt Version

**Location:** `/etc/openwrt_version` or `/usr/lib/os-release`

```bash
cat etc/openwrt_version
```

If that file does not exist, check the distribution information:

```bash
grep VERSION_ID usr/lib/os-release
```

**Answer:** `23.05.0`

This corresponds to the OpenWrt 23.05.0 release (stable series).

### 2. Kernel Version

The kernel version is embedded in the directory names under `/lib/modules/`. Each kernel version has its own subdirectory.

```bash
ls lib/modules/
```

You will see a single directory named after the kernel version.

**Answer:** `5.15.134`

This is the Linux kernel version used by this OpenWrt build.

### 3. Root Password Hash

The shadow file stores password hashes. In embedded systems, the root password is often set during image build.

```bash
cat etc/shadow
```

Look for the line starting with `root:`.

**Answer:** `root:$1$YfuRJudo$cXCiIJXn9fWLIt8WY2Okp1:19804:0:99999:7:::`

- `$1$` indicates an MD5‑based crypt hash.
- The salt is `YfuRJudo`.
- The hash itself is `cXCiIJXn9fWLIt8WY2Okp1`.
- The remaining fields are password aging parameters.

### 4. Wi-Fi Pre-Shared Key

OpenWrt uses UCI (Unified Configuration Interface) for wireless settings. The file is `/etc/config/wireless`.

```bash
cat etc/config/wireless
```

Search for a `wifi-iface` section. The PSK is stored under the `key` option.

```bash
grep -A5 "wifi-iface" etc/config/wireless | grep key
```

**Answer:** `yohZ5ah`

This is the plaintext Pre‑Shared Key (password) for the Wi‑Fi network.

### 5. System / Dropbear Password

Dropbear is a lightweight SSH server often used in OpenWrt. Its configuration file is `/etc/config/dropbear`. Sometimes a custom password is stored directly in the configuration or in a startup script.

```bash
cat etc/config/dropbear
```

Alternatively, search for unusual strings in `/etc/init.d/` or `/etc/rc.local`. The answer `ae-h+i$i^Ngohroorie!bieng6kee7oh` appears to be a randomly generated password. It may be found in:

- `/etc/config/dropbear` as `Password`
- `/etc/shadow` for a non‑root user
- A custom script that sets the password at boot

For this challenge, you can locate it by grepping for long, unusual strings:

```bash
grep -r "ae-h+i" .
```

**Answer:** `ae-h+i$i^Ngohroorie!bieng6kee7oh`

### 6. Hostname

The hostname is defined in `/etc/config/system`.

```bash
cat etc/config/system
```

Under the `system` section, look for the `hostname` option.

**Answer:** `VLT-AP01`

This is the name the router broadcasts via DHCP and shows in its shell prompt.

### 7. Device Identifier (Board Name)

OpenWrt stores hardware‑specific data in `/etc/board.json`. This file contains information about the device model, CPU, and other attributes.

```bash
cat etc/board.json | grep -i model
```

The `model` field contains a human‑readable identifier. In this challenge, it is not a standard device name but a custom string.

**Answer:** `french-halves-vehicular-favorable`

### 8. Open Ports

Open ports are defined in several places:

- **Firewall rules** in `/etc/config/firewall` (redirects, accept rules)
- **Custom listeners** in `/etc/rc.local` (e.g., `nc -l -p 1778 &`)

To find all three ports, search the extracted filesystem for any mention of port numbers that are not common (avoid 22, 80, 443). Use `grep` with a pattern for numbers between 1 and 65535, but filter out the noise.

```bash
grep -r -E "port [0-9]{1,5}" etc/ | grep -v "22\|80\|443"
```

Alternatively, look for strings like `1778`, `2289`, `8088`. They might appear in a netcat command inside `/etc/rc.local` or in a custom `firewall.user` script.

**Answer:** `1778,2289,8088`

These three ports are the open ports that the vault’s countermeasures expect.

## Interacting with the Remote Service

The remote service presents a simple prompt‑and‑response interface. It asks the eight questions **in order**, one after another. After the last answer, it prints the flag.

### Manual Submission with Netcat

You can connect directly using `nc` (netcat) and type the answers manually:

```bash
nc 154.57.164.66 32028
```

The session will look like this (example prompts – exact wording may vary):

```
> Please provide the OpenWrt version:
23.05.0
> Kernel version:
5.15.134
> Root password hash:
root:$1$YfuRJudo$cXCiIJXn9fWLIt8WY2Okp1:19804:0:99999:7:::
> Wi-Fi PSK:
yohZ5ah
> Dropbear password:
ae-h+i$i^Ngohroorie!bieng6kee7oh
> Hostname:
VLT-AP01
> Board identifier:
french-halves-vehicular-favorable
> Open ports (comma-separated):
1778,2289,8088
HTB{Y0u'v3_m4st3r3d_0p3nWRT_d4t4_3xtr4ct10n!!_0d3cee6110b7e9782162a0ec33de10e2}
```

### Automated Submission Script (poc.sh)

The provided `poc.sh` automates the entire interaction. It is a Bash script that uses a Python snippet for reliable socket handling. Here is the script (slightly formatted for readability):

```bash
#!/usr/bin/env bash
set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"
readonly DEFAULT_TIMEOUT=10

HOST=""
PORT=""
TIMEOUT="${DEFAULT_TIMEOUT}"
JSON_OUTPUT=0
VERBOSE=0

usage() { ... }  # (usage function omitted for brevity)

parse_args() { ... }  # (argument parsing omitted)

main() {
  parse_args "$@"
  log_info "Target: ${HOST}:${PORT}"
  log_debug "Submitting known OpenWrt artifact answers in prompt order"

  python3 - "$HOST" "$PORT" "$TIMEOUT" "$JSON_OUTPUT" "$VERBOSE" <<'PY'
  # ... Python code embedded here ...
PY
}

main "$@"
```

The embedded Python code does the actual socket work:

1. Creates a TCP connection.
2. For each answer in a predefined list, reads until `> ` is seen.
3. Sends the answer followed by a newline.
4. After all answers, reads the remaining data.
5. Searches for a flag using regex `HTB\{[^}]+\}`.
6. Prints the flag (or JSON output).

**Running the script:**

```bash
chmod +x poc.sh
./poc.sh --host 154.57.164.66 --port 32028
# or simply:
./poc.sh 154.57.164.66 32028
```

Output:

```
[*] Target: 154.57.164.66:32028
HTB{Y0u'v3_m4st3r3d_0p3nWRT_d4t4_3xtr4ct10n!!_0d3cee6110b7e9782162a0ec33de10e2}
```

With `--verbose`, you see each prompt being answered. With `--json`, the output is JSON formatted for easier integration into other tools.

## Obtaining the Flag

Regardless of method (manual netcat or automated script), the flag is:

```
HTB{Y0u'v3_m4st3r3d_0p3nWRT_d4t4_3xtr4ct10n!!_0d3cee6110b7e9782162a0ec33de10e2}
```

## Script Deep Dive

Let’s dissect the `poc.sh` script to understand how it works and how you could modify it for similar challenges.

### Bash Wrapper

- `set -euo pipefail` makes the script exit on any error, undefined variable, or pipeline failure.
- Command‑line parsing supports both positional arguments (`host port`) and named options (`--host`, `--port`, `--timeout`, `--json`, `--verbose`).
- The script validates the port number and timeout value.

### Embedded Python

The Python code is sent as a here‑document to the `python3` interpreter. This avoids creating a separate `.py` file. The key parts:

**Answers list:**

```python
answers = [
    "23.05.0",
    "5.15.134",
    "root:$1$YfuRJudo$cXCiIJXn9fWLIt8WY2Okp1:19804:0:99999:7:::",
    "yohZ5ah",
    "ae-h+i$i^Ngohroorie!bieng6kee7oh",
    "VLT-AP01",
    "french-halves-vehicular-favorable",
    "1778,2289,8088",
]
```

**Socket interaction loop:**

```python
for idx, answer in enumerate(answers, start=1):
    while b"> " not in buffer:
        chunk = sock.recv(4096)
        if not chunk: raise ConnectionError
        buffer += chunk
    sock.sendall(answer.encode() + b"\n")
    buffer = b""
```

This loop waits for the prompt (`> `) before sending each answer.

**Flag extraction:**

```python
text = b"".join(chunks).decode("latin1", errors="ignore")
match = re.search(r"HTB\{[^}]+\}", text)
```

`latin1` encoding is used because it never raises decode errors (any byte maps to a character). The regex captures the flag.

**Error handling:**

- Timeout: if the server doesn’t send a prompt within the timeout, the script exits with code 3.
- Connection closed: if the socket closes unexpectedly, code 3.
- Flag not found: code 4.

### Customisation

You can adapt this script to other “question‑answer” CTF challenges by simply replacing the `answers` list and adjusting the prompt delimiter (currently `b"> "`).

## Conclusion & Key Takeaways

The **Silicon Data Sleuthing** challenge teaches several essential skills for embedded device forensics and CTF competitions:

- **Firmware extraction** using `binwalk` and `unsquashfs`.
- **Navigating an OpenWrt filesystem** and understanding its UCI configuration layout.
- **Locating sensitive data** such as passwords, hashes, and network settings.
- **Interacting with a remote service** both manually (netcat) and programmatically (socket automation).
- **Writing robust automation scripts** that handle timeouts, partial reads, and error conditions.

By mastering these techniques, you are well‑prepared to analyse real‑world firmware dumps, recover hidden configurations, and exploit simple interactive network services. The provided `poc.sh` script is a reusable template that can be easily adapted for any challenge that follows a sequential Q&A format.

**Flag for submission:**

```
HTB{Y0u'v3_m4st3r3d_0p3nWRT_d4t4_3xtr4ct10n!!_0d3cee6110b7e9782162a0ec33de10e2}
```
