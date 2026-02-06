# DOS 4.0 Hidden Features, Debug Modes & Dead Code

## Overview

MS-DOS 4.0 source code (`v4.0/src/`) contains debug modes, undocumented CONFIG.SYS
directives, disabled features, vestigial hardware code, and colorful developer commentary
spanning nearly a decade of DOS development (2.0→4.0).

---

## Debug Modes & Developer Tools

### DEBUG.COM — ZTRACE / "P" Command

- **Files:** `CMD/DEBUG/DEBCOM3.ASM:54-164`, `DEBDATA.ASM:148`, `DEBUG.ASM:1018`
- **Command:** Type `P` at the DEBUG prompt
- **Dispatch:** `DEBUG.ASM:1018` — `DW ZTRACE ; P`

**What it does:** Advanced trace mode that automatically **steps OVER** control-flow instructions:
- CALL instructions (near and far)
- INT instructions
- LOOP, LOOPE, LOOPNE
- REP-prefixed string instructions

Sets the `FZTRACE` flag to -1, analyzes opcodes to determine instruction length, and sets a breakpoint at the next instruction after the control-flow operation completes. This is equivalent to a modern debugger's "Step Over" (vs "Step Into").

**Credit:** Developer "zibo" (DEBUG.ASM:62 revision comment)

### DEBUG.COM — Parity Error Detection

- **Files:** `CMD/DEBUG/DEBERR.ASM:80-133`, `DEBUG.ASM:286,568-574`, `DEBDATA.ASM:154`
- **Functions:** `TRAPPARITY` (DEBERR.ASM:80), `RELEASEPARITY` (DEBERR.ASM:109)

**What it does:** DEBUG captures NMI parity error interrupts, logs them, and displays:
*"Parity error or nonexistent memory error detected"*

**Credit:** Developer "zibo" (DEBUG.ASM:64)

### COMMAND.COM — EMGDEBUG Flag

- **File:** `CMD/COMMAND/COMEQU.ASM:6`
- **Declaration:** `EMGDEBUG = FALSE`
- **Status:** Compile-time flag, always disabled in retail builds
- **Purpose:** Emergency debug mode for COMMAND.COM — when TRUE, enables additional diagnostic output during shell initialization and command processing

### MEMM.EXE — Kernel Debugger Interface

- **Files:** `MEMM/MEMM/INITDEB.ASM:81-173`, `MEMM/MEMM/MAKEFILE:60`
- **Compile flag:** `-DNoBugMode` in AFLAGS (MAKEFILE:60)
- **Status:** Compiled OUT of retail builds

**Debug infrastructure:**
```asm
InitDeb STRUC
    ; Real mode code/data segments
    ; Protected mode code/data selectors
    ; GDT and IDT aliases
    ; Break flag and COM port settings
InitDeb ENDS
```

- `_Debug_Entry` — far call entry point (line 166)
- `AL=FFh` — break to debugger on initialization
- `AL=00h` — no break
- Supports COM port debugging (serial debugger)

### DOS Kernel DEBUG Flag

- **Files:** `INC/DOSSYM.INC:8-9`, `INC/SMDOSSYM.INC:8-9`
- **Setting:** `DEBUG = FALSE`
- **Effect:** When TRUE, includes `BugLev`/`BugTyp` variables and `bugtyp.asm` for kernel-level debugging (see `DOS/MISC.ASM:60-64`)

---

## Undocumented CONFIG.SYS Directives

### MULTITRACK=ON|OFF

- **Files:** `BIOS/SYSCONF.ASM:522,1117-1121`, `BIOS/MSDISK.ASM:11`
- **Syntax:** `MULTITRACK=ON` or `MULTITRACK=OFF`
- **Purpose:** Enables/disables multi-track disk I/O — when ON, BIOS can read/write across track boundaries in a single INT 13h call, improving disk performance on compatible controllers
- **Flags:**
  - `MULTRK_OFF1 = 00000000B` — initial state (no command given)
  - `MULTRK_OFF2 = 00000001B` — user explicitly specified OFF
- **Annotation:** AN002, d24 (indicates this was a design note, not public documentation)

### SWITCHES=/F/H/S/T/D/I/C/N

- **File:** `BIOS/SYSINIT2.ASM:132,163,191,201,238-285,509,1608-1615`
- **Syntax:** `SWITCHES=/X` where X is one of `F H S T D I C N`
- **Switch list:** Defined at line 1608 as `switchlist "FHSTDICN"` (8 valid switches)
- **Known flags:**
  - `flagec35` (00000100B) — Electrically compatible 3.5" disk drive
  - `flagdrive` (00001000B) — Drive specification flag
  - `flagseclim` — Sector limit flag (line 351)
  - `flagheads` — Heads flag (line 356)

### STACKS=0,0 (Special Case)

- **File:** `BIOS/SYSCONF.ASM:476-480,2094-2177`
- **Syntax:** `STACKS=n,m` where n=8-64, m=32-512
- **Special case:** `STACKS=0,0` — completely disables DOS hardware interrupt stack switching
- **Purpose:** Some applications/TSRs conflicted with DOS stack switching; this was the escape hatch

### IFS= (Installable File System)

- **File:** `BIOS/SYSCONF.ASM` (implicit references)
- **Purpose:** Support for installable file systems — planned but never fully shipped in retail DOS 4.0

---

## Vestigial & Dead Code

### The Intentional Timer Bug — "IBM IS FUNDAMENTALY BRAIN DAMAGED"

- **File:** `BIOS/MSBIO1.ASM:82-89`
- **Developer:** ARR (Aaron Reynolds)
- **Date:** 7/13/1983 (REV 2.15)

```
;   REV 2.15  7/13/83 ARR BECAUSE IBM IS FUNDAMENTALY BRAIN DAMAGED, AND
;       BASCOM IS RUDE ABOUT THE 1CH TIMER INTERRUPT, THE TIMER
;       HANDLER HAS TO GO BACK OUT!!!!!  IBM SEEMS UNWILLING TO
;       BELIEVE THE PROBLEM IS WITH THE BASCOM RUNTIME, NOT THE
;       DOS.  THEY HAVE EVEN BEEN GIVEN A PATCH FOR BASCOM!!!!!
;       THE CORRECT CODE IS COMMENTED OUT AND HAS AN ARR 2.15
;       ANNOTATION.  THIS MEANS THE BIOS WILL GO BACK TO THE
;       MULTIPLE ROLL OVER BUG.
```

**Story:** MS fixed a timer rollover bug, but IBM's BASCOM runtime was hooking INT 1Ch improperly. IBM refused to fix BASCOM (even after being given a patch!), so MS had to revert the fix, knowingly reintroducing the timer rollover bug. The correct code remains commented out as a monument to this conflict.

### Other BIOS Revision History Highlights (MSBIO1.ASM)

| Rev | Date | Dev | Change |
|-----|------|-----|--------|
| 2.20 | 8/5/83 | ARR | Half-height drive support — IBM hardware change, head settle workaround |
| 2.21 | 8/11/83 | MZ | IBM wants write-with-verify to use head settle 0 |
| 2.25 | 6/20/83 | MJB | 96TPI and "Salmon" (codename) support |
| 2.30 | 6/27/83 | MJB | Real-time clock added |
| 2.40 | 7/8/83 | MJB | Volume-ID checking, INT 2F macros |
| 2.42 | 11/3/83 | ARR | Disk open/close/format hooked out to shrink BIOS |
| 2.43 | 12/6/83 | MZ | 16-bit FAT detection in boot sectors |

### PCjr ROM Cartridge Support (HALO)

- **Files:** `INC/MSHALO.ASM`, `DOS/MSHALO.ASM`
- **Condition:** Only when `IBMVER=TRUE`
- **Called from:** COMMAND.COM (TCODE.ASM) via `ROM_SCAN`/`ROM_EXEC`

**How it works:**
1. Checks for PCjr signature byte `0FDh` at memory `0FFFFE:0`
2. Scans ROM address space `C0000h`–`F0000h` on 2KB boundaries
3. Looks for ROM header signature `AA55h`
4. Matches user-typed commands against ROM cartridge name tables
5. If match found, jumps to cartridge entry point

**Effect:** On PCjr, typing `BASIC` would execute from ROM cartridge before looking on disk. This code was vestigial by DOS 4.0 (1988) since the PCjr was discontinued in 1985.

### RAMDRIVE Memory Testing (Removed)

- **File:** `DEV/RAMDRIVE/RAMDRIVE.ASM:5624-5746`
- **Marker:** `; **START OF CODE REMOVED` through `;***END OF CODE REMOVED***`
- **Purpose:** Memory testing and validation for RAM drives — disabled to save space or avoid compatibility issues
- **Related:** Line 5953 — "PARITY CHECKING DISABLED"

### RAMDRIVE INT 9 Handler (Removed)

- **File:** `DEV/RAMDRIVE/RAMDRIVE.ASM:157`
- **Comment:** *"removed int 9 trapping"*
- **Purpose:** Keyboard interrupt handler was removed (likely conflicted with other handlers)

### CheckThisDevice (Dead Code)

- **File:** `DOS/DIR2.ASM:1065`
- **Wrapped in:** `if FALSE` — never executed
- **Purpose:** Device/file name resolution function, made obsolete by other improvements

### XMA Emulation (Deleted)

- **File:** `DEV/XMAEM/INDEXMA.ASM:650,699`
- **Comment:** *"XMA 1 emulation code deleted"*
- **Purpose:** Old XMA (eXtended Memory Architecture) support replaced by newer XMS

### MEMM ON/OFF/AUTO (Removed)

- **File:** `MEMM/MEMM/INIT.ASM:38`
- **Comment:** *"removed ON/OFF/AUTO support"*
- **Purpose:** Runtime toggling of memory manager — feature was cut

### IFS (Installable File Systems)

- **File:** `CMD/IFSFUNC/IFSHAND.ASM:756` and CONFIG.SYS parsing
- **Status:** Stubs present, never fully implemented for retail
- **Comment:** *"This call is driven by the new INT 21H call 57H. *** REMOVED"*

### DOS Pipes Stub

- **File:** `H/DOSCALLS.H:117`
- **Declaration:** `extern unsigned far pascal DOSMAKEPIPE`
- **Status:** Declared but stub only — full pipe support wasn't in DOS 4.0

### ALTVECT Build Switch

- **File:** `BIOS/SYSINIT2.ASM:42`
- **Setting:** `ALTVECT = FALSE`
- **Purpose:** Alternate interrupt vector initialization — disabled in standard build

---

## Hardware Workarounds & Ghost Code

### Intel 386 B0 Stepping Bug Workarounds

- **Files:** `DEV/SMARTDRV/SMARTDRV.ASM:5015`, `DEV/RAMDRIVE/RAMDRIVE.ASM:2514`
- **Comment:** *"386 were removed in the B1 stepping. When executed on the B1, INSERT..."*
- **Status:** Workaround code for early 386 processor bugs, never cleaned up after B1 stepping fixed the issues

### COMPAQ Hardware Workarounds

- **File:** `DEV/SMARTDRV/SMARTDRV.ASM:1878`
- **Comment:** *"the next two instructions were removed because of the way Compaq..."*
- **Status:** COMPAQ-specific compatibility code, removed after fixing underlying issue

### Olivetti Ghost Flag

- **File:** `DEV/SMARTDRV/SMARTDRV.ASM:470`
- **Comment:** *"The Olivetti guys have defined a flag of their own. This should be removed"*
- **Status:** Never removed. Shipped, presumably. Vestigial Olivetti PC support.

---

## Developer Comments & Hacks

### "HACK! HACK! HACK!" — SmartDrive Cache Allocation

- **File:** `DEV/SMARTDRV/SMARTDRV.ASM:1285`
- **Comment:** *"HACK! HACK! HACK! Smartdrv should only allocate as many pages as it needs."*
- **Issue:** Over-allocates EMS pages — known problem, never fixed

### "KLUDGE ALERT!!" — FC.C goto Usage

- **File:** `CMD/FC/FC.C:584,622,640`
- **Comment:** *"KLUDGE ALERT!! GOTO USED"* (appears THREE times)
- **Context:** File comparison utility apologizing for using `goto` in C (considered bad practice even in the late 1980s)

### "Sort of HACKey" — COMMAND.COM 2.0 Code

- **File:** `CMD/COMMAND/COMMAND1.ASM:82`
- **Comment:** *"Some code for new 2.0 DOS, sort of HACKey. Not enough time to..."*
- **Status:** Quick temporary fix from the DOS 2.0 era that survived through DOS 4.0

### Disk BIOS "Hack" for Compatibility

- **File:** `BIOS/MSDISK.ASM:676`
- **Comment:** *"IN DOS. IN ORDER TO SUPPORT THEM, WE HAVE TO INTRODUCE A 'HACK' THAT WILL..."*

---

## Keyboard Driver Dead Code

- **Files:** `DEV/KEYBOARD/KDFUK.ASM:1`, `KDFUK168.ASM`, `KDFFR.ASM:3`
- **Convention:** *"CODE to be deleted has a double ;; followed by actual asm code....****"*
- **Status:** Systematic marking of code for deletion across ALL keyboard driver variants (UK, UK-168, French, etc.) — marked but never actually removed from the source tree

---

## Conditional Compilation Summary

| Flag | File | Default | Purpose |
|------|------|---------|---------|
| `IBMVER` | `INC/VERSION.INC:34` | TRUE | IBM PC hardware-specific code |
| `IBMCOPYRIGHT` | `INC/VERSION.INC:35` | FALSE | IBM branding vs MS branding |
| `BUFFERFLAG` | `INC/VERSION.INC:37` | `NOT IBMCOPYRIGHT` | Extra buffering for non-IBM |
| `IBMJAPVER` | `INC/VERSION.INC:71` | FALSE | Japanese IBM version |
| `INTERNAT` | `INC/VERSION.INC:76` | FALSE | DBCS/SBCS international |
| `KANJI` | `INC/VERSION.INC:82` | FALSE | Kanji character support |
| `KOREA` | `INC/VERSION.INC:95` | FALSE | Korean character support |
| `JAPAN` | `INC/VERSION.INC:99` | FALSE | Japanese character support |
| `DEBUG` | `INC/DOSSYM.INC:8` | FALSE | Kernel debug output |
| `EMGDEBUG` | `CMD/COMMAND/COMEQU.ASM:6` | FALSE | COMMAND.COM emergency debug |
| `ALTVECT` | `BIOS/SYSINIT2.ASM:42` | FALSE | Alternate vector init |
| `NoBugMode` | `MEMM/MEMM/MAKEFILE:60` | Defined | Disables kernel debugger |

---

## DOS Shell (SELECT) Removed Features

- **File:** `SELECT/SELECT0.ASM`
- **Revision annotations showing removed features:**

| Annotation | Date | Change |
|------------|------|--------|
| A016 - D445 | 2/4/88 | Removed /REF from DOSSHELL.BAT |
| A018 - D418 | 2/4/88 | Removed default /TEXT option on Model 25 |
| A020 - D438 | 2/5/88 | Removed EXT_DISKETTE_SCREEN and EXT_DISK_PARMS_SCREEN |
| A037 - P3672 | 3/1/88 | Removed /MSG from SHELL= in CONFIG.SYS |
| A050 - P4020 | 3/25/88 | Removed quote marks and spaces between "1 MB" |
| A071 - P4428 | 4/22/88 | Removed /PRE parameter from DOSSHELL.BAT |
