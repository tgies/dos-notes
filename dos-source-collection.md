# DOS Source Code Collection & Availability

## Overview

We have access to source code for nearly every major MS-DOS version, spanning the
entire 1981-1993 lineage. This document tracks what exists, where it came from, how
complete it is, and what's still missing.

---

## Source Code Inventory

### Official Microsoft Releases (MIT Licensed)

| Version | Year | Location | License | Contents |
|---------|------|----------|---------|----------|
| DOS 1.25 | 1982 | `MS-DOS/v1.25/` | MIT | Full source, Tim Paterson email |
| DOS 2.0 | 1983 | `MS-DOS/v2.0/` | MIT | Full source, OEM Adaptation Kit docs |
| DOS 4.0 (multitasking) | 1986 | `MS-DOS/v4.0-ozzie/` | MIT | Binaries and PDFs only, no source |
| DOS 4.0 (retail) | 1988 | `MS-DOS/v4.0/src/` | MIT | Full OAK source, building in CI |

Released by Microsoft on GitHub in three waves:
- March 2014: DOS 1.25, 2.0 (original release via Computer History Museum)
- September 2018: Re-released on GitHub under MIT license
- April 25, 2024: DOS 4.0 added (joint release with IBM); repo archived (read-only)

The DOS 4.0 release was initiated by researcher Connor "Starfrost" Hyde, who
contacted Ray Ozzie about unreleased MT-DOS beta floppies from his Lotus days.
Scott Hanselman, Jeff Sponaugle, and Jeff Wilcox coordinated the release.
Hanselman said DOS 3.3, 5, and 6 are "next on the list" but as of Feb 2026
none have been released (third-party licensing restrictions cited).

GitHub: https://github.com/microsoft/MS-DOS

### Leaked Source Code (Proprietary)

| Version | Year | Location | Origin | Contents |
|---------|------|----------|--------|----------|
| DOS 3.30 | 1987 | Not yet cloned | ? | OAK (partial, see below) |
| DOS 6.0 beta | 1993 | `~/msdos/astro/` | ? | Mostly complete (build 0204) |

DOS 3.3 may come from the 42.9 GB Microsoft source code torrent that appeared on 4chan
on September 24, 2020. That torrent also included Windows NT 3.5/4, Windows 2000,
Windows XP SP1, Server 2003, Windows CE, Xbox OS, and DDKs from Win 3.11 to Win 7.

Some sources reference a "Russian" source for the DOS 6 leak -- need to check.

GitHub mirrors by user AR1972:
- DOS 3.3: https://github.com/AR1972/DOS3.3
- DOS 6 (Astro): https://github.com/AR1972/astro

### Not Available

| Version | Year | Status | Notes |
|---------|------|--------|-------|
| DOS 3.0-3.2 | 1984-86 | No source known | Developed by Microsoft with IBM involvement |
| DOS 5.0 | 1991 | No separate source | See "DOS 5 via DOS 6" below |

---

## Detailed Analysis by Version

### DOS 1.25 -- Full Source (MIT)

- **Primary developer:** Tim Paterson (sole author)
- **Key file:** `v1.25/source/STDDOS.ASM` -- "MS-DOS version 1.25 by Tim Paterson"
- **Build flags:** `IBM EQU FALSE` / `NOT IBM` for IBM vs generic builds
- **Also includes:** Tim Paterson's 2013 email confirming history

### DOS 2.0 -- Full Source (MIT)

- **Primary developers:** Aaron Reynolds (ARR), Mark Zbikowski (MZ), Chris Peters
- **Key files:** `v2.0/source/STDSW.ASM` (build flags), `MSCODE.ASM` (kernel)
- **Build flags:** `MSVER`/`IBM` in STDSW.ASM

### DOS 3.30 -- OAK Only (Leaked)

**Source:** https://github.com/AR1972/DOS3.3 (from 2020 leak)

This is the MS-DOS 3.30 OEM Adaptation Kit. It's missing MSDOS.SYS and COMMAND.COM
sources.

**Included (with source):**
- IO.SYS (BIOS layer) -- full source, the part OEMs needed to customize
- Boot sector code
- FORMAT, SYS, PRINT, SORT -- hardware-dependent utilities with full source
- AG.DOC -- Adaptation Guide, dated July 24, 1987
- README.BF -- Bug fix diskette documentation
- Bug fix patches for FDISK, FORMAT, boot sequence, file handle issues

**Binary/object only (no source):**
- MSDOS.SYS (kernel) -- shipped as object files only
- COMMAND.COM -- binary only

**Why partial:** Microsoft deliberately withheld kernel source from 3.x OAKs to
prevent OEM fragmentation. Per OS/2 Museum: "an increasing number of utilities and
applications relied on certain details of DOS implementation" so Microsoft wanted to
prevent unnecessary OEM variations. By DOS 4.0 (IBM-developed), the full kernel
source was included.

**Key document: AG.DOC**
- Full title: "MS-DOS 3.3 Adaptation Guide"
- Copyright: Microsoft Corporation, 1982-1986
- Last updated: July 24, 1987 (Final Release)
- Contents: OEM installation procedures, device driver architecture, boot process,
  FAT structure documentation, disk interface specs

### DOS 4.0 (Retail) -- Full OAK Source (MIT)

- **Primary developers:** IBM Boca Raton team (J.K., Ellen G, Sunil P., Bill L.)
  with earlier Microsoft code (ChrisP, AaronR, MarkZ)
- **Build system:** Working CI pipeline (see MEMORY.md for build commands)
- **Build flags:** IBMVER/IBMCOPYRIGHT three-way matrix in VERSION.INC
- **Message system:** First version with structured BUILDMSG system.
  - BUILDMSG.EXE compiles .SKL + .MSG into .CL* files
  - Binary format with class IDs, version headers, message ID tables
  - Length-prefixed strings with self-relative offsets
  - 948 extractable messages across 48 utilities
  - See [message-system-localization.md](message-system-localization.md)

#### Version Identification (4.00 vs 4.01)

The released source is a **transitional 4.00→4.01 tree**:
- `SETENV.BAT` says "MS-DOS 4.01 SOURCE BAK" (Binary Adaptation Kit)
- All version constants and user-visible strings say 4.00
- Built binaries perfectly match October 1988 "MS-DOS 4.00" WinWorldPC images
- Contains MS-only bug fixes dated Aug 5-7, 1988 (after IBM's PC-DOS 4.0 shipped Aug 2)
- Latest source date: Oct 3, 1988 (MEMM/A20TRAP.ASM)
- PTM (IBM Problem Tracking) numbers up to P5197
- Howard Harte's separate 4.01 reconstruction matches the April 1989 MS-DOS 4.01 release

**Historical names:** Originally "MS-DOS 3.40" (Microsoft-Zenith contract, Aug 1987).
IBM code name: "Tugboat". Only DOS version primarily developed by IBM.

See [dos4-community-research.md](dos4-community-research.md) for full community findings.

### DOS 5.0 -- Via DOS 6 Source

**No separate DOS 5 source exists in any known leak.**

However, the Astro (DOS 6.0) source essentially contains most of DOS 5 unchanged.
Per BetaArchive research:

- A huge number of DOS 6.00 components are **byte-for-byte identical** to DOS 5.00
- Only three differences in most files:
  1. License string: "MS DOS 5.00" / "1981-1991" -> "MS DOS 6" / "1981-1993"
  2. Version check: `0005h` -> `0006h`
  3. Most files are exactly **3 bytes smaller** in 6.0 ("5.00" is 3 chars > "6")
- Actual code changes in 6.0 concentrated in system files (IO.SYS, MSDOS.SYS,
  COMMAND.COM, FORMAT.COM, SYS.COM) plus new components (DoubleSpace, MS Backup,
  Defrag, etc.)
- Most utilities weren't actually changed until DOS 6.20

**Source:** https://www.betaarchive.com/forum/viewtopic.php?t=29579

### DOS 6.0 (Astro) -- Mostly Complete (Leaked)

- **Location:** `~/msdos/astro/`
- **Build:** 6.00.0204 (beta, not final RTM)
- **Primary developers:** Microsoft (SMR, HKN, CAS, MD, MRW, etc.)
- **Status:** Mostly complete but some include files with localization strings missing
- **Caveat:** From `astro/README.MD` -- some code may be modern reconstruction to
  replace missing pieces. Copyright notices and developer attributions appear original.
- **Message system:** Same structured BUILDMSG system as DOS 4, inherited and extended

---

## Completeness Visualization

```
DOS 1.25  [==========] Full source (official, MIT)
DOS 2.0   [==========] Full source (official, MIT)
DOS 3.0   [----------] No source known
DOS 3.30  [====------] OAK: BIOS + utils, no kernel (leaked)
DOS 4.0   [==========] Full OAK, building in CI (official, MIT)
DOS 5.0   [=========-] ~95% via DOS 6 source (leaked)
DOS 6.0   [=========-] Beta build 0204, mostly complete (leaked)
```

The one genuine gap is the **DOS 3.30 kernel source** (MSDOS.SYS). Microsoft
intentionally withheld it from the OAK. Everything else is either directly available
or derivable.

---

## The 2020 Microsoft Source Code Leak

### What Happened

On September 24, 2020, a 42.9 GB torrent appeared on 4chan containing compressed
source code for multiple Microsoft operating systems and products.

### Legal Status

All leaked code remains proprietary Microsoft intellectual property. The MIT license
only applies to the officially released DOS 1.25, 2.0, and 4.0. The leaked DOS 3.3
and 6.0 source cannot be legally used for any purpose.

### GitHub Mirrors

User AR1972 maintains GitHub mirrors of the DOS-related leaks:
- https://github.com/AR1972/DOS3.3 (19 stars)
- https://github.com/AR1972/astro (69 stars)
- https://github.com/AR1972/NT3.5

---

## What Was NOT Leaked

Notable absences from the 2020 torrent:
- **DOS 5.0** -- not included as a separate item (partially available via DOS 6)
- **DOS 3.0-3.2** -- not included
- **DOS Shell (DOSSHELL)** -- never open-sourced or leaked
- **SELECT.EXE installer** -- shares code with DOSSHELL, also unavailable
- **GWBASIC / BASICA** -- source not in any known leak

---

## Message System Evolution Across Versions

| Version | Message System | Localization Method |
|---------|----------------|---------------------|
| DOS 1.25 | Hardcoded `DB` strings | Edit source, reassemble |
| DOS 2.0 | Hardcoded `DB` strings with `$` terminator | Edit source, reassemble |
| DOS 3.30 | Hardcoded strings (likely, needs confirmation) | Edit source + COUNTRY.SYS for formatting |
| DOS 4.0 | **Structured BUILDMSG system** | Swap .MSG files, rebuild with BUILDMSG |
| DOS 5.0+ | Structured BUILDMSG (inherited) | Same as DOS 4 |
| DOS 6.0 | Structured BUILDMSG (inherited) | Same as DOS 4 |

The structured system was introduced in DOS 4.0 as part of the IBM joint development.
DOS 3.3 improved internationalization through COUNTRY.SYS and KEYBOARD.SYS but did
not use the structured message system.

See [message-system-localization.md](message-system-localization.md) for full details.

---

## Development Timeline (Who Wrote What)

| Version | Year | Primary Developer | Evidence |
|---------|------|-------------------|----------|
| DOS 1.0 | 1981 | Tim Paterson / Microsoft | Source, universal consensus |
| DOS 1.25 | 1982 | Tim Paterson / Microsoft | Source: "by Tim Paterson" |
| DOS 2.0 | 1983 | Microsoft (Zbikowski, Reynolds, Peters) | Source developer names |
| DOS 3.0-3.2 | 1984-86 | Microsoft with IBM | OS/2 Museum |
| DOS 3.3 | 1987 | **IBM (Boca Raton)** | OS/2 Museum, source PTMs, "Boca folks" |
| DOS 3.31 | 1987 | **Compaq** (OEM backport of FAT16B) | OS/2 Museum, WinWorld |
| DOS 4.0 | 1988 | **IBM (Boca Raton)** | Source: IBM devs, PTMs, SCCS |
| DOS 5.0 | 1991 | **Microsoft** (response to DR DOS) | Computer History Wiki |
| DOS 6.0 | 1993 | **Microsoft** | Source: developer initials throughout |

---

## TODO: Future Investigation

- [ ] Clone DOS 3.3 repo locally and examine contents in detail
- [ ] Confirm DOS 3.3 message system (grep for BUILDMSG, .MSG, .SKL, $M_CLASS)
- [ ] Compare DOS 3.3 OAK structure with DOS 4.0 OAK structure
- [ ] Try building DOS 3.3 OAK (may need period-correct MASM)
- [ ] Examine if DOS 3.3 bug fix patches are applicable to DOS 4 source
- [ ] Look for DOS 5.0-specific code paths in Astro source (version checks, changelogs)

---

## Sources

### Official
- Microsoft MS-DOS GitHub: https://github.com/microsoft/MS-DOS
- MS-DOS 4.0 release announcement (April 2024)

### Leak Documentation
- LinuxReviews: https://linuxreviews.org/42.9_GB_Of_Microsoft_Source_Code_Leaked
- VOGONS thread: https://www.vogons.org/viewtopic.php?t=76823
- BetaArchive contents: https://www.betaarchive.com/forum/viewtopic.php?t=41873
- DOS 5/6 component comparison: https://www.betaarchive.com/forum/viewtopic.php?t=29579

### Technical Analysis
- OS/2 Museum (Michal Necasek): https://www.os2museum.com/wp/dos/
- OS/2 Museum on OAKs: https://www.os2museum.com/wp/ms-dos-oaks/
- OS/2 Museum DOS 3.3: https://www.os2museum.com/wp/dos/dos-3-3/
- OS/2 Museum DOS 4.0: https://www.os2museum.com/wp/dos/dos-4-0/
- Computer History Wiki: https://gunkies.org/wiki/MS-DOS
