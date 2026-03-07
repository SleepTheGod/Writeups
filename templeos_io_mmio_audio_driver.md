# TempleOS IO-to-MMIO Audio Driver Guide

This document outlines how to create a **TempleOS-native audio driver** that works from **traditional I/O ports to memory-mapped MMIO registers**. The goal is a fully native driver compatible with the TempleOS ISO.

---

## 1. Concept Overview
TempleOS drivers do not use syscalls. Instead, they:

1. Detect the audio device using PCI configuration space.
2. Map device MMIO space or access legacy I/O ports.
3. Initialize the audio codec (HDA or similar).
4. Allocate DMA buffer for streaming.
5. Provide simple playback API (RAW PCM / WAV).

This guide combines **I/O port programming** (for legacy or early devices) with **MMIO** access (modern HDA). The code can be included in the TempleOS ISO.

---

## 2. PCI Detection
```
for bus in 0..255:
  for dev in 0..31:
    for fun in 0..7:
      vendor_device = PCIReadU32(bus, dev, fun, 0x00)
      if (vendor_device & 0xFFFF) == VENDOR_ID:
        device_id = PCIReadU16(bus, dev, fun, 0x02)
        if device_id in SUPPORTED_AUDIO_DEVICES:
          bar0 = PCIReadU32(bus, dev, fun, 0x10) & ~0xF
          irq  = PCIReadU8(bus, dev, fun, 0x3C)
```
**Supported Devices:** Intel ICH6-ICH10, Realtek (vendor-specific IDs)

---

## 3. Legacy I/O Port Initialization (optional)
For older audio chips that use **I/O ports**:
```
OUTB(port + 0x00, 0x01)   ; reset
OUTB(port + 0x00, 0x00)   ; clear reset
```
- Ports vary by chipset (e.g., 0x220–0x22F for AC'97)
- Use `INB` / `OUTB` for byte, `INW` / `OUTW` for 16-bit
- Use `Sleep(ms)` after writes for stable reset

---

## 4. MMIO Programming (HDA / Modern Audio)
### Global Control
```
MMIO + 0x08  ← Read old
MMIO + 0x08  ← old | 0x1       ; reset
Sleep(10)
MMIO + 0x08  ← old & ~0x1       ; clear reset
Sleep(10)
```

### Codec Commands
```
MMIO + 0x60  ← 0x000F0000      ; query codec ID
codec_id = MMIO + 0x68 READ
MMIO + 0x60  ← 0x00070100      ; power on
MMIO + 0x60  ← 0x00020B00      ; enable front output pin
MMIO + 0x60  ← 0x0007077F      ; unmute / max volume
MMIO + 0x60  ← 0x00020C00      ; optional pin setup
```

### Volume Control
```
level = vol * 127
MMIO + 0x60 ← 0x00070700 | level
```

---

## 5. DMA / Stream Buffer
TempleOS kernel automatically handles DMA if you allocate a global buffer:
```
snd_buf     = Alloc(SND_BUF_LEN)
snd_buf_ptr = snd_buf
```
To write audio:
```
MemCpy(snd_buf_ptr, audio_data, len)
snd_buf_ptr += len
if snd_buf_ptr >= snd_buf + SND_BUF_LEN:
    snd_buf_ptr = snd_buf
```
This works for both legacy and HDA setups as long as the driver signals the device (via MMIO/I/O).  

---

## 6. Playback API (Minimal)
```
U0 PlayRawPCM(U8 *buf, I64 len) { ... }
U0 PlayWavFile(U8 *filename) { ... }
U0 SetVolume(F64 vol) { ... }
```
- All functions write directly to `snd_buf` or MMIO registers.
- TempleOS Sup1 task streams automatically from the buffer.

---

## 7. Full Initialization Sequence
```
HDAInit()          ; detect device + init MMIO
InitSoundBuffer()  ; allocate snd_buf
PlayRawPCM(...)    ; enqueue audio
SetVolume(1.0)     ; set max volume
```
- Optional I/O port writes for older chips before MMIO
- IRQ line read from PCI config, but TempleOS handles interrupts internally if using Sup1 DMA

---

## 8. Notes
- This driver template supports **Intel HDA primarily**, with optional legacy I/O support.
- Realtek / VIA require vendor-specific verbs.
- No external libraries or syscalls needed; pure MMIO/I/O port + memory buffer.
- Works natively in a TempleOS ISO.

---

This file serves as a **complete roadmap** for writing a TempleOS driver from **I/O ports to MMIO**, including DMA setup and simple playback.
