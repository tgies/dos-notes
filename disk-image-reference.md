# DOS Disk Image Creation & Testing Reference

## Overview

This document covers all verified working approaches for:
1. Creating bootable DOS disk images
2. Testing images in QEMU
3. Testing images in dosemu

---

## Floppy Images (FAT12)

### Approach 1: mformat (current CI method)

```bash
# Create blank image
dd if=/dev/zero of=floppy.img bs=512 count=2880

# Format with mformat
mformat -f 1440 -v MSDOS4 -i floppy.img ::

# Then use dosemu to SYS the image (see dosemu section)
```

**Pros:** Simple, one command formats
**Cons:** Requires mtools, still needs dosemu for SYS

### Approach 2: mkfs.fat + boot sector patching (no dosemu needed)

This approach creates a bootable floppy using only mkfs.fat and mtools, without needing dosemu to run SYS.

```bash
#!/bin/bash
# Requires: dosfstools, mtools
# Requires: extracted boot sector and system files from CI artifacts

OUTPUT="floppy.img"
BOOT_SECTOR="bootsect-floppy.bin"  # 512-byte DOS boot sector
IO_SYS="IO.SYS"
MSDOS_SYS="MSDOS.SYS"
COMMAND_COM="COMMAND.COM"

# 1. Create blank 1.44MB image
dd if=/dev/zero of="$OUTPUT" bs=512 count=2880 status=none

# 2. Format as FAT12 - CRITICAL: do NOT use -n flag (volume label)
#    The -n flag puts volume label as first directory entry,
#    but DOS boot code expects IO.SYS as the FIRST entry
mkfs.fat -F 12 "$OUTPUT" >/dev/null

# 3. Patch boot sector
#    - Preserve mkfs.fat's BPB (bytes 11-61) - describes filesystem geometry
#    - Overlay DOS boot code (bytes 0-10 and 62-511)

# Extract BPB from mkfs.fat filesystem
dd if="$OUTPUT" of=/tmp/bpb.bin bs=1 skip=11 count=51 status=none

# Start with DOS boot sector
cp "$BOOT_SECTOR" /tmp/patched-boot.bin

# Overlay the mkfs.fat BPB
dd if=/tmp/bpb.bin of=/tmp/patched-boot.bin bs=1 seek=11 conv=notrunc status=none

# Write patched boot sector back
dd if=/tmp/patched-boot.bin of="$OUTPUT" conv=notrunc status=none

# 4. Copy system files - ORDER MATTERS! IO.SYS must be FIRST
mcopy -i "$OUTPUT" "$IO_SYS" ::/IO.SYS
mcopy -i "$OUTPUT" "$MSDOS_SYS" ::/MSDOS.SYS
mcopy -i "$OUTPUT" "$COMMAND_COM" ::/COMMAND.COM

# 5. Set file attributes
mattrib -i "$OUTPUT" +r +s +h ::/IO.SYS
mattrib -i "$OUTPUT" +r +s +h ::/MSDOS.SYS
```

**Key points:**
- Do NOT use `-n` flag with mkfs.fat (volume label breaks boot)
- IO.SYS must be the FIRST file copied (first directory entry)
- Must patch boot sector to have DOS boot code with mkfs.fat's BPB

**Pros:** No dosemu needed, pure Linux tools
**Cons:** Requires pre-extracted boot sector from working DOS image

---

## Hard Disk Images (FAT16)

### Approach 1: mkfatimage16 + dosemu SYS (current CI method, ONLY working method)

```bash
#!/bin/bash
# Requires: dosemu2 (provides mkfatimage16 and dosemu)

SIZE_KB=65536  # 64MB
OUTPUT="target.img"
STAGING_DIR="/path/to/staging"  # Contains IO.SYS, MSDOS.SYS, COMMAND.COM, SYS.COM

# 1. Create FAT16 image with mkfatimage16
mkfatimage16 -l MSDOS4 -k "$SIZE_KB" -f "$OUTPUT" -p

# 2. Mark partition as bootable (mkfatimage16 leaves it inactive)
#    Partition entry is at offset 574 (128-byte dosemu header + 446)
printf '\x80' | dd of="$OUTPUT" bs=1 seek=574 count=1 conv=notrunc 2>/dev/null

# 3. Create dosemu config
cat > dosemurc << EOF
\$_hdimage = "$STAGING_DIR $OUTPUT"
EOF

# 4. Create AUTOEXEC.BAT in staging dir
cat > "$STAGING_DIR/AUTOEXEC.BAT" << 'EOF'
@ECHO OFF
SYS C: D:
COPY C:\COMMAND.COM D:\
exitemu
EOF

# 5. Run dosemu to transfer system
dosemu -f dosemurc -dumb -td -kt < /dev/null

# 6. Strip dosemu 128-byte header for raw image (QEMU/VirtualBox)
dd if="$OUTPUT" of="${OUTPUT%.img}.raw.img" bs=128 skip=1
```

**Why mkfs.fat doesn't work for FAT16:**
- mkfs.fat uses 4 reserved sectors for FAT16 (hardcoded minimum)
- DOS boot code expects 1 reserved sector
- The `-R 1` flag only sets minimum, doesn't override FAT16's 4-sector requirement
- This causes "Non-System disk" error because boot code can't find FAT/root directory

**Partition geometry notes:**
- mkfatimage16 creates partition at sector 17 (old CHS geometry)
- Modern sfdisk defaults to sector 2048 (LBA alignment)
- Must match geometry when patching or recreating images

---

## QEMU Testing

### Hard Disk Image

```bash
# Basic boot test (VGA text to terminal)
timeout 5 qemu-system-i386 \
    -drive file=image.raw.img,format=raw \
    -boot c \
    -nographic
```

### Floppy Image

```bash
timeout 5 qemu-system-i386 \
    -fda floppy.img \
    -boot a \
    -nographic
```

### Important Notes

1. **Use `timeout`** - DOS will sit at prompt forever otherwise

2. **Use `-nographic`** - Captures VGA text output to terminal

3. **Do NOT pipe to `head`** - Breaks output buffering, you'll get no output
   ```bash
   # BAD - no output
   timeout 5 qemu-system-i386 ... | head -50

   # GOOD - output works
   timeout 5 qemu-system-i386 ...
   ```

4. **Specify `format=raw`** - Avoids warning about format detection

5. **Alternative output modes** (if -nographic doesn't work):
   ```bash
   # Serial output
   -display none -serial stdio

   # Monitor multiplexed with serial
   -nographic -serial mon:stdio
   ```

### Common QEMU Error Messages

| Error | Cause | Fix |
|-------|-------|-----|
| `no active partition found` | Partition not marked bootable | Set byte at partition entry to 0x80 |
| `partition signature != 55AA` | Boot sector missing/corrupt | Check boot sector bytes 510-511 |
| `Non-System disk or disk error` | IO.SYS not first entry, or BPB mismatch | Ensure IO.SYS is first dir entry, check reserved sectors |
| No output at all | Piped to head/grep | Don't pipe QEMU output |

---

## dosemu Testing

### Basic Invocation

```bash
dosemu -f config.dosemurc -dumb -td -kt < /dev/null
```

**Flags:**
- `-f config.dosemurc` - Use custom config file
- `-dumb` - Dumb terminal mode (text output to stdout)
- `-td` - Transparent disk access
- `-kt` - Keyboard timeout (auto-exit when no input)
- `< /dev/null` - No stdin (prevents hanging)

### Config File Format

**Directory as drive + disk image:**
```
$_hdimage = "/path/to/staging/dir /path/to/disk.img"
```
- First path = C: drive (directory mode, boots from here)
- Second path = D: drive (disk image)

**Floppy image:**
```
$_hdimage = "/path/to/staging/dir"
$_floppy_a = "/path/to/floppy.img:threeinch"
```

### AUTOEXEC.BAT for Image Creation

```batch
@ECHO OFF
SYS C: D:
COPY C:\COMMAND.COM D:\
exitemu
```

**Important:** Use DOS line endings (`\r\n`):
```bash
printf '@ECHO OFF\r\n' > AUTOEXEC.BAT
printf 'SYS C: D:\r\n' >> AUTOEXEC.BAT
```

### Common Issues

1. **`exitemu` before output flushes** - Output may not appear. Use `timeout` wrapper instead of relying on exitemu for testing.

2. **Command is `dosemu` not `dosemu2`** - Even though package is dosemu2

3. **KVM permission errors** - Harmless warning, dosemu still works:
   ```
   ERROR: KVM: error opening /dev/kvm: Permission denied
   ```

---

## Boot Sector Anatomy

```
Offset  Size  Description
------  ----  -----------
0       3     Jump instruction (EB xx 90 or E9 xx xx)
3       8     OEM name (e.g., "MSDOS4.0")
11      51    BPB (BIOS Parameter Block) - filesystem geometry
62      448   Boot code
510     2     Signature (55 AA)
```

### BPB Fields (selected)

```
Offset  Size  Description
------  ----  -----------
11      2     Bytes per sector (usually 512)
13      1     Sectors per cluster
14      2     Reserved sectors (FAT12: usually 1, FAT16: 1 or 4)
16      1     Number of FATs (usually 2)
17      2     Root directory entries
19      2     Total sectors (16-bit, 0 if >65535)
21      1     Media descriptor
22      2     Sectors per FAT
24      2     Sectors per track
26      2     Number of heads
28      4     Hidden sectors (partition start offset)
32      4     Total sectors (32-bit)
```

---

## Extracting Components from Working Images

### Extract boot sector from floppy
```bash
dd if=working-floppy.img of=bootsect-floppy.bin bs=512 count=1
```

### Extract partition boot sector from HD image
```bash
# First find partition offset
OFFSET=$(sfdisk -J image.raw.img | python3 -c "import json,sys; print(json.load(sys.stdin)['partitiontable']['partitions'][0]['start'])")

# Extract boot sector
dd if=image.raw.img of=bootsect-hd.bin bs=512 skip=$OFFSET count=1
```

### Extract MBR boot code (first 446 bytes, excludes partition table)
```bash
dd if=image.raw.img of=mbr-code.bin bs=446 count=1
```

### Extract system files from image
```bash
# For floppy or partition image
mcopy -i image.img ::/IO.SYS ./IO.SYS
mcopy -i image.img ::/MSDOS.SYS ./MSDOS.SYS
mcopy -i image.img ::/COMMAND.COM ./COMMAND.COM
```
