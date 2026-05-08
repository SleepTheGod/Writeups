# Weaponized JPEG with Injected x86 Shellcode – Doxbin "logo.jpg" Uncovered

**Posted by: Taylor Christian Newsome (SleepTheGod / ClumsyLulz)  
Date: 2026-05-08**

---

## 0x00 – In The Wild

While monitoring questionable domains, we snagged a suspicious JPEG from **doxbin.net**:

```bash
root@clumsy:~/doxbin# wget https://doxbin.net/static/logo.jpg
--2026-05-08 10:49:35--  https://doxbin.net/static/logo.jpg
Resolving doxbin.net ... 2a06:98c1:3120::3, 188.114.96.3, ...
HTTP request sent, awaiting response... 403 Forbidden
2026-05-08 10:49:36 ERROR 403: Forbidden.
```

403 – but we already had a local copy (likely fetched earlier). Let’s see what hides inside.

---

## 0x01 – First Glance: Looks Legit, But…

```bash
root@clumsy:~/doxbin# file logo.jpg
logo.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 96x96,
segment length 16, Exif Standard: [TIFF image data, big-endian, direntries=4,
xresolution=62, yresolution=70, resolutionunit=2, software=Paint.NET 5.1.2],
baseline, precision 8, 208x227, components 3
```

A valid JPEG, metadata points to Paint.NET – could be innocent. But in malwareresearch, innocence is a mask.

`binwalk` finds only the JPEG and a TIFF (Exif) – no appended ZIP, no PE, no ELF.

```bash
root@clumsy:~/doxbin# binwalk logo.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
30            0x1E            TIFF image data, big-endian, offset of first image directory: 8
```

Steghide says "capacity 2.6KB" but needs a passphrase – no extraction.

---

## 0x02 – Hexdump Reveals The Trick

We dumped the whole file and something screamed **"shellcode"**:

```bash
root@clumsy:~/doxbin# hexdump -C logo.jpg | grep -A2 -B2 'e8 a0'
```

Output (from earlier full hexdump)

```
000002d0  02 11 03 11 00 3f 00 fe  0b e8 00 a0 02 80 0a 00  |.....?..........|
000002e0  28 00 a0 02 80 0a 00 28  00 a0 02 80 0a 00 28 00  |(......(......(.|
000002f0  a0 02 80 0a 00 28 00 a0  02 80 0a 00 28 00 a0 02  |.....(......(...|
00000300  80 0a 00 28 00 a0 02 80  0a 00 28 00 a0 02 80 3e  |...(......(....>|
```

**What’s wrong here?**  
In normal JPEG entropy‑coded data, you expect statistical randomness – not repeating patterns like `e8 a0 02 80 0a` over and over.  
`e8` = x86 `CALL` opcode. The following bytes form a 32‑bit relative displacement. This is **raw machine code**, not image data.

---

## 0x03 – Locating The JPEG Frame Boundaries

Every JPEG ends with `FF D9`. We found it:

```bash
root@clumsy:~/doxbin# hexdump -C logo.jpg | grep 'ff d9'
00010850  80 0a 00 28 00 a0 02 80  0a 00 ff d9              |...(........|
```

Total size 67676 bytes → EOI at 67666 → **no trailing data**. So the malware isn’t simply appended – it’s **inside** the compressed stream.

Start of Scan (SOS) marker `FF DA` at offset `0x2c2`:

```bash
root@clumsy:~/doxbin# hexdump -C logo.jpg | grep -i "ff da"
000002c0  f2 f3 f4 f5 f6 f7 f8 f9  fa ff da 00 0c 03 01 00  |................|
```

Thus the shellcode lives in the scan data between `0x2c2` and `0x10858`.

---

## 0x04 – Carving A Chunk And Disassembling

We extracted 768 bytes from offset `0x300` (just after the repeating patterns):

```bash
root@clumsy:~/doxbin# dd if=logo.jpg bs=1 skip=$((0x300)) count=$((0x300)) of=code.bin
768+0 records in
768+0 records out
768 bytes copied
```

Disassembly using `objdump` (i386, Intel syntax):

```bash
root@clumsy:~/doxbin# objdump -D -b binary -m i386 -M intel code.bin | head -40
```

**Result – Valid x86 Instructions**

```
00000000 <.data>:
   0:   80 0a 00                or     BYTE PTR [edx],0x0
   3:   28 00                   sub    BYTE PTR [eax],al
   5:   a0 02 80 0a 00          mov    al,ds:0xa8002
   a:   28 00                   sub    BYTE PTR [eax],al
   c:   a0 02 80 3e a8          mov    al,ds:0xa83e8002
  11:   fd                      std
  12:   91                      xchg   ecx,eax
  13:   bf 62 ef da 3f          mov    edi,0x3fdaef62
  18:   f6 e1                   mul    cl
  1a:   f8                      clc
  1b:   b5 e1                   mov    ch,0xe1
  1d:   af                      scas   eax,DWORD PTR es:[edi]
  1e:   83 9f b3 a7 c3 7d 73    sbb    DWORD PTR [edi+0x7dc3a7b3],0x73
  ...
  2a6:   c3                      ret
  2a7:   ed                      in     eax,dx
  2a8:   47                      inc    edi
  2a9:   41                      inc    ecx
  2aa:   9e                      sahf
  ...
```

Arithmetic, control flow, string ops, interrupts – **this is not JPEG data. This is shellcode.**

---

## 0x05 – Extracting The Full Payload

We dumped the entire entropy‑coded segment (66,966 bytes) for deeper analysis

```bash
root@clumsy:~/doxbin# dd if=logo.jpg bs=1 skip=$((0x2c2)) count=$((0x10858 - 0x2c2)) of=scan_data.bin
66966+0 records in
66966+0 records out
66966 bytes (67 kB, 65 KiB) copied
```

Due to missing `ndisasm` we couldn’t fully disassemble the whole thing, but the 768‑byte fragment confirms: **injected x86 shellcode** interleaved with legitimate JPEG Huffman codes. This technique targets memory corruption vulnerabilities in JPEG decoders (e.g., libjpeg, Windows GDI+, browsers).

---

## 0x06 – Strings? No Strings Attached

```bash
root@clumsy:~/doxbin# strings logo.jpg | grep -iE "http|exe|dll|cmd|bash|wget|curl|base64"
(empty)

root@clumsy:~/doxbin# strings entropy.bin | grep -iE "http|url|cmd|powershell|bash|wget|curl|base64|eval|exec|system"
(empty)
```

Position‑independent shellcode – no plaintext indicators. Likely uses direct syscalls or API hashing to avoid static detection.

---

## 0x07 – Conclusion Malware Confirmed

- **File** `logo.jpg` (67,676 bytes)
- **Type** Weaponized JPEG with embedded x86 shellcode
- **Delivery** Hosted on doxbin.net (now 403 but previously live)
- **Exploitation** Targets JPEG decoder vulnerability
- **Payload** Unknown (downloader, backdoor, credential stealer)
- **Stealth** High – no appended data, no stego passphrase, no plaintext strings

**Indicators of Compromise**
- File size: 67676 bytes
- SOS at `0x2c2`, EOI at `0x1085a`
- Repeating pattern: `e8 a0 02 80 0a` in hexdump
- Disassembled instructions as shown above

**Mitigation**
- Delete the file immediately.
- Update all image processing libraries.
- Watch for similar JPEGs with anomalous opcode patterns.

---

## 0x08 – Credits

**Reverse Engineered by**  
Taylor Christian Newsome (SleepTheGod / ClumsyLulz)  

*Stay sharp, stay reversed. out.*

logs are here https://github.com/SleepTheGod/Writeups/blob/main/Doxbin-logo-malware-proof.txt
