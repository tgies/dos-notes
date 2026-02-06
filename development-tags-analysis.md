# DOS 4.0 Development Tags & Change Management Analysis

The DOS 4.0 source code contains a frozen development database embedded in comments:
four distinct tag systems that together reconstruct version control, bug tracking,
feature tracking, and per-line attribution from IBM Boca Raton's 1986-1988 workflow.

## Tag Systems Overview

### 1. SCCSID — Source Code Control System

**Format:** `SCCSID = @(#)filename version date`
**Example:** `SCCSID = @(#)buffer.asm 1.1 85/04/09`

- From Bell Labs SCCS, earliest VCS (1972), used on Unix/Xenix
- `@(#)` is a magic marker for the Unix `what` command (searches binaries)
- 283 tags across 203 files
- C files embed as `static char *SCCSID = "..."` (survives into binary)
- ASM files use comment-only form

**Date distribution (by year-month):**
```
1985/04: 109 files  ← Mass check-in, DOS 3.x baseline import
1985/05:  43 files  ← COMMAND.COM and utilities
1985/06:   7
1985/07:  34        ← Mid-year revision wave
1985/08:  10
1985/09:  19
1985/10:  20
1985/11:   3
1985/12:   2
1986/*:    9         ← Scattered updates (MAPPER, BIOS)
1987/04-05: 23      ← IFS subsystem (new for DOS 4.0)
1987/08:   1
1988/02:   1
```

**Key observation:** The April 9-10, 1985 cluster (109 files) represents the DOS 3.x
baseline being checked into SCCS. Files dated 1987 are new DOS 4.0 subsystems (IFS,
SMIFSSYM, etc). This gives us code genealogy — which files predate 4.0 vs. were
written for it.

### 2. PTM — Problem Tracking Memorandum (IBM Bug Tracker)

**Format varies:** `PTM P1234`, `PTM0011`, `ptm p097`, `PTMP3955`
**376 unique PTM numbers** across 97 files, ranging from P5 to P5031

IBM's internal bug tracking system — the 1980s Jira. Each PTM represents a specific
defect report filed during testing.

**Normalization challenge:** PTM numbers appear in many formats:
- `PTM P265` (most common — "P" prefix + number)
- `PTM0011` (zero-padded, no P)
- `ptm p097` (lowercase)
- `PTM0000045` (heavily zero-padded, APPEND style)
- `PTMP3955` (no space)

**Strategy for tools:** Normalize all to plain integers. Strip leading P, leading zeros,
whitespace. Case-insensitive matching.

**Bug density by module (files referencing most unique PTMs):**
- COMMAND.COM: 67 PTMs (A001-A067) — most-patched component
- CHKDSK: 55 PTMs — in dedicated CHKCHNG.INC
- XCOPY: 27 PTMs
- FORMAT: ~28 PTMs (in FORCHNG.INC)
- RECOVER: 32 PTMs (in RECCHNG.INC)
- IFSFUNC (IFS): ~15+ per file, spread across 8+ files
- FDISK: 16 PTMs (in FDCHNG.INC)

**Timeline:** Dates in change logs span August 1987 through June 1988 (~10 months).
~376 bugs / ~300 working days = ~1.2 bugs filed per day.

### 3. ;ANxxx; and ;ACxxx; — Inline Change Tags

**Format:** `;AN000;` (new line, change 0) or `;AC015;` (changed line, change 15)
**104,388 tags** across 467 files — the most pervasive system

- **AN** = Addition, New line
- **AC** = Addition, Changed line (modified existing)
- The 3-digit number maps to a change log entry in the file header

**CRITICAL: Numbers are PER-MODULE, not global.**
- AN001 in XCOPY = PTM0011 (path >63 chars)
- AN001 in COMMAND1 = PTM P20 (move system_cpage)
- AN001 in CHKDSK = PTM P265 (subst drive check)
- AN001 in MSDISK = DCR D24 (multitrack config)

**Each file (or module group) has its own numbering starting at A000.**

**Change number ranges observed:**
- Most files: 000-030 range
- COMMAND.COM: 000-067
- MODE: up to 665 (!)
- SYSCONF.ASM: has 1,361 AN/AC tags (most tagged lines of any file)
- SYSINIT1.ASM: 849 tags

**AN000 is special:** It represents "DOS 4.00 Spec additions and DCRs through
unit/function test" — the initial 3.x→4.0 port work. The vast majority of tagged
lines are AN000.

### 4. DCR — Design Change Request (Feature Tracker)

**Format:** `DCR D166`, `DCR0201`, `dcr d490`
**~57 unique DCR numbers**

DCRs are planned enhancements, not bug fixes. Examples:
- DCR D166: Enable 128K FAT (major new capability)
- DCR D201: New extended attribute format
- DCR D17: Search for specified extension only
- DCR D43: Allow APPEND far call to SYSPARSE
- DCR D284: Change invalid code page tag from -1 to 0

### 5. Other Change Markers

**BIOS files:** Use `ANxxx` followed by description and `- J.K.` attribution
```
;AN001; d24 Multi Track enable/disable command in CONFIG.SYS  6/27/87 J.K.
```

**APPEND uses @@xx style:**
```
;  @@01 07/11/86 Fix hang in TopView start        PTM P00000??
```

**NLSFUNC uses =X style:**
```
; =A  7/29/86  RG   PTM P64
; =B  7/29/86  RG   PTM P67
```

**SHARE uses Axnnn style with module-level numbering.**

**Inline PTM comments without formal tags:** Some kernel files reference PTMs
in ad-hoc comments: `;PTM.`, `;fix ptm 112`, `;PTM P894`. These are less structured
but still parseable.

## SCCSID File Renaming Patterns

The SCCSID tags preserve **original filenames** that differ from current names,
revealing the IBM→MS branding layer and file reorganization.

### ibm* → ms* Renamings (IBM PC-DOS → MS-DOS branding)
```
ibmhalo.asm   → MSHALO.ASM     (hardware abstraction layer)
ibmdata.asm   → MSDATA.ASM     (DOS data segment)
ibmdata.asm   → MSDATA2.ASM    (DOS data segment, part 2)
ibmcode.asm   → MSCODE.ASM     (DOS code segment)
ibmcpmio.asm  → MSCPMIO.ASM    (CP/M compatibility I/O)
ibmctrlc.asm  → MSCTRLC.ASM    (Ctrl-C handler)
ibmdisp.asm   → MSDISP.ASM     (INT 21h dispatch)
ibmdosmes.asm → MSDOSME.ASM    (DOS messages)
ibmproc.asm   → MSPROC.ASM     (process management)
ibmsw.asm     → MSSW.ASM       (software switches)
ibmtable.asm  → MSTABLE.ASM    (dispatch tables)
IBMBDS.ASM    → MSBDS.INC      (BIOS data structure)
IBMEXTRN.ASM  → MSEXTRN.INC    (external declarations)
IBMIOCTL.INC  → MSIOCTL.ASM    (IOCTL handler)
```

### .ASM → .INC Renames (restructuring)
```
arena.asm    → ARENA.INC      buffer.asm   → BUFFER.INC
cpmfcb.asm   → CPMFCB.INC     curdir.asm   → CURDIR.INC
dirent.asm   → DIRENT.INC     dosmac.asm   → DOSMAC.INC
dossym.asm   → DOSSYM.INC     dpb.asm      → DPB.INC
error.asm    → ERROR.INC      exe.asm      → EXE.INC
filemode.asm → FILEMODE.INC   find.asm     → FIND.INC
intnat.asm   → INTNAT.INC     mi.asm       → MI.INC
mult.asm     → MULT.INC       pdb.asm      → PDB.INC
sf.asm       → SF.INC         syscall.asm  → SYSCALL.INC
sysvar.asm   → SYSVAR.INC     vector.asm   → VECTOR.INC
BPB.ASM      → BPB.INC        DEVSYM.ASM   → DEVSYM.INC
```

### Other Interesting Renamings
```
dossym.asm    → SMDOSSYM.INC   (small-model version of DOSSYM)
biomes.asm    → MSBIO.SKL      (became skeleton file for message insertion)
stdsw.asm     → EDLSTDSW.INC   (became EDLIN-specific)
strin.asm     → KSTRIN.ASM     (K prefix = ???)
mscode.asm    → MS_CODE.ASM    (underscored variant)
msdata.asm    → MS_DATA.ASM    (underscored variant)
mstable.asm   → MS_TABLE.ASM   (underscored variant)
parse.asm     → PARSE2.ASM     (second parse module)
keybxx.asm    → KBD.ASM        (keyboard, genericized name)
readclock.asm → READCLOC.INC   (8.3 truncation!)
stddosmes.as → STDDOSME.ASM   (original name already truncated!)
doscalls.hwc  → DOSCALLS.H     (.hwc = "header, wide char"? OS/2 origin)
subcalls.hwc  → SUBCALLS.H     (same .hwc pattern)
```

**Analysis patterns for tools:**
1. `ibm*` → `ms*`: The OEM branding abstraction layer
2. `.asm` → `.inc`: Files that became include-only (no longer assembled standalone)
3. Underscore insertion (`mscode` → `MS_CODE`): Possibly to distinguish from 3.x versions
4. 8.3 truncation artifacts: Some original names already exceeded 8.3!
5. `.hwc` → `.h`: OS/2 shared headers losing their OS/2-specific extension

## Developer Identifications

### Named Developers (confirmed attributions)
| ID | Full Name | Area | Source |
|----|-----------|------|--------|
| Ellen G | Ellen G(?) | COMMAND.COM (3.30→4.0) | COMMAND1.ASM revision history |
| Mark T. / MT | Mark T(?) | CHKDSK, FDISK, FORMAT | CHKCHNG.INC, FDISKMSG.H |
| Bruce B. | Bruce B(?) | CHKDSK (FAT, disk space) | CHKCHNG.INC |
| Dennis M. / DM / DMS | Dennis M(?) | CHKDSK, VDISK | CHKCHNG.INC, VDISKSYS.ASM |
| S. Maes / S.M | S. Maes | REPLACE, LABEL, XCOPY | REPLACE.C, LABEL.ASM |
| E. Kiser / Ed K | Ed Kiser | Parser (PARSE.ASM) | PARSE.ASM revision history |
| Bill L | Bill L(?) | Parser, COMP | PARSE.ASM, COMP2.ASM |
| J.K. | J.K.(?) | BIOS, boot, SYSINIT | Throughout BIOS/*.ASM |
| R.G. / RG | R.G.(?) | APPEND, NLSFUNC | APPEND.ASM, NLSFUNC.ASM |
| RMG / RGazzia | R(?) M(?) Gazzia | IFSFUNC (IFS subsystem) | IFSDEV.ASM and all IFS files |
| FJG | F(?) J(?) G(?) | SHARE | SHAREHDR.INC |
| GHG | G(?) H(?) G(?) | SELECT | CASERVIC.ASM |
| DRM | D(?) R(?) M(?) | XCOPY | DOS.EQU |
| RPS | R(?) P(?) S(?) | IFSFUNC | IFSDIR.ASM, IFSHAND.ASM |
| DL | D(?) L(?) | SHARE | SHAREHDR.INC |
| MZ | Mark Zbikowski | SHARE (1983-84) | SHAREHDR.INC |

### Pre-DOS-4 Developers (from revision histories)
| ID | Full Name | Area | Era |
|----|-----------|------|-----|
| -MU | Likely Mark Zbikowski | DOS 2.x | "Rev 2.50 all the 2.x new stuff -MU" |
| ARR / AARONR | Aaron Reynolds | CHKDSK (original author) | REV 1.1 through 3.0 (1982-83) |
| NP / NANCYP | Nancy Panners(?) | CHKDSK (extent reporting) | REV 1.5, 2.3 |
| RS | R(?) S(?) | CHKDSK (split, 0F0h media) | REV 3.05-3.20 (1984-85) |
| GA, DS | Two developers | CHKDSK | Change log |

### Locating Developer Info
- `Developer: XXX` lines in `*CHNG.INC` files (most structured)
- End-of-line initials on PTM lines: `3/88 RMG`
- Parenthetical in revision history: `(Ellen G)`
- After date on AN/AC comment: `6/27/87 J.K.`
- Embedded in code comments: `dms; fix ptm 112`

## Change Log Data Sources

### Dedicated Change Log Files (*CHNG.INC)
These are the richest, most structured source. 6 files:
```
CMD/CHKDSK/CHKCHNG.INC   - 55 entries, most detailed (date + developer + description)
CMD/FORMAT/FORCHNG.INC    - 28 entries
CMD/RECOVER/RECCHNG.INC   - 32 entries
CMD/FDISK/FDCHNG.INC      - 16 entries
CMD/EXE2BIN/E2BCHNG.INC   - 1 entry
CMD/FILESYS/FSCHNG.INC    - 2 entries
```

Format is highly structured with separator lines:
```
;* NNN - DOS 4.00 PTM Pxxxx - Description                                   *
;*   Date: mm/dd/yy   Developer: Name                                       *
```

### REVISION HISTORY Sections in File Headers
51 files have `REVISION HISTORY` sections. Format varies:
- XCOPY style: `A001 PTM0011 date description`
- COMMAND1 style: `A001  PTM P20  description`
- IFSFUNC style: `A001 - PTM 316 - description  date developer`
- BIOS style: `AN001; description  date initials`

### Inline-Only PTM References
Some files reference PTMs only in code comments, without a change log header:
```asm
;PTM.                    (kernel files, very terse)
;fix ptm 112             (VDISK)
;PTM P894  mt 12/8/86    (FORMAT)
```

## Quantitative Summary

| Metric | Count |
|--------|-------|
| Unique PTM numbers | ~376 |
| Unique DCR numbers | ~57 |
| Unique AN/AC change numbers | 100 (per-module, so ~100 × module count total) |
| Total inline AN/AC tags | 104,388 |
| Files with AN/AC tags | 467 |
| SCCSID tags | 283 across 203 files |
| Files with PTM references | 97 |
| Files with REVISION HISTORY | 51 |
| Dedicated *CHNG.INC files | 6 |
| Identifiable developers | ~15+ |
| File renamings detected via SCCSID | 54 |

## Planned Tooling

We intend to create tools to extract useful insights from this data.

### Tier 1: Data Extraction (foundation)
**`dos4_changelog_db.py`** — Parse all metadata into SQLite
- Parse *CHNG.INC structured entries
- Parse REVISION HISTORY sections (multiple format variants)
- Parse all ;ANxxx;/;ACxxx; inline tags with line numbers
- Parse SCCSID tags (filename, version, date)
- Normalize PTM numbers (strip P, leading zeros, whitespace)
- Extract developer attributions from all patterns
- Link inline tags → change log → PTM/DCR

### Tier 2: Views
**`dos_blame.py`** — per-file line-by-line attribution (like git blame)
**`dos_log.py`** — chronological change list (like git log), filterable
**`ptm_show.py`** — single-PTM deep dive (like gh issue view)

### Tier 3: Analysis & Visualization
**`dos_stats.py`** — bug density, developer workload, timeline
**`sccs_archaeology.py`** — file lineage, renaming tracker, code genealogy
**Timeline/heatmap** — HTML visualization of PTM timeline, directory heatmap

### Patterns to Capture in Analysis Tools

1. **IBM→MS renaming layer** — SCCSID `ibm*` → current `ms*` filenames
2. **Code genealogy** — SCCSID dates show DOS 3.x origin (1985) vs DOS 4.0 new (1987+)
3. **Bug clustering** — Which modules attracted the most PTMs and why
4. **Developer specialization** — Who owned which subsystem
5. **Bug discovery timeline** — Were bugs front-loaded or steady-state?
6. **AN000 vs ANxxx ratio** — How much of the code is base port vs. bug fixes?
7. **Cross-file PTMs** — Bugs that required changes across multiple files
8. **Orphan PTM references** — Inline `;PTM.` or `;fix ptm` without formal change log
9. **SCCSID version numbers** — Most are 1.1, but `IBMBDS.ASM 1.9` and `msdata.asm 1.8`
   had many pre-import revisions (heavily-worked files)
10. **8.3 filename artifacts** — `stddosmes.as`, `readclock.asm` showing filesystem limits
11. **OS/2 shared code** — `.hwc` extensions, BASEMID.H/UTILMID.H headers (OS/2 origins)
12. **;PTM. ad-hoc tags** — Kernel files use informal PTM markers without the full system
13. **Missing PTM numbers in range** — Gaps in P1-P5031 suggest PTMs for components not in this tree (OS/2, LAN Manager, etc.)
