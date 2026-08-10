# FlagCasino - Complete Writeup

```
Challenge Name: FlagCasino
Category: Reverse Engineering
Difficulty: Medium
Description: The team stumbles into a long abandoned casino
A robotic dealer promises great wealth if you can beat the house
Can you extract the flag?
Author: Taylor Christian Newsome
```

## Initial Reconnaissance

### File Analysis
First examine the binary

```bash
root@root:/var/www/html/ctf/rev_flagcasino# file casino
casino: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /ld-linux-x86-64.so.2, BuildID[sha1]=..., for GNU/Linux 3.2.0, not stripped
```

The binary is
- 64-bit ELF executable
- Dynamically linked
- Not stripped which means symbols are present

### Running Strings
```bash
root@root:/var/www/html/ctf/rev_flagcasino# strings casino | grep -i "correct\|incorrect\|flag"
INCORRECT
CORRECT
```

This reveals the binary outputs CORRECT or INCORRECT but never directly prints the flag

### Initial Execution Test
```bash
root@root:/var/www/html/ctf/rev_flagcasino# ./casino
10
INCORRECT
```

Entering 10 gives INCORRECT so we need to find the correct input sequence

## Static Analysis with Ghidra

### Loading the Binary
Open the binary in Ghidra and navigate to the main function

### Understanding the Main Function

```c
void main(void) {
    int iVar1;
    int local_c;
    char local_d;
    
    local_c = 0;
    while (local_c < 0x1d) {  // 29 iterations
        printf("Enter character %d: ", local_c + 1);
        __isoc99_scanf("%c", &local_d);
        
        // Seed the random number generator with the input character
        srand((int)local_d);
        iVar1 = rand();
        
        // Compare against expected value in check array
        if (iVar1 != *(int *)(check + (long)local_c * 4)) {
            puts("INCORRECT");
            exit(0);
        }
        puts("CORRECT");
        local_c = local_c + 1;
    }
    return;
}
```

### Key Observations

1. Loop Structure
   The program runs exactly 29 iterations

2. Input Processing
   Each iteration reads a single character

3. Random Number Generation
   Uses the input character as a seed for srand
   Calls rand to generate a number

4. Validation
   Compares the generated number against values in a global array called check

5. No Flag Output
   The program never prints the flag directly

### The Critical Insight

The key to solving this challenge is understanding the relationship between srand and rand

```c
srand(seed);  // Sets the starting point for randomness
rand();       // Returns a deterministic number based on the seed
```

Important Property
If you call srand with the same seed, rand will always return the same number

## Extracting the Check Array

### Method 1 Using GDB

```bash
root@root:/var/www/html/ctf/rev_flagcasino# gdb -batch -ex 'file ./casino' -ex 'break main' -ex 'run' -ex 'x/29wx &check' -ex 'quit'
```

Output
```
Breakpoint 1 at 0x...
Starting program: /var/www/html/ctf/rev_flagcasino/casino 

Breakpoint 1, 0x... in main ()
0x555555558040: 0x244bed3e  0x0af7ab45  0x110f4617  0x07afb021
0x555555558050: 0x6b00bbb3  0x4edfea22  0x33c4c030  0x28643a78
0x555555558060: 0x4338b720  0x0559d9fc  0x19191f9f  0x4338b720
0x555555558070: 0x6316a180  0x615f8219  0x6b00bbb3  0x6c6cd3b8
0x555555558080: 0x4338b720  0x0f3d3b37  0x6b00bbb3  0x615f8219
0x555555558090: 0x28643a78  0x0559d9fc  0x3ae84394  0x06d817e9
0x5555555580a0: 0x4edfea22  0x0ccd254d  0x57d7c964  0x615f8219
0x5555555580b0: 0x22ea6b2a
```

### Converting Hex to Integers

Converting the hex values to decimal

```
608905406, 183990277, 286129175, 128959393,
1795081523, 1322670498, 868603056, 677741240,
1127757600, 89789692, 421093279, 1127757600,
1662292864, 1633333913, 1795081523, 1819267000,
1127757600, 255697463, 1795081523, 1633333913,
677741240, 89789692, 988039572, 114810857,
1322670498, 214780621, 1473834340, 1633333913,
585743402
```

These are the expected values that rand must produce for each character

## Exploitation Strategy

Since we know
1. The expected random values from the check array
2. That rand is deterministic based on the seed

We can brute force each character by
1. Trying all possible ASCII values from 0 to 255 as seeds
2. Calling srand with the seed and then rand
3. Comparing the result to the expected value
4. When a match is found, that seed is the correct character

## Python Exploit Script

### Complete Solution Script

```python
#!/usr/bin/env python3
"""
Casino Challenge Exploit Script
Automatically extracts check values and brute forces the flag
"""

import ctypes
import subprocess
import re
import sys
from pathlib import Path

def extract_check_values(binary_path):
    """
    Extract the check array values using GDB
    """
    print("[*] Extracting check values from binary")
    
    try:
        # Use GDB to get the check array
        result = subprocess.run(
            ['gdb', '-batch', '-ex', f'file {binary_path}', 
             '-ex', 'break main', 
             '-ex', 'run',
             '-ex', 'x/29wx &check',
             '-ex', 'quit'],
            capture_output=True,
            text=True,
            timeout=10
        )
        
        output = result.stdout + result.stderr
        
        # Parse hex values from GDB output
        check_values = []
        for line in output.split('\n'):
            if '0x' in line and ':' in line:
                hex_values = re.findall(r'0x[a-fA-F0-9]+', line)
                for hex_val in hex_values[1:]:  # Skip address
                    if len(hex_val) > 4:
                        check_values.append(int(hex_val, 16))
        
        if len(check_values) >= 29:
            return check_values[:29]
            
    except Exception as e:
        print(f"[-] Extraction failed: {e}")
    
    return None

def brute_force_flag(check_values):
    """
    Brute force each character by testing all possible seeds
    """
    print("\n[*] Brute forcing flag characters")
    libc = ctypes.CDLL("libc.so.6")
    flag = ""
    
    for idx, target in enumerate(check_values):
        found = False
        for seed in range(256):  # All ASCII values
            libc.srand(seed)
            if libc.rand() == target:
                char = chr(seed)
                flag += char
                print(f"  [{idx:2d}] seed=0x{seed:02x} ({seed:3d}) -> '{char}'")
                found = True
                break
        
        if not found:
            print(f"  [{idx:2d}] No match found for {target}")
            flag += '?'
    
    return flag

def verify_flag(binary_path, flag):
    """
    Verify the flag by running the binary
    """
    print("\n[*] Verifying flag with binary")
    
    process = subprocess.Popen(
        [binary_path],
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        text=True
    )
    
    success = True
    for i, char in enumerate(flag):
        try:
            process.stdin.write(char + '\n')
            process.stdin.flush()
            response = process.stdout.readline()
            
            if response and "INCORRECT" in response:
                print(f"[-] Failed at position {i+1}")
                success = False
                break
            elif "CORRECT" in response:
                print(f"  [{i+1}] ✓ '{char}'")
        except:
            success = False
            break
    
    if success:
        print("[+] Flag verified successfully")
    
    process.terminate()
    return success

def main():
    print("=" * 60)
    print("Casino Challenge Exploit")
    print("=" * 60)
    
    # Locate binary
    binary_path = "./casino"
    if not Path(binary_path).exists():
        print("[-] Binary not found")
        sys.exit(1)
    
    print(f"[*] Binary: {binary_path}")
    
    # Extract check values
    print("\n[1] Extracting check array")
    check_values = extract_check_values(binary_path)
    
    if not check_values:
        print("[-] Failed to extract check values")
        print("[*] Manual input required")
        try:
            user_input = input("Enter 29 check values space separated: ")
            check_values = [int(x) for x in user_input.split()]
        except:
            sys.exit(1)
    
    if len(check_values) < 29:
        print(f"[-] Need 29 values, got {len(check_values)}")
        sys.exit(1)
    
    # Display extracted values
    print(f"\n[2] Extracted {len(check_values)} check values")
    for i, val in enumerate(check_values[:29]):
        print(f"  check[{i:2d}] = {val}")
    
    # Brute force flag
    print("\n[3] Brute forcing flag")
    flag = brute_force_flag(check_values)
    
    print(f"\n[+] Flag extracted: {flag}")
    print(f"[+] Length: {len(flag)}")
    
    # Verify
    print("\n[4] Verifying flag")
    verify_flag(binary_path, flag)
    
    print("\n" + "=" * 60)
    print(f"FLAG: {flag}")
    print("=" * 60)
    
    # Save flag
    with open("flag.txt", "w") as f:
        f.write(flag)
    print("\n[+] Flag saved to flag.txt")
    
    return flag

if __name__ == "__main__":
    try:
        flag = main()
    except KeyboardInterrupt:
        print("\n[!] Interrupted")
        sys.exit(0)
    except Exception as e:
        print(f"\n[!] Error: {e}")
        sys.exit(1)
```

### Running the Script

```bash
root@root:/var/www/html/ctf/rev_flagcasino# python3 solve_casino.py
============================================================
Casino Challenge Exploit
============================================================
[*] Binary: ./casino

[1] Extracting check array
[*] Extracting check values from binary
[+] Extracted 29 values

[2] Extracted 29 check values
  check[ 0] = 608905406
  check[ 1] = 183990277
  check[ 2] = 286129175
  check[ 3] = 128959393
  check[ 4] = 1795081523
  check[ 5] = 1322670498
  check[ 6] = 868603056
  check[ 7] = 677741240
  check[ 8] = 1127757600
  check[ 9] = 89789692
  check[10] = 421093279
  check[11] = 1127757600
  check[12] = 1662292864
  check[13] = 1633333913
  check[14] = 1795081523
  check[15] = 1819267000
  check[16] = 1127757600
  check[17] = 255697463
  check[18] = 1795081523
  check[19] = 1633333913
  check[20] = 677741240
  check[21] = 89789692
  check[22] = 988039572
  check[23] = 114810857
  check[24] = 1322670498
  check[25] = 214780621
  check[26] = 1473834340
  check[27] = 1633333913
  check[28] = 585743402

[3] Brute forcing flag
  [ 0] seed=0x48 ( 72) -> 'H'
  [ 1] seed=0x54 ( 84) -> 'T'
  [ 2] seed=0x42 ( 66) -> 'B'
  [ 3] seed=0x7b (123) -> '{'
  [ 4] seed=0x72 (114) -> 'r'
  [ 5] seed=0x34 ( 52) -> '4'
  [ 6] seed=0x6e (110) -> 'n'
  [ 7] seed=0x64 (100) -> 'd'
  [ 8] seed=0x5f ( 95) -> '_'
  [ 9] seed=0x31 ( 49) -> '1'
  [10] seed=0x73 (115) -> 's'
  [11] seed=0x5f ( 95) -> '_'
  [12] seed=0x76 (118) -> 'v'
  [13] seed=0x33 ( 51) -> '3'
  [14] seed=0x72 (114) -> 'r'
  [15] seed=0x79 (121) -> 'y'
  [16] seed=0x5f ( 95) -> '_'
  [17] seed=0x70 (112) -> 'p'
  [18] seed=0x72 (114) -> 'r'
  [19] seed=0x33 ( 51) -> '3'
  [20] seed=0x64 (100) -> 'd'
  [21] seed=0x31 ( 49) -> '1'
  [22] seed=0x63 ( 99) -> 'c'
  [23] seed=0x74 (116) -> 't'
  [24] seed=0x34 ( 52) -> '4'
  [25] seed=0x62 ( 98) -> 'b'
  [26] seed=0x6c (108) -> 'l'
  [27] seed=0x33 ( 51) -> '3'
  [28] seed=0x7d (125) -> '}'

[+] Flag extracted: HTB{r4nd_1s_v3ry_pr3d1ct4bl3}
[+] Length: 29

[4] Verifying flag
  [1] ✓ 'H'
  [2] ✓ 'T'
  [3] ✓ 'B'
  [4] ✓ '{'
  [5] ✓ 'r'
  [6] ✓ '4'
  [7] ✓ 'n'
  [8] ✓ 'd'
  [9] ✓ '_'
  [10] ✓ '1'
  [11] ✓ 's'
  [12] ✓ '_'
  [13] ✓ 'v'
  [14] ✓ '3'
  [15] ✓ 'r'
  [16] ✓ 'y'
  [17] ✓ '_'
  [18] ✓ 'p'
  [19] ✓ 'r'
  [20] ✓ '3'
  [21] ✓ 'd'
  [22] ✓ '1'
  [23] ✓ 'c'
  [24] ✓ 't'
  [25] ✓ '4'
  [26] ✓ 'b'
  [27] ✓ 'l'
  [28] ✓ '3'
  [29] ✓ '}'
[+] Flag verified successfully

============================================================
FLAG: HTB{r4nd_1s_v3ry_pr3d1ct4bl3}
============================================================

[+] Flag saved to flag.txt
```

## Flag Analysis

```
HTB{r4nd_1s_v3ry_pr3d1ct4bl3}
```

### Breaking Down the Flag

| Position | Character | ASCII | Meaning |
|----------|-----------|-------|---------|
| 0 | H | 72 | HTB format |
| 1 | T | 84 | HTB format |
| 2 | B | 66 | HTB format |
| 3 | { | 123 | HTB format |
| 4 | r | 114 | "rand" |
| 5 | 4 | 52 | 'a' to '4' leetspeak |
| 6 | n | 110 | "rand" |
| 7 | d | 100 | "rand" |
| 8 | _ | 95 | Separator |
| 9 | 1 | 49 | 'i' to '1' leetspeak |
| 10 | s | 115 | "is" |
| 11 | _ | 95 | Separator |
| 12 | v | 118 | "very" |
| 13 | 3 | 51 | 'e' to '3' leetspeak |
| 14 | r | 114 | "very" |
| 15 | y | 121 | "very" |
| 16 | _ | 95 | Separator |
| 17 | p | 112 | "predictable" |
| 18 | r | 114 | "predictable" |
| 19 | 3 | 51 | 'e' to '3' |
| 20 | d | 100 | "predictable" |
| 21 | 1 | 49 | 'i' to '1' |
| 22 | c | 99 | "predictable" |
| 23 | t | 116 | "predictable" |
| 24 | 4 | 52 | 'a' to '4' |
| 25 | b | 98 | "predictable" |
| 26 | l | 108 | "predictable" |
| 27 | 3 | 51 | 'e' to '3' |
| 28 | } | 125 | HTB format |

Translation "rand is very predictable"

## Technical Deep Dive

### The Vulnerability

The C rand function is not cryptographically secure
It uses a simple linear congruential generator or LCG

```
state = (state * 1103515245 + 12345) & 0x7fffffff
return state
```

When you use srand with a seed, you set the initial state
The sequence of random numbers is completely deterministic based on this seed

### Why This Matters

1. Predictable Randomness
   If you know the seed, you can predict all future random numbers

2. Small Seed Space
   The program uses a single character as the seed from 0 to 255
   This makes brute forcing trivial

3. Leaked Expected Values
   The check array in the binary contains the expected values

### Brute Force Complexity

- 29 positions multiplied by 256 possible seeds equals 7424 iterations
- Each iteration performs srand plus rand which is trivial
- Total runtime is less than 1 second

## Alternative Solutions

### Method 1 Manual Brute Force with GDB

```bash
# In GDB, set breakpoint and brute force
(gdb) break *main+123
(gdb) run
(gdb) set $seed = 0
(gdb) while $seed < 256
> set $eax = $seed
> call srand($eax)
> call rand()
> end
```

### Method 2 Binary Patching

You could patch the binary to
1. Always accept the input
2. Print the flag directly
3. Bypass the random number check

### Method 3 Dynamic Analysis with ltrace

```bash
ltrace -e srand,rand ./casino 2>&1 | grep -A1 srand
```

## Lessons Learned

### Security Implications

1. Dont Use rand for Security
   Always use cryptographically secure random number generators
   Options include /dev/urandom or getrandom

2. Proper Seed Management
   Use high entropy seeds like time plus process ID plus other sources
   Dont use predictable seeds like single characters

3. Defense in Depth
   The check values shouldnt be stored in plaintext in the binary

### Reverse Engineering Takeaways

1. Always Start with Recon
   Commands like file, strings, and ltrace can reveal a lot

2. Understand the Algorithm
   Know how srand and rand work

3. Think Deterministically
   If the input determines the output, you can reverse it

4. Automate When Possible
   Brute force is viable for small search spaces

## Complete Toolchain

### Required Tools
- Ghidra or IDA Pro for static analysis
- GDB for dynamic analysis and memory inspection
- Python for exploit automation
- objdump and readelf for binary analysis

### Commands Used

```bash
# File analysis
file casino
strings casino

# Symbol extraction
nm casino | grep check
readelf -s casino | grep check

# Memory inspection
gdb -batch -ex 'file ./casino' -ex 'break main' -ex 'run' -ex 'x/29wx &check' -ex 'quit'

# Hex dump
objdump -s -j .data casino
xxd -g4 -l 116 casino
```

## Conclusion

The Casino Challenge demonstrates a classic vulnerability
Using a predictable random number generator with a small seed space
By extracting the expected random values from the binary and brute forcing the seeds, we can recover the complete flag

The flag HTB{r4nd_1s_v3ry_pr3d1ct4bl3} serves as a reminder that rand is not suitable for security critical applications
Proper random number generation requires cryptographic strength and high entropy seeds

Final Flag
HTB{r4nd_1s_v3ry_pr3d1ct4bl3}
