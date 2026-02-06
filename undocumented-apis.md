# Undocumented DOS APIs & Internal System Calls

## Overview

Both DOS 4.0 and DOS 6 contain extensive undocumented APIs marked with "CAVEAT PROGRAMMER"
warnings in the source code. These were essential for the third-party ecosystem (Norton Utilities,
PC Tools, Stacker, TSR programs) but were never part of the official API contract.

---

## INT 21h — "CAVEAT PROGRAMMER" Functions (DOS 4.0)

These functions are marked with ornate warning banners in the source:
```
;----+----+----+----+----+----+----+----+----+----+----+----+----+----+----;
;          C  A  V  E  A  T     P  R  O  G  R  A  M  M  E  R             ;
;          THIS FUNCTION CALL IS SUBJECT TO CHANGE!!!                     ;
;----+----+----+----+----+----+----+----+----+----+----+----+----+----+----;
```

### AH=1Bh — $SLEAZEFUNC (Get FAT ID Byte)

- **File:** `v4.0/src/DOS/MISC.ASM:66-118`
- **Dispatch:** `v4.0/src/DOS/MS_TABLE.ASM:272`
- **Input:** None (uses default drive)
- **Output:**
  - DS:BX → pointer to FAT ID byte
  - DX = total allocation units
  - CX = sector size
  - AL = sectors per allocation unit
- **Warning:** *"GOD help anyone who tries to do ANYTHING except READ this ONE byte."*
- **Caveat:** NOT re-entrant — uses fixed memory cell `FATBYTE`
- **Purpose:** CP/M compatibility holdover; returns raw FAT media descriptor byte

### AH=1Ch — $SLEAZEFUNCDL (Get FAT ID Byte, Specific Drive)

- **File:** `v4.0/src/DOS/MISC.ASM:89-115`
- **Dispatch:** `v4.0/src/DOS/MS_TABLE.ASM:273`
- **Input:** DL = drive number (0=default, 1=A, 2=B, ...)
- **Output:** Same as AH=1Bh
- **Same caveats as 1Bh**

### AH=1Fh — Get_Default_DPB (Get Drive Parameter Block)

- **File:** `v4.0/src/DOS/MISC.ASM:173-200`
- **Output:** DS:BX → DPB for default drive; AL=0 if OK, -1 if bad drive
- **Purpose:** Direct access to low-level drive geometry/FAT info

### AH=32h — Get_DPB (Specific Drive)

- **File:** `v4.0/src/DOS/MISC.ASM:178-200`
- **Input:** DL = drive number
- **Output:** Same as AH=1Fh

### AH=34h — Get_InDOS_Flag

- **File:** `v4.0/src/DOS/MISC.ASM:131-138`
- **Output:** ES:BX → INDOS flag (non-zero = DOS in critical section)
- **Purpose:** Critical for TSR programming — interrupt handlers check this flag to determine if it's safe to call DOS. Foundation of all resident utility software.

### AH=50h — Set_Current_PDB (Set PSP)

- **File:** `v4.0/src/DOS/DISP.ASM:153-157`
- **Input:** BX = new PDB/PSP segment address
- **Output:** None
- **Note:** Dispatched with interrupts DISABLED for atomic operation
- **Purpose:** Process context switching — used by task switchers and debuggers

### AH=51h — Get_Current_PDB (Get PSP)

- **File:** `v4.0/src/DOS/DISP.ASM:163-167`
- **Output:** BX = current PDB/PSP segment
- **Note:** Also dispatched with interrupts disabled

### AH=52h — Get_In_Vars (The "List of Lists")

- **File:** `v4.0/src/DOS/MISC.ASM:150-155`
- **Output:** ES:BX → SYSINITVAR structure
- **Warning:** *"version dependent and is subject to change without notice"*
- **Purpose:** The single most powerful undocumented call. Returns pointer to ALL DOS internal data:
  - Buffer chain pointers
  - System File Table (SFT)
  - Current Directory Structure (CDS) array
  - Device driver chain
  - Disk buffer info
  - MCB chain start
- **Significance:** This is what Norton Utilities, PC Tools, and every disk utility used to access DOS internals

### AH=53h — SetDPB (Set Drive Parameter Block)

- **File:** `v4.0/src/DOS/DISP.ASM:364-365`
- **Purpose:** Modify low-level drive geometry — extremely dangerous

### AH=55h — Dup_PDB (Clone Process Descriptor)

- **File:** `v4.0/src/DOS/DISP.ASM:373-376`
- **Purpose:** Create a new Process Data Block — used internally by EXEC

### AH=64h — Set_Printer_Flag

- **File:** `v4.0/src/DOS/DISP.ASM:176-180`
- **Input:** AL = flag value
- **Purpose:** Internal printer control flag

---

## INT 21h AH=33h — Undocumented Subfunctions (DOS 3.4+)

- **File:** `v4.0/src/DOS/DISP.ASM:73-136`
- **Documented:** AL=0 (get ^C status), AL=1 (set ^C), AL=2 (get+set ^C)
- **Undocumented extensions:**

| AL | Function | Notes |
|----|----------|-------|
| 3 | Get CPSW state → DL | Code Page Switch state (Korean DBCS) |
| 4 | Set CPSW state from DL | Interim mode character handling |
| 5 | Get boot drive → DL | Returns drive letter DOS booted from |

---

## INT 21h AH=5Dh — ServerCall (Internal Multiplexer)

- **File:** `v4.0/src/DOS/SRVCALL.ASM:56-136`
- **Input:** DS:DX → DPL (DOS Private List) structure
- **Subfunctions (AL):**

| AL | Function | Purpose |
|----|----------|---------|
| 0 | Server DOS call | Execute DOS call from server context |
| 1 | Commit all files | Flush all file buffers |
| 2 | Close file by name | Requires SHARE loaded; DS:DX in DPL |
| 3 | Close all files for UID | Close user's files |
| 4 | Close all files for UID/PID | Close specific process files |
| 5 | Get open file list entry | BX=file idx, CX=user idx → ES:DI→name |
| 6 | Get DOS data area | DS:SI→start, CX=swap size, DX=always-swap |
| 7 | Get truncate flag | Internal flag state |
| 8 | Set truncate flag | Internal flag state |
| 9 | Close all spool files | Print spooler cleanup |
| 10 | SetExtendedError | Set extended error info |
| 11 | Get DOS data area (4.0) | Returns swap table pointer (DOS 4.0 new) |

---

## The OEM Extension Backdoor — INT 21h AH=F8h

- **File:** `v4.0/src/DOS/DISP.ASM:550-576`
- **Condition:** Only in non-IBM builds (`IF NOT IBM`)
- **Variable:** `v4.0/src/DOS/MSCONST.ASM:54-57` — `OEM_HANDLER DD -1`

### How It Works

1. **Install handler:** Call `INT 21h, AH=F8h` with `DS:DX` = handler address
   - Stores handler in `OEM_HANDLER` DWORD
2. **Call handler:** Any `INT 21h` with `AH=F9h through FFh` → jumps to handler
3. **Uninitialized:** If handler not set (`OEM_HANDLER = -1`), returns "bad call"

```asm
$SET_OEM_HANDLER:
    JNE     DO_OEM_FUNC             ; If above F8, try to jump to handler
    MOV     WORD PTR [OEM_HANDLER],DX
    MOV     WORD PTR [OEM_HANDLER+2],DS
    IRET

DO_OEM_FUNC:
    CMP     WORD PTR [OEM_HANDLER],-1
    JNZ     OEM_JMP
    JMP     BADCALL

OEM_JMP:
    JMP     [OEM_HANDLER]
```

### Significance

This is a plugin system for DOS — OEM vendors (Compaq, Dell, HP) could extend DOS with 7 proprietary system calls (F9h-FFh) without forking the kernel. The variable is declared even in IBM builds but the code path is compiled out.

Note from `MSCONST.ASM:54`: *"We include the decl of OEM_HANDLER in IBM DOS even though it is not used."*

---

## Unpublished IOCTL Codes (DOS 4.0 BIOS)

- **File:** `v4.0/src/BIOS/MSIOCTL.INC`

### Disk IOCTL Subfunctions (Category 8)

| Code | Name | Direction | Status |
|------|------|-----------|--------|
| 60h | GET_DEVICE_PARAMETERS | Read | Documented |
| 61h | READ_TRACK | Read | Documented |
| 62h | VERIFY_TRACK | Read | Documented |
| 66h | GET_MEDIA_ID | Read | Changed from 63h in DOS 4 (AN003) |
| **67h** | **GET_ACCESS_FLAG** | **Read** | **UNPUBLISHED** (AN002/AN003) |
| 40h | SET_DEVICE_PARAMETERS | Write | Documented |
| 41h | WRITE_TRACK | Write | Documented |
| 42h | FORMAT_TRACK | Write | Documented |
| 46h | SET_MEDIA_ID | Write | Changed from 43h in DOS 4 (AN003) |
| **47h** | **SET_ACCESS_FLAG** | **Write** | **UNPUBLISHED** (AN002/AN003) |

### GetAccessFlag (67h) — Lines 1316-1337

- **Purpose:** Read `UNFORMATTED_MEDIA` bit from BDS FLAGS
- **Output:** `DAC_ACCESS_FLAG` = 0 (I/O not allowed) or 1 (I/O allowed)
- **Restriction:** "Only for Hard file"
- **Marked:** *"Unpublished function"*

### SetAccessFlag (47h) — Lines 1340-1360

- **Purpose:** Set/Reset `UNFORMATTED_MEDIA` bit
- **Input:** `DAC_ACCESS_FLAG` = 0 (disallow) or 1 (allow)
- **Marked:** *"Unpublished function"*

### Media ID Structures (DOS 4.0 New — AN000)

```asm
A_MEDIA_ID_INFO STRUC
    MI_LEVEL    DW  0               ; Info level
    MI_SERIAL   DD  ?               ; Serial number
    MI_LABEL    DB  11 DUP (' ')    ; Volume label
    MI_SYSTEM   DB  8 DUP (' ')     ; File system type ("FAT12   ", "FAT16   ")
A_MEDIA_ID_INFO ENDS
```

---

## INT 2Fh Multiplex — Internal DOS Function Table

- **File:** `v4.0/src/DOS/MS_TABLE.ASM:416-471`
- **Table:** `DOSTable` — 48 internal functions

| Index | Function | Purpose |
|-------|----------|---------|
| 00h | DOSInstall | Install check |
| 01h | DOS_CLOSE | Close file internal |
| 02h | RECSET | Record set |
| 03h | DOSGetGroup | Get DOSGROUP segment |
| 04h | PATHCHRCMP | Path character compare |
| 05h | OUTT | Output primitive |
| 06h | NET_I24_ENTRY | Network INT 24 entry |
| 07h | PLACEBUF | Place buffer |
| 08h | FREE_SFT | Free System File Table entry |
| 09h | BUFWRITE | Buffer write |
| 0Ah | SHARE_VIOLATION | Share violation handler |
| 0Bh | SHARE_ERROR | Share error handler |
| 0Ch | SET_SFT_MODE | Set SFT mode |
| 0Dh | DATE16 | 16-bit date conversion |
| 0Fh | SCANPLACE | Scan+place |
| 11h | StrCpy | String copy |
| 12h | StrLen | String length |
| 13h | Ucase | Uppercase conversion |
| 14h | POINTCOMP | Pointer compare |
| 15h | CHECKFLUSH | Check and flush buffers |
| 16h | SFFromSFN | SFT from SFN |
| 17h | GetCDSFromDrv | CDS from drive letter |
| 18h | Get_User_Stack | User stack pointer |
| 19h | GetThisDrv | Current drive info |
| 1Ah | DriveFromText | Parse drive from text |
| 1Bh | SETYEAR | Set year |
| 1Ch | DSUM | Data sum |
| 1Dh | DSLIDE | Data slide |
| 1Eh | StrCmp | String compare |
| 1Fh | InitCDS | Init Current Directory Structure |
| 20h | pJFNFromHandle | JFN from handle |
| 21h | $NameTrans | Name translation |
| 22h | CAL_LK | Calculate lock |
| 23h | DEVNAME | Device name |
| 25h | DStrLen | Double-byte string length |
| 26h | NLS_OPEN | NLS open (DOS 3.3) |
| 27h | $CLOSE | Close (DOS 3.3) |
| 28h | NLS_LSEEK | NLS seek (DOS 3.3) |
| 29h | $READ | Read (DOS 3.3) |
| 2Ah | FastInit | Fast init (DOS 3.4) |
| 2Bh | NLS_IOCTL | NLS IOCTL (DOS 3.3) |
| 2Ch | GetDevList | Get device list (DOS 3.3) |
| 2Dh | NLS_GETEXT | NLS get extended (DOS 3.3) |
| 2Eh | MSG_RETRIEVAL | Message retrieval (DOS 4.0 new) |
| 2Fh | Fake_Version | Fake version (DOS 4.0 new) |

---

## DOS Internal Flags & Structures

From `v4.0/src/DOS/MS_TABLE.ASM:684-705`:

| Flag | Purpose |
|------|---------|
| DISK_FULL | Disk full condition |
| EXTOPEN_ON | Extended open active |
| DOS34_FLAG | Common DOS 3.4 flag |
| ACT_PAGE | Active EMS page (-1 = invalidated) |
| CPSWFLAG/CPSWSAVE | Code page switch flags (DBCS) |
| FETCHI_TAG | IBMDOS compatibility tag (must equal 22642) |
| SC_STATUS | Secondary cache status |
| SC_FLAG | Secondary cache flag |
| IFS_DRIVER_ERR | IFS driver error code |
| SEQ_SECTOR | Last sector read (-1 = init) |
| XA_CONDITION | Extended Attributes condition |

### Critical Section Flags (DISP.ASM:14-25)

| Flag | Protects |
|------|----------|
| critDisk | Buffer cache manipulation |
| critDevice | All device drivers |
| critMem | Memory allocation |
| critNet | Network operations |

---

## SmartDrive INT 2Fh Interface (DOS 6 Astro)

- **File:** `astro/dev/smartdrv/int2f.asm`
- **Multiplex:** INT 2Fh, AX=4A10h

### Documented Functions

| BX | Name | Purpose |
|----|------|---------|
| BAMBI_GET_STATS | Get statistics | Returns hit/miss counts, dirty elements, version |
| BAMBI_COMMIT_ALL | Commit all | Flush dirty cache to disk |
| BAMBI_REINITIALIZE | Reinitialize | Reset cache (workaround for QEMM bug) |
| BAMBI_CACHE_DRIVE | Cache drive | Enable/disable caching per drive |
| BAMBI_GET_INFO | Get info | Cache block size, queue length, element count |

### Undocumented Functions

| BX | Name | Purpose | Source |
|----|------|---------|--------|
| BAMBI_INTERNAL_POINTERS | Return internals | Pointers to valid index, dirty index, element IDs, queue length | `int2f.asm:458-486` — "Chuck's debug stuff!!!" |
| 1234h | Display message | Triggers `warning_pop_up(al=3)` — displays popup dialog | `int2f.asm:462-527` — Magic number test hook |
| BAMBI_GET_ORIGINAL_DD_HEADER | Get original DD | Returns pointer to original device driver header | `int2f.asm:456-457` |

The `if 1` conditional on lines 458 and 466 means this debug code was compiled into the shipping product.

---

## User Stack Structure (Direct Register Manipulation)

```asm
user_environ STRUC
    user_AX     DW  ?
    user_BX     DW  ?
    user_CX     DW  ?
    user_DX     DW  ?
    user_SI     DW  ?
    user_DI     DW  ?
    user_BP     DW  ?
    user_DS     DW  ?
    user_ES     DW  ?
    user_IP     DW  ?
    user_CS     DW  ?
    user_F      DW  ?   ; Flags
user_environ ENDS
```

Internal functions use `get_user_stack` (DISP.ASM:545-548) to get DS:SI pointing to this structure, then directly modify user registers before IRET. Used by SLEAZEFUNC, Get_INDOS_Flag, and other backdoor calls.
