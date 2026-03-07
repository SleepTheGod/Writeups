# TempleOS Firmware Dumping Guide

This document describes how to create a **TempleOS-native tool to dump firmware** from PCI or memory-mapped devices. It can be used on the TempleOS ISO.

---

## 1. Concept Overview
Firmware dumping in TempleOS involves:

1. **Detecting the hardware** (PCI or known MMIO base addresses).
2. **Mapping the memory** or accessing I/O space.
3. **Reading the firmware bytes** sequentially.
4. **Saving the dump** to a file.

All operations are direct memory/I/O accesses — no syscalls are used.

---

## 2. Detect PCI Device
```
for bus in 0..255:
  for dev in 0..31:
    for fun in 0..7:
      vendor_device = PCIReadU32(bus, dev, fun, 0x00)
      if (vendor_device & 0xFFFF) == VENDOR_ID:
        device_id = PCIReadU16(bus, dev, fun, 0x02)
        if device_id == TARGET_DEVICE:
          bar0 = PCIReadU32(bus, dev, fun, 0x10) & ~0xF
```
- `VENDOR_ID` = vendor (Intel, Realtek, etc.)
- `TARGET_DEVICE` = specific device to dump firmware from

---

## 3. Memory-mapped Firmware Dump
If firmware is in MMIO space:
```
U8 *fw_base = bar0
I64 fw_size = FW_SIZE_BYTES  ; known or max size
CDoc *dump = DocNew(FW_SIZE_BYTES)
for offset = 0 to fw_size-1:
    dump->body[offset] = *(fw_base + offset)
DocSave(dump, "FirmwareDump.bin")
DocDel(dump)
```
- `FW_SIZE_BYTES` = size of firmware to dump (commonly 64 KB, 128 KB, or device-specific)
- MMIO can be read with `InU8/16/32` if needed.

---

## 4. I/O Port Firmware Dump
Some legacy devices store firmware in I/O space:
```
U16 port = BASE_PORT
I64 fw_size = FW_SIZE_BYTES
CDoc *dump = DocNew(fw_size)
for offset = 0 to fw_size-1:
    dump->body[offset] = InU8(port + offset)
DocSave(dump, "FirmwareDump.bin")
DocDel(dump)
```
- `BASE_PORT` = starting I/O port of firmware (chipset specific)
- Use `InU16` or `InU32` for wider reads if supported

---

## 5. Saving Firmware
- Use `DocNew(size)` to allocate buffer.
- Write to `dump->body[offset]` for each byte.
- `DocSave(dump, "Filename.bin")` saves it to disk.
- `DocDel(dump)` frees memory.

---

## 6. Safety Notes
- Dumping firmware may trigger undefined behavior if device registers are not read-only.
- Avoid dumping from active devices unless safe to reset them.
- Some firmware areas may be protected; attempt only on accessible MMIO/I/O regions.
- Ensure sufficient memory for buffer allocation.

---

## 7. Complete Sequence Example
```
HDAInit()              ; detect device
fw_base = hda.base      ; example MMIO
fw_size = 0x10000       ; 64 KB firmware
CDoc *dump = DocNew(fw_size)
for i=0 to fw_size-1:
    dump->body[i] = *(fw_base + i)
DocSave(dump, "HDA_Firmware.bin")
DocDel(dump)
```
- Replace `hda.base` with your target device BAR.
- Adjust `fw_size` to match firmware.

---

This guide provides **TempleOS-native firmware dumping** from PCI/MMIO/I/O space, suitable for inclusion in a TempleOS ISO.
