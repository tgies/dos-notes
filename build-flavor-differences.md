# MS-DOS 4.0 Build Flavor Technical Differences

## Overview

The DOS 4.0 source code supports two build flavors controlled by the `IBMCOPYRIGHT` flag in `src/INC/VERSION.INC`:

| Flavor | IBMVER | IBMCOPYRIGHT | System Files | Our Build Name |
|--------|--------|--------------|--------------|----------------|
| IBM PC-DOS | TRUE | TRUE | IBMBIO.COM, IBMDOS.COM | `pcdos` |
| OEM MS-DOS | TRUE | FALSE | IO.SYS, MSDOS.SYS | `msdos` (default) |

**Key Finding:** The `msdos` flavor (IBMCOPYRIGHT=FALSE) contains **more bug fixes** than the `pcdos` flavor.

---

## Category 1: File Naming (Cosmetic)

These differences only affect what filenames the utilities look for or create.

### Boot Sector (BOOT/MSBOOT.ASM:500-505)
```asm
IF IBMCOPYRIGHT
    BIO DB "IBMBIO  COM"
    DOS DB "IBMDOS  COM"
ELSE
    BIO DB "IO      SYS"
    DOS DB "MSDOS   SYS"
ENDIF
```
**Impact:** Boot sector looks for different system file names.

### SYS.COM (CMD/SYS/SYS1.ASM)
Multiple locations (lines 22-28, 33-37, 47-51, 237-242) define source/target filenames:
- **pcdos:** `\IBMBIO.COM`, `\IBMDOS.COM`
- **msdos:** `\IO.SYS`, `\MSDOS.SYS`

### FORMAT.COM (CMD/FORMAT/MSFOR.ASM:157-162)
```asm
IF IBMCOPYRIGHT
    BiosFile db "x:\IBMBIO.COM", 0
    DosFile  db "x:\IBMDOS.COM", 0
ELSE
    BiosFile db "x:\IO.SYS", 0
    DosFile  db "x:\MSDOS.SYS", 0
ENDIF
```

### BACKUP.COM (CMD/BACKUP/BACKUP.C:1918-1921)
```c
#if ! IBMCOPYRIGHT
    strcmp(dta.file_name,"IO.SYS")      == SAME ||
    strcmp(dta.file_name,"MSDOS.SYS")   == SAME ||
#endif
```
**Impact:** MS-DOS version excludes both IBM and MS system files from backup. PC-DOS only excludes IBM files (potential bug - would back up IO.SYS/MSDOS.SYS if present).

---

## Category 2: OEM Identification

### DOS Kernel OEM Number (DOS/DOSMES.ASM:32-36)
```asm
IF IBMCOPYRIGHT
    OEMNUM DB 0       ; IBM OEM number
ELSE
    OEMNUM DB 0FFH    ; Generic/MS OEM number
ENDIF
```
**Impact:** Programs can detect which OEM's DOS is running via INT 21h/AH=30h.

---

## Category 3: Actual Code Logic Differences

### 3.1 DOS Kernel: INT 24 Error Handling Bug Fix (DOS/DISK2.ASM:487-490)

```asm
IF NOT IBMCOPYRIGHT
    JC SET_ACC_ERR_DS    ; fix to take care of I24 fail
                         ; migrated from 330a - HKN
ENDIF
```

| Flavor | Behavior |
|--------|----------|
| **msdos** | Has bug fix for INT 24 (critical error) failure handling |
| **pcdos** | Missing this fix - may not properly handle disk errors |

**Severity:** HIGH - This is a kernel-level bug fix for disk error handling.

---

### 3.2 DOS Kernel: Extended Open Mode Setting (DOS/FILE.ASM:184-189, CREATE.ASM:359-361)

```asm
IF NOT IBMCOPYRIGHT
    push es    ; set up es:di to point to SFT
    push di
    push ds
    pop  es
    ; ... additional mode setting code ...
ENDIF
```

| Flavor | Behavior |
|--------|----------|
| **msdos** | Additional code to properly set modes for Extended Open (INT 21h/AX=6C00h) |
| **pcdos** | Missing extended open mode handling |

**Severity:** MEDIUM - Affects applications using Extended Open API.

---

### 3.3 FASTOPEN.EXE: EMS Memory Handling (CMD/FASTOPEN/FASTINIT.ASM)

#### EMS Function Code (lines 189-196)
```asm
IF NOT IBMCOPYRIGHT
    EMS_GET_COUNT EQU 5801H
ELSE
    EMS_GET_COUNT EQU 5800H
ENDIF
```

#### EMS Frame Buffer Size (lines 623-630)
```asm
IF IBMCOPYRIGHT
    FRAME_BUFFER DB 30h DUP(0)   ; 48 bytes
ELSE
    FRAME_BUFFER DB 100h DUP(0)  ; 256 bytes
ENDIF
```

#### Buffer Size Check (lines 2817-2821)
```asm
IF NOT IBMCOPYRIGHT
    CMP AX, 100h    ; Check against 256
ELSE
    CMP AX, 30H     ; Check against 48
ENDIF
```

| Flavor | EMS Function | Buffer Size |
|--------|--------------|-------------|
| **msdos** | 5801h | 256 bytes |
| **pcdos** | 5800h | 48 bytes |

**Severity:** MEDIUM - MS-DOS can handle more EMS page frames in FASTOPEN.

---

### 3.4 FDISK.EXE: Cylinder Calculation (CMD/FDISK/INT13.C:44-48)

```c
#if IBMCOPYRIGHT
    total_disk[i] = ((((unsigned)(regs.h.cl & 0xC0 )) & 0x00C0) << 2)+ ((unsigned)regs.h.ch) +1;
#else
    total_disk[i] = ((((unsigned)(regs.h.cl & 0xc0)) << 2) | regs.h.ch) + 1;
#endif
```

| Flavor | Code Quality |
|--------|--------------|
| **msdos** | Uses `\|` (bitwise OR) - cleaner |
| **pcdos** | Has redundant `& 0x00C0` mask, uses `+` instead of `\|` |

**Severity:** LOW - Both produce same result, but MS-DOS code is cleaner.

---

### 3.5 FDISK.EXE: Percent-to-Cylinders Conversion (CMD/FDISK/CONVERT.C:204-208)

```c
#if IBMCOPYRIGHT
    cylinders_out = ((percent_in * total_cylinders) / 100);
#else
    cylinders_out = (unsigned)((ul(percent_in) * ul(total_cylinders)) / 100);
#endif
```

| Flavor | Behavior |
|--------|----------|
| **msdos** | Uses `ul()` macro for unsigned long - prevents integer overflow |
| **pcdos** | Simple integer math - may overflow on large disks |

**Severity:** MEDIUM - PC-DOS may miscalculate partition sizes on large disks.

---

### 3.6 FDISK.EXE: Display Colors (CMD/FDISK/FDISK.H:109-118)

```c
#if IBMCOPYRIGHT
    #define HIWHITE_ON_BLUE       0x1F
    #define WHITE_ON_BLUE         0x17
    #define BLINK_HIWHITE_ON_BLUE 0x9F
#else
    #define HIWHITE_ON_BLUE       0x70
    #define WHITE_ON_BLUE         0x07
    #define BLINK_HIWHITE_ON_BLUE 0xF0
#endif
```

| Flavor | Color Scheme |
|--------|--------------|
| **pcdos** | Blue background (IBM corporate colors) |
| **msdos** | Black/white scheme |

**Severity:** COSMETIC

---

### 3.7 EXE2BIN.EXE: Wildcard Validation (CMD/EXE2BIN/E2BINIT.ASM:503-518, 565-580)

```asm
IF IBMCOPYRIGHT
    ; (empty - no validation)
ELSE
    cmp al,'*'
    je  File1_Err
    cmp al,'?'
    je  File1_Err
ENDIF
```

| Flavor | Behavior |
|--------|----------|
| **msdos** | Validates that filenames don't contain wildcards |
| **pcdos** | No wildcard validation - may produce confusing errors |

**Severity:** LOW - MS-DOS has better input validation.

---

### 3.8 COMMAND.COM: Batch File String Copy (CMD/COMMAND/TBATCH.ASM)

Multiple differences in batch file processing (lines 474-479, 514-519, 530-540, 998+):

```asm
IF IBMCOPYRIGHT
    dec di
    pop ds
ELSE
    pop ds
    jc  LineTooL    ; Check for line too long error
ENDIF
```

| Flavor | Behavior |
|--------|----------|
| **msdos** | Has "line too long" error handling in batch file processing |
| **pcdos** | Missing overflow check |

Additionally, the StrCpy function implementation differs between versions.

**Severity:** LOW-MEDIUM - MS-DOS has better error handling in batch files.

---

### 3.9 MODE.COM: Typematic Rate on Non-IBM Hardware (CMD/MODE/TYPAMAT.ASM:167-254)

```asm
if IBMCOPYRIGHT
.IF <machine_type EQ AT4> OR
.IF <machine_type EQ XT286> OR
; ... list of specific IBM machine types ...
.IF <machine_type EQ PS2Model80> THEN
endif
    ; ... typematic rate code ...
if IBMCOPYRIGHT
.ELSE
   DISPLAY Function_not_supported
   MOV noerror,false
.ENDIF
endif
```

| Flavor | Behavior |
|--------|----------|
| **pcdos** | Only sets typematic rate on specific IBM machine types |
| **msdos** | Always attempts to set typematic rate |

**Severity:** LOW - MS-DOS works on more hardware.

---

### 3.10 CHKDSK.COM: RAM Disk Detection (CMD/CHKDSK/CHKINIT.ASM:86-91)

```asm
IF IBMCOPYRIGHT
    ; (empty)
ELSE
    myramdisk db 'RDV 1.20'
ENDIF
```

| Flavor | Behavior |
|--------|----------|
| **msdos** | Has string for detecting specific RAM disk version |
| **pcdos** | Missing RAM disk detection string |

**Severity:** LOW - Affects CHKDSK behavior on RAM disks.

---

### 3.11 DOS Kernel: EMS Buffer Caching (INC/VERSION.INC:37, multiple kernel files)

```asm
BUFFERFLAG  EQU  NOT IBMCOPYRIGHT
```

The `BUFFERFLAG` conditional controls a **major MS-only feature**: EMS-based disk
buffer management. Throughout the kernel (DISP.ASM, BUF.ASM, HANDLE.ASM, EXEC.ASM,
DISK2.ASM, DISK3.ASM, CTRLC.ASM), `IF BUFFERFLAG` blocks implement disk buffer
caching in EMS expanded memory. This feature was **exclusive to MS-DOS** and not
present in IBM PC-DOS 4.00.

| Flavor | Behavior |
|--------|----------|
| **msdos** | EMS-based disk buffer caching available |
| **pcdos** | No EMS buffer caching (IBM had their own BUFFERS /X with known bugs) |

**Severity:** HIGH - Major feature difference. IBM's separate BUFFERS /X implementation
had the notorious EMS bug (assumed 6+ EMS handles, data corruption on non-IBM boards).

### 3.12 DOS Kernel: Secondary Cache Safety (DOS/MS_CODE.ASM:151-161)

```asm
if not ibmcopyright
; Here is a gross temporary fix to get around a serious design flaw in
;  the secondary cache.  The secondary cache does not check for media
;  changed (it should).  Hence, you can change disks, do an absolute
;  read, and get data from the previous disk.  To get around this,
;  we just won't use the secondary cache for absolute disk reads.
;                                                      -mw 8/5/88
```

| Flavor | Behavior |
|--------|----------|
| **msdos** | Disables secondary cache for absolute disk reads (prevents data from wrong disk) |
| **pcdos** | Secondary cache bug present - can return stale data after disk change |

**Severity:** HIGH - Data integrity issue. Dated August 5, 1988 (3 days after IBM's release).

---

## Summary Table

| Component | Difference Type | Severity | Better Version |
|-----------|-----------------|----------|----------------|
| Boot Sector | File naming | N/A | Same |
| SYS.COM | File naming | N/A | Same |
| FORMAT.COM | File naming | N/A | Same |
| BACKUP.COM | File exclusion | Low | msdos |
| DOS Kernel OEM | Identification | N/A | Same |
| DOS Kernel I24 | Bug fix | **HIGH** | **msdos** |
| DOS Kernel ExtOpen | Feature | Medium | msdos |
| FASTOPEN EMS | Buffer size | Medium | msdos |
| FDISK cylinder | Code quality | Low | msdos |
| FDISK percent | Overflow fix | Medium | msdos |
| FDISK colors | Cosmetic | N/A | Preference |
| EXE2BIN | Validation | Low | msdos |
| COMMAND.COM | Error handling | Low-Med | msdos |
| MODE.COM | Hardware compat | Low | msdos |
| CHKDSK | RAM disk | Low | msdos |
| DOS Kernel EMS | Buffer caching | **HIGH** | **msdos** |
| DOS Kernel Cache | Media change fix | **HIGH** | **msdos** |

---

## Conclusion

The **msdos** flavor (IBMCOPYRIGHT=FALSE) is objectively better maintained:

1. **Contains critical bug fix** for INT 24 error handling in the kernel
2. **Better integer overflow protection** in FDISK
3. **Larger EMS buffers** in FASTOPEN
4. **Better input validation** in EXE2BIN
5. **Better error handling** in batch file processing
6. **Broader hardware compatibility** in MODE

The **pcdos** flavor represents the code that IBM shipped, which required their approval process. Microsoft's OEM version received fixes more quickly.

**Recommendation:** Use `msdos` flavor for actual use. Use `pcdos` flavor only for historical accuracy when recreating IBM PC-DOS specifically.

---

## Message Branding: Why PC-DOS Still Says "MS-DOS"

### The Problem

Both flavors display "MS-DOS" in the VER command output, even when built with `--flavor=pcdos`. This is because the message strings are **not controlled by IBMCOPYRIGHT**.

### How the Message System Works

```
┌─────────────────┐     ┌─────────────────┐
│  COMMAND.SKL    │     │   usa-ms.msg    │
│  (skeleton)     │     │  (message DB)   │
│  Msg templates  │     │  Actual strings │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────────┬───────────┘
                     ▼
              ┌─────────────┐
              │  BUILDMSG   │  (build tool)
              └──────┬──────┘
                     ▼
              ┌─────────────┐
              │  .CTL file  │  (compiled messages)
              └─────────────┘
```

1. **`.SKL` files** (e.g., `COMMAND.SKL`) define message numbers and default text
2. **`usa-ms.msg`** is the master message database that overrides SKL defaults
3. **`BUILDMSG.EXE`** combines them at build time
4. **`BUILDIDX.EXE`** creates an index for runtime lookup

### What's in the Message Files

**COMMAND.SKL line 178** (default):
```
:def 1040 "Microsoft DOS             Version %1.%2"
```

**usa-ms.msg line 205** (override - this is what's used):
```
1040 U 0000 "MS-DOS Version %1.%2"
```

**usa-ms.msg line 175** (startup banner):
```
0464 U 0001 CR,LF,CR,LF,"Microsoft(R) MS-DOS(R) Version 4.00",CR,LF
             "(C)Copyright Microsoft Corp 1981-1988",CR,LF
```

### Why IBM Branding Is Missing

The open-sourced code only includes `usa-ms.msg` (USA/English + Microsoft branding). IBM would have had their own message file, likely named something like `usa-ibm.msg`, containing:

```
1040 U 0000 "IBM PC DOS Version %1.%2"
0464 U 0001 CR,LF,CR,LF,"IBM Personal Computer DOS Version 4.00",CR,LF
             "(C)Copyright IBM Corp 1981-1988",CR,LF
```

This file was **not open-sourced**, so our `pcdos` build produces binaries with IBM system file names but Microsoft branding in the UI.

### Summary

| Build Flavor | System Files | VER Output | Startup Banner |
|--------------|--------------|------------|----------------|
| `msdos` | IO.SYS, MSDOS.SYS | "MS-DOS Version 4.00" | "Microsoft(R) MS-DOS(R)" |
| `pcdos` | IBMBIO.COM, IBMDOS.COM | "MS-DOS Version 4.00" | "Microsoft(R) MS-DOS(R)" |
| Real IBM PC-DOS | IBMBIO.COM, IBMDOS.COM | "IBM PC DOS Version 4.00" | "IBM Personal Computer DOS" |

