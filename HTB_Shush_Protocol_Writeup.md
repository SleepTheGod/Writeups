# HTB Challenge: Shush Protocol - Writeup

## Scenario
PCAP from an ICS environment at a fertilizer plant. Need to find the password used by the control device to connect to the PLC.

## Solution

### Step 1: Download and Extract
```bash
curl -L "http://ctf.hackthebox.com/challenges/40745/download?expires=1784227286&signature=a8147eb064175966a1fb9dd16efa0dce1dc42264e2817b72131235aa87aa40c7" -o htb.zip
unzip htb.zip
cd ics_shush_protocol
```

### Step 2: Extract Flag from PCAP
Since `tshark` wasn't available, used `strings` to extract readable text:

```bash
strings traffic.pcapng | grep -E "HTB\{"
```

**Output:**
```
.HTB{50m371m35_cu570m_p2070c01_423_n07_3n0u9h7}
```

## Flag
```
HTB{50m371m35_cu570m_p2070c01_423_n07_3n0u9h7}
```

## Decoded
```
HTB{sometimes_custom_protocol_is_not_enough}
```

## Why This Worked
The flag was stored in plaintext within the network capture. Even without packet analysis tools, basic Linux utilities revealed the hidden data.
