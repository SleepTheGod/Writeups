# TempleOS Native HD Audio Driver Hex Reference

This document lists all the PCI and MMIO accesses required for a minimal Intel HD Audio (ICH9/ICH10) driver in TempleOS.

---

## 1. PCI CONFIGURATION SPACE (HEX EXACT)

### Detect device
```
PCI CFG 0x00  READ U32   → VENDOR | DEVICE
mask 0x0000FFFF == 0x8086
```

### Read device ID
```
PCI CFG 0x02  READ U16
0x293E  (ICH9)
0x3A3E  (ICH10)
```

### Enable MMIO + Bus Master
```
PCI CFG 0x04  READ  U16
PCI CFG 0x04  WRITE U16  |= 0x0006
```

### Read MMIO base
```
PCI CFG 0x10  READ U32
MMIO_BASE = value & 0xFFFFFFF0
```

### Read IRQ
```
PCI CFG 0x3C  READ U8
```

---

## 2. HD AUDIO MMIO REGISTERS (BAR0)

### Global Control (GCTL)
```
MMIO + 0x08  READ  U32
MMIO + 0x08  WRITE U32  = old | 0x00000001   ; reset assert
MMIO + 0x08  WRITE U32  = old & 0xFFFFFFFE   ; reset clear
```

---

## 3. CODEC COMMAND / RESPONSE RING

### Command Output
```
MMIO + 0x60  WRITE U32
```

### Response Input
```
MMIO + 0x68  READ  U32
```

---

## 4. ALL HDA VERBS USED (FULL HEX LIST)

### Get codec ID
```
WRITE 0x000F0000
READ  RESPONSE @ 0x68
```

### Power up audio function group
```
WRITE 0x00070100
```

### Enable front DAC pin
```
WRITE 0x00020B00
```

### Set pin control (output enable)
```
WRITE 0x00020C00
```

### Unmute + max volume
```
WRITE 0x0007077F
```

### Volume control (runtime)
```
WRITE 0x00070700 | level   ; level = 0x00–0x7F
```

---

## 5. STREAM / DMA (TempleOS Sup1 compatible)

TempleOS kernel reads directly from the following buffer:
```
snd_buf
snd_buf_ptr
snd_buf_len
```
Memory writes:
```
MemCpy(snd_buf_ptr, data, size)
snd_buf_ptr += size
wrap at snd_buf + snd_buf_len
```
No additional MMIO writes are required.

---

## 6. COMPLETE HEX WRITE SUMMARY (ONE LIST)

```
PCI:
  0x04 |= 0x0006

MMIO:
  +0x08 ← |0x00000001
  +0x08 ← &0xFFFFFFFE
  +0x60 ← 0x000F0000
  +0x60 ← 0x00070100
  +0x60 ← 0x00020B00
  +0x60 ← 0x00020C00
  +0x60 ← 0x0007077F
  +0x60 ← 0x00070700–0x0007077F
```

---

**Notes:**
- Supports only Intel HDA (ICH9/ICH10).
- Sup1 sound task automatically streams DMA from `snd_buf`.
- Other chipsets require vendor-specific initialization.
- No syscalls are used; all accesses are direct MMIO/PCI.
