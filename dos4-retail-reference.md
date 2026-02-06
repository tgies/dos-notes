# DOS 4 Retail Disk Reference

## Build Flavors

The DOS 4 source supports three build configurations via `src/INC/VERSION.INC`:

| Flavor | IBMVER | IBMCOPYRIGHT | System Files | Status |
|--------|--------|--------------|--------------|--------|
| **msdos** | TRUE | FALSE | IO.SYS, MSDOS.SYS | ✅ Default build |
| **pcdos** | TRUE | TRUE | IBMBIO.COM, IBMDOS.COM | ✅ Works |
| msdos-portable | FALSE | FALSE | IO.SYS, MSDOS.SYS | ❌ Doesn't build |

### What The Flags Actually Control

**IBMVER** controls **IBM PC hardware-specific code**:
- `IF IBMVER` blocks enable direct hardware access
- INT 10H video BIOS calls (DEBUG.COM screen size detection)
- 8259 PIC manipulation (DEBUG.COM interrupt masking during trace)
- INT 1Ch hardware timer (PRINT.COM background printing)
- PCjr ROM cartridge scanning (COMMAND.COM `ROM_SCAN`/`ROM_EXEC` in MSHALO.ASM)

**IBMCOPYRIGHT** controls **IBM branding**:
- System file names: IBMBIO.COM/IBMDOS.COM vs IO.SYS/MSDOS.SYS
- SYS.COM looks for different filenames
- FORMAT.COM writes different boot sector OEM strings
- Copyright messages in various utilities

### The Three Configurations Explained

1. **msdos** (IBMVER=TRUE, IBMCOPYRIGHT=FALSE)
   - OEM MS-DOS for IBM PC clones (Compaq, Dell, HP)
   - IBM PC-compatible hardware code, MS-DOS branding
   - What most people think of as "MS-DOS"

2. **pcdos** (IBMVER=TRUE, IBMCOPYRIGHT=TRUE)
   - Official IBM PC-DOS
   - IBM PC hardware code, IBM branding

3. **msdos-portable** (IBMVER=FALSE, IBMCOPYRIGHT=FALSE)
   - For non-IBM-compatible machines (DEC Rainbow, HP 150, Wang PC)
   - DOS-only system calls, no direct hardware access

### Files With IBMVER/MSVER Conditionals

Key files with hardware-specific code:
```
src/CMD/COMMAND/TCODE.ASM    - PCjr ROM cartridge support (includes MSHALO.ASM)
src/CMD/COMMAND/TMISC1.ASM   - ROM_EXEC, ROM_SCAN references
src/CMD/DEBUG/DEBUG.ASM      - INT 10H video, screen size detection
src/CMD/DEBUG/DEBCOM3.ASM    - 8259 PIC mask save/restore
src/CMD/PRINT/PRIDEFS.ASM    - Hardware timer vs DOS timing
src/CMD/MORE/MORE.ASM        - Control character handling
src/INC/MSHALO.ASM           - PCjr ROM cartridge scanning code
```

### Build Commands

```bash
# Build MS-DOS (default)
./mak.sh

# Build IBM PC-DOS
./mak.sh --flavor=pcdos

# Explicit MS-DOS
./mak.sh --flavor=msdos

# Clean before building (required when switching flavors)
./mak.sh --clean --flavor=pcdos
```

---

## Retail Disk Sets

### IBM PC-DOS 4.01 - 360KB (5 disks)

Source: https://archive.org/details/ibm-pc-dos-v4.01

| Disk | Name | Size | Key Contents |
|------|------|------|--------------|
| 1 | Install | 338KB | IBMBIO.COM, IBMDOS.COM, COMMAND.COM, FORMAT, FDISK, SYS, SELECT |
| 2 | Select | 350KB | SELECT.EXE, BACKUP, RESTORE, EGA.CPI, XMA2EMS |
| 3 | Operating 1 | 324KB | DEBUG, ATTRIB, XCOPY, MEM, BASICA, utilities |
| 4 | Operating 2 | 322KB | CHKDSK, PRINT, DOS Shell (SHELLC.EXE - 154KB!) |
| 5 | Operating 3 | 325KB | IBMBIO/IBMDOS again, drivers (ANSI.SYS etc), CPI files |

**Notes:**
- Disk 1 is bootable (installer boot disk)
- Disk 5 is ALSO bootable (working system disk)
- DOS Shell takes almost half of Disk 4

### MS-DOS 4.01a - 720KB (3 disks)

Source: https://archive.org/details/ms-dos-4.01a

| Disk | Name | Size | Key Contents |
|------|------|------|--------------|
| 1 | Installation | 690KB | IO.SYS, MSDOS.SYS, COMMAND.COM, SELECT, FORMAT, FDISK, SYS, drivers, CPI files |
| 2 | Operating System | 682KB | COMMAND.COM, utilities (DEBUG, CHKDSK, etc), GWBASIC, LINK, EMM386 |
| 3 | Shell | 706KB | IO.SYS, MSDOS.SYS, COMMAND.COM, DOS Shell, BACKUP/RESTORE, utilities |

**Notes:**
- Disk 1 is bootable (installer)
- Disk 3 is bootable (has system files)
- Much more consolidated than 360KB set

---

## Size Constraints by Floppy Format

| Format | Capacity | Typical Usage |
|--------|----------|---------------|
| 360KB (5.25" DD) | 368,640 bytes | 5-6 disk set |
| 720KB (3.5" DD) | 737,280 bytes | 3 disk set |
| 1.2MB (5.25" HD) | 1,228,800 bytes | 2 disk set |
| 1.44MB (3.5" HD) | 1,474,560 bytes | 1-2 disk set |

---

## Utility Priority for Space-Constrained Disks

### Tier 1: Essential (include on any bootable disk)
- FORMAT.COM (22KB) - format disks
- SYS.COM (11KB) - make disks bootable
- FDISK.EXE (61KB) - partition hard disks
- CHKDSK.COM (18KB) - check disk integrity

### Tier 2: Important utilities
- DEBUG.COM (21KB) - debugging/hex editing
- EDLIN.COM (14KB) - line editor
- MODE.COM (23KB) - device configuration
- ATTRIB.EXE (18KB) - file attributes
- XCOPY.EXE (17KB) - extended copy

### Tier 3: Additional utilities
- BACKUP.COM, RESTORE.COM - backup tools
- MEM.EXE - memory display
- FC.EXE - file compare
- FIND.EXE, SORT.EXE - text utilities
- PRINT.COM - print spooler

### Tier 4: Drivers and international
- ANSI.SYS, DISPLAY.SYS, DRIVER.SYS
- KEYB.COM, KEYBOARD.SYS
- CPI files (code pages)

### Tier 5: Optional/Large
- EMM386.SYS (88KB) - expanded memory
- ~~SELECT.EXE (95KB)~~ - **NOT AVAILABLE** (depends on DOS Shell)
- ~~DOS Shell (SHELLC.EXE ~154KB)~~ - **NOT OPEN-SOURCED**
- GWBASIC.EXE (81KB) - not in open source release

---

## Sources

- IBM PC-DOS 4.01 360KB: https://archive.org/details/ibm-pc-dos-v4.01
- MS-DOS 4.01a 720KB: https://archive.org/details/ms-dos-4.01a
- PCjs detailed listings: https://www.pcjs.org/software/pcx86/sys/dos/microsoft/4.01/
- WinWorld downloads: https://winworldpc.com/product/ms-dos/4x
