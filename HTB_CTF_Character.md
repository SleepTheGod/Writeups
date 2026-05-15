# Character (Hack The Box) - Professional PoC Writeup

## Challenge Profile

| Field | Value |
|---|---|
| Challenge | Character |
| Category | Terminal / Warmup |
| Vulnerability Class | Incremental secret disclosure |
| Transport | Raw TCP |
| Prompt style | One character per requested index |

## Scope And Ethics

This material is for authorized CTF infrastructure only. Do not run this PoC
against systems you do not own or have explicit permission to test.

## Technical Summary

The service asks for a numeric index and returns one flag character at that
position. By repeatedly querying index `0,1,2,...` and concatenating responses,
the full flag can be reconstructed until the server returns `Index out of range`.

## Vulnerable Behavior

1. Service discloses secret data deterministically by index.
2. No meaningful anti-automation controls prevent bulk extraction.
3. Response format is stable: `Character at Index N: X`.
4. Entire secret can be rebuilt with a simple loop.

## Manual Verification Steps

1. Connect to the service:

```bash
nc <HOST> <PORT>
```

2. Send a few indexes manually:

```text
0
1
2
3
```

3. Observe output format:

```text
Character at Index 0: H
Character at Index 1: T
Character at Index 2: B
Character at Index 3: {
```

4. Continue incrementing indexes until:

```text
Index out of range
```

## Automated PoC

Script:
`character_poc.sh`

### Usage

```bash
cd "/home/eliah/Desktop/CTF/HackTheBox/Character"
chmod +x character_poc.sh
./character_poc.sh <host> <port>
```

### Common examples

```bash
./character_poc.sh 154.57.164.76 31059
./character_poc.sh --host 154.57.164.76 --port 31059 --verbose
./character_poc.sh --host 154.57.164.76 --port 31059 --json
```

## Options

- `--host <host>`: target host or IP.
- `--port <port>`: target TCP port.
- `--timeout <seconds>`: socket timeout, default `8`.
- `--max-index <n>`: maximum index attempts, default `500`.
- `--json`: machine-readable JSON output.
- `--verbose`: print debug details.
- `-h`, `--help`: show usage help.

## Exit Codes

- `0`: exploit succeeded and flag extracted.
- `2`: invalid CLI arguments.
- `3`: target connectivity failure.
- `4`: protocol parse failure.
- `5`: extraction completed but no `HTB{...}` pattern detected.

## Why The Exploit Works

- Returning one character at a time still leaks the full secret.
- Index-based access gives direct random access into flag contents.
- Repetition is trivial to automate over a single persistent socket.

## Defensive Guidance

- Do not expose secret bytes/characters by user-controlled index.
- Enforce strict authorization before revealing sensitive values.
- Add rate limits, anomaly detection, and challenge/response controls.
- Prefer challenge designs that never return real secret material incrementally.

## Result Note

Flag values are instance-specific. The format remains `HTB{...}`.

## Final Flag

Target instance: `154.57.164.76:31059`  
Solved on: `2026-04-02`  
Flag: `HTB{tH15_1s_4_r3aLly_l0nG_fL4g_i_h0p3_f0r_y0Ur_s4k3_tH4t_y0U_sCr1pTEd_tH1s_oR_els3_iT_t0oK_qU1t3_l0ng!!}`
