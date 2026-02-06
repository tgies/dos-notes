# DOS 6.0 (Astro) Hidden Features, Debug Modes & Undocumented Switches

## Overview

MS-DOS 6.0 "Astro" source code (`astro/`) contains undocumented command-line switches,
developer debug code left in shipping builds, conditional compilation flags, and
internal testing infrastructure.

---

## Undocumented Command-Line Switches

### FORMAT.COM — 12 Hidden Switches

- **Files:** `cmd/format/forparse.inc`, `cmd/format/forswtch.inc`, `cmd/format/forinit.asm`, `cmd/format/format.asm`

| Switch | Flag | Type | Purpose |
|--------|------|------|---------|
| `/AUTOTEST` | `SWITCH_AUTOTEST` (0800h) | Internal QA | Automated test mode — suppresses all prompts, skips "format another?" |
| `/SELECT` | `SWITCH_SELECT` | Setup integration | Used when FORMAT is called from DOS Setup/Install |
| `/BACKUP` | `SWITCH_BACKUP` | Internal | Used during backup operations, suppresses interactive prompts |
| `/Z` | Conditional (`ShipDisk`) | OEM | Force 1 sector/cluster — conditional compilation |
| `/FS:xxxxx` | Conditional (`FSExec`) | Planned | File system selection — conditional compilation, may be unfinished feature |
| `/1` | `SWITCH_1` | Compatibility | Legacy single-sided format (from DOS 1.x/2.x) |
| `/4` | `SWITCH_4` | Compatibility | Legacy 360KB in 1.2MB drive |
| `/8` | `SWITCH_8` | Compatibility | Legacy 8 sectors/track format |
| `/B` | `SWITCH_B` | Compatibility | Reserve space for system files |
| `/T:nn` | Numeric param | Advanced | Specify track count |
| `/N:nn` | Numeric param | Advanced | Specify sectors per track |
| `/U` | `SWITCH_U` | Semi-documented | Unconditional format (no UNFORMAT data saved) |
| `/Q` | `SWITCH_Q` | Semi-documented | Quick format |

**Key behavior for `/AUTOTEST`, `/SELECT`, `/BACKUP`** — `format.asm:1441`:
```asm
test    SwitchMap,(SWITCH_SELECT or SWITCH_AUTOTEST or SWITCH_BACKUP)
jz      @F
stc                     ; flag automatic 'no' response
jmp     SHORT Exit_More
```
All three suppress "Format another?" prompt for unattended operation. Also at `format.asm:2615,2640,2645` they affect verification and formatting behavior.

**Switch definitions:** `forswtch.inc:20` — `SWITCH_AUTOTEST EQU 0800h`

### DELTREE.EXE — /Y (Suppress Confirmation)

- **File:** `c6ers/newcmds/deltree.c:107-122`
- **Switch:** `/Y`
- **Effect:** Sets `fAsk = FALSE` — deletes directory trees without prompting

```c
/*
 *  /Y is NOT localized, as is the standard for 1-char switch names
 */
if (!strcmp (*v+1, "y") || !strcmp (*v+1, "Y"))
    fAsk = FALSE;
```

**Not shown in `/?` help output.** The comment reveals MS's localization convention: single-character switches keep their English letter across all language versions.

### MOVE.EXE — /Y (Suppress Overwrite Confirmation)

- **File:** `c6ers/newcmds/move.c:248-266`
- **Switch:** `/Y`
- **Effect:** Suppresses confirmation when overwriting existing files

### BACKUP.COM — /F[:size] (Format Target Diskette)

- **File:** `cmd/backup/backup.c:43`
- **Switch:** `/F[:size]`
- **Status:** **Explicitly marked "undocumented" in the source code itself**
- **Purpose:** Formats target backup diskette to specified size during backup operation

---

## SmartDrive (SMARTDRV/BAMBI) Hidden Features

### "Chuck's Debug Stuff" — Internal Cache Pointers

- **File:** `dev/smartdrv/int2f.asm:458-486`
- **Interface:** INT 2Fh, AX=4A10h, BX=BAMBI_INTERNAL_POINTERS
- **Status:** Compiled in via `if 1` (always true) — shipped in retail

```asm
if      1                       ;  ***Chuck's debug stuff!!!
cmp     bx,BAMBI_INTERNAL_POINTERS
je      return_internals
endif
```

**Returns pointer to structure at `internals_struct` (lines 479-484):**
```asm
internals_struct:
    dw      offset validindexoffset
    dw      offset DirtyIndexOffset
    dw      ElementIdentifierLowordOffset
    dw      ElementIdentifierHiwordOffset
    dw      Queuelength
```

Exposes SmartDrive's internal cache data structure layout for debugging/monitoring.

### Magic Number 1234h — Popup Trigger

- **File:** `dev/smartdrv/int2f.asm:462-527`
- **Interface:** INT 2Fh, AX=4A10h, BX=1234h
- **Effect:** Calls `warning_pop_up` with `al=3`

```asm
cmp     bx,1234h
je      display_the_message
...
display_the_message:
    mov     al,3
    call    warning_pop_up
    jmp     exit_no_chain
```

The popup system (`dev/smartdrv/popup.asm`) intercepts INT 9 (keyboard) for custom dialog handling. Two popup messages defined in `rtext.asm`:
- **popup1:** *"ATTENTION: A serious disk error has occured while writing to drive @. Retry (r)?"*
- **popup2:** *"Waiting for system shutdown."*

The `al=3` parameter likely triggers a third/test popup or the shutdown message.

### SmartDrive Debug/Tracker Flags

- **File:** `dev/smartdrv/bambi.asm:8-10`

```asm
debug = 0       ; Debug output (was 1 during development)
tracker = 0     ; Tracker events (set to 0ffffh to enable)
```

### SmartDrive Dirty Write Debug Flag (left on)

- **File:** `dev/smartdrv/dirtywrt.asm:2`
- **Setting:** `debug = 1`
- **Status:** Debug flag is set to 1 in what is presumably shipped source.
- **Effect:** Lines 88, 110, 283 have `if debug` blocks that are compiled in
- **Contains:** BUGBUG comment at line 257 about optimization opportunity

### QEMM Workaround in Reinitialize

- **File:** `dev/smartdrv/int2f.asm:511-522`
```asm
;QEMM in a dos box will toast the handle, dont know why
;but we don't want to eat the disk, and sincethe resize
;part of this api is removed, we just do the dorky way
```
The reinitialize function was simplified because QEMM (Quarterdeck Expanded Memory Manager) corrupted SmartDrive's XMS handle in DOS boxes.

---

## EMM386 Debug Infrastructure

### LoadHi Debug Query

- **File:** `dev/emm386/lhvxd/instinit.asm:501-571`
- **Procedure:** `LoadHi_Debug_Query` (lines 503-569)
- **Condition:** `IFDEF DEBUG`
- **Purpose:** Debug query handler for LoadHigh VxD operations

### Debug Macros

- **File:** `dev/emm386/debmac.inc`
- **Contains:** Extensive `ifdef DEBUG` blocks (lines 25, 50, 78, 99, 197, 207)
- **Functions:** Conditional debug output infrastructure for EMM386

### SmartDrive VxD Debug

- **File:** `dev/smartdrv/sdvxd/debug.inc`
- **Macros:** `Debug_Test_Valid_Handle`, `Out_Debug_String`, `In_Debug_Chr`, `Test_Debug_Installed`, `Queue_Debug_String`, `Debug_Test_Cur_VM`
- **Condition:** Active when `DEBLEVEL=1` (from vmm.inc)

---

## BUGBUG Comments (Known Issues Left in Code)

| File | Line | Issue |
|------|------|-------|
| `dev/smartdrv/dirtywrt.asm` | 257 | Optimization opportunity in dirty write |
| `dev/smartdrv/bambi.asm` | 519 | Register preservation question |
| `dev/smartdrv/queueman.asm` | 366 | Queue management issue |
| `dev/smartdrv/queueman.asm` | 469 | Another queue issue |

These BUGBUG markers are Microsoft's internal convention for "known bug, fix later".

---

## Developer Names in Source

| Name | Component | File |
|------|-----------|------|
| **Chuck** | SmartDrive debug code | `dev/smartdrv/int2f.asm:458,466` |
| **Ericst** | CHOICE.COM | `c6ers/choice/choice.asm:1-3` (created 5/14/92) |
| **c-PaulB** | Format/utility tools | Various |
| **Steve Zeck** | Compression (EXPAND) | `c6ers/tools2/expand/expand.c` |
| **David Dickman** | Compression (EXPAND) | `c6ers/tools2/expand/expand.c` |
| **scottq** | SmartDrive MagicDrv detection | `dev/smartdrv/drvtype.asm:337`, `bambinit.asm:789` (7/30/92) |
| **Floyd Rogers [floydr]** | h2inc tool | Multiple modifications 1989 |
| **Jameelh** | h2inc tool | Multiple files |
| **Lindsay Harris [lindsayh]** | h2inc tool | Multiple files |

---

## Conditional Compilation Flags

| Flag | File | Default | Purpose |
|------|------|---------|---------|
| `debug` | `dev/smartdrv/bambi.asm:8` | 0 | SmartDrive debug output |
| `debug` | `dev/smartdrv/dirtywrt.asm:2` | 1 (!) | Dirty write debug |
| `tracker` | `dev/smartdrv/bambi.asm:9` | 0 | SmartDrive event tracking |
| `USE_VALID` | `dev/smartdrv/bambi.asm:1741` | Conditional | Alternate debug code path |
| `DEBUG` | `dev/emm386/debmac.inc` | Undefined | EMM386 debug macros |
| `DEBUG` | `dev/emm386/lhvxd/instinit.asm` | Undefined | LoadHigh debug |
| `DBLSPACE_HOOKS` | `cmd/format/` | Conditional | DoubleSpace integration in FORMAT |
| `ShipDisk` | `cmd/format/forparse.inc` | Conditional | Enables FORMAT /Z switch |
| `FSExec` | `cmd/format/forparse.inc` | Conditional | Enables FORMAT /FS: switch |

---

## MagicDrive (Stacker) Detection

- **Files:** `dev/smartdrv/drvtype.asm`, `dev/smartdrv/bambinit.asm:789`
- **Developer:** scottq (7/30/92)
- **Purpose:** SmartDrive contains extensive code to detect MagicDrive/Stacker compressed drives and handle them separately from normal drives
- **Significance:** Shows the competitive landscape — MS had to ensure SmartDrive worked correctly with third-party disk compression before shipping their own DoubleSpace

---

## SmartDrive Popup Dialog System

- **File:** `dev/smartdrv/popup.asm`
- **Mechanism:** Custom INT 9 (keyboard) handler captures keypresses for inline dialogs
- **Messages (from `rtext.asm`):**
  1. *"ATTENTION: A serious disk error has occured while writing to drive @. Retry (r)?"*
  2. *"Waiting for system shutdown."*
- **Note:** The `@` in popup1 is a placeholder replaced with the drive letter at runtime

---

## UNIX Compatibility Artifacts

- **File:** `tools/include/SYS/TYPES.H:14`
- **Comment:** `/* i-node number (not used on DOS) */`
- **Purpose:** UNIX type definitions included for cross-compilation compatibility but unused on DOS
- **Significance:** Shows that MS's build tools had UNIX heritage (Xenix)

---

## Summary of Most Notable Findings

1. **FORMAT /AUTOTEST** — Internal QA switch for fully automated formatting, never documented
2. **DELTREE /Y and MOVE /Y** — Suppress confirmations, intentionally hidden from help text
3. **SmartDrive BX=1234h** — Developer test hook left in shipping code
4. **SmartDrive `debug=1`** in dirtywrt.asm — Debug flag accidentally left enabled
5. **Chuck's internal pointers** — Full cache structure exposure via INT 2Fh
6. **BACKUP /F[:size]** — Source code itself says "undocumented"
7. **Multiple BUGBUG comments** — Known bugs documented but never fixed
