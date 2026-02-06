# DOS 4.0 Community Research & External Findings

This document is a synthesis and consolidation of pre-existing community
responses, research, and findings regarding the DOS 4.0 source release.

Compiled from VOGONS, OS/2 Museum, BetaWiki, WinWorldPC, Hacker News, GitHub,
fabulous.systems, Virtually Fun, and others.
Research date: February 2026.

---

## 1. Version Identification: Is This DOS 4.00 or 4.01?

### Consensus

The released source is a **transitional snapshot between 4.00 and 4.01**. Build
scripts label it "4.01 SOURCE BAK" but version constants still say 4.00, and
compiled binaries match October 1988 "MS-DOS 4.00" distribution images.

### Evidence

| Evidence | Points to |
|----------|-----------|
| `SETENV.BAT` line 2: `"MS-DOS 4.01 SOURCE BAK"` | 4.01 |
| `RUNME.BAT`: `"MS-DOS 4.00 Build Notes"` | 4.00 |
| `VERSIONA.INC`: `minor_version equ 00` | 4.00 |
| All user-visible strings: `"Version 4.00"` | 4.00 |
| MS-only bug fixes dated Aug 5-7, 1988 (after IBM 4.0 shipped Aug 2) | Post-4.00 |
| Latest source date: Oct 3, 1988 (MEMM/A20TRAP.ASM) | Post-4.00 |
| Built binaries match WinWorldPC images dated 10/06/1988 | 4.00 Oct release |
| Howard Harte's 4.01 reconstruction matches April 1989 archive.org release | Different from this |

**WinWorld user Iori's characterization**: "the source code is of MS-DOS 4.00
from October 1988, which was based on PC DOS 4.01, not on PC DOS 4.00 from June
1988."

**Michal Necasek's conclusion**: "The just released source code almost certainly
corresponds to this quiet 4.01 update."

### Our Source Tree Forensics Found

- `;MS.` comment tags throughout kernel mark Microsoft-specific divergences
- `BUFFERFLAG EQU NOT IBMCOPYRIGHT` — EMS buffer caching exclusive to MS-DOS
- `IF NOT IBMCOPYRIGHT` guards on bug fixes dated Aug 5-7, 1988
- Fix in DISK2.ASM "migrated from 330a" (DOS 3.30a backport, NOT in IBM version)
- PTM numbers range up to P5197 (IBM's Problem Tracking system)
- SCCS IDs from DOS 2.x lineage preserved (e.g., `@(#)mstable.asm 1.3 85/07/25`)

---

## 2. The Release Story (April 25, 2024)

### Key People

| Person | Role |
|--------|------|
| **Connor "Starfrost" Hyde** | English researcher who initiated the effort; MT-DOS historian |
| **Ray Ozzie** | Former Microsoft CTO; found unreleased MT-DOS beta floppies from Lotus days |
| **Scott Hanselman** | Microsoft VP; coordinated disk imaging and release |
| **Jeff Sponaugle** | Internet archivist; imaged original disks |
| **Jeff Wilcox** | Microsoft OSPO; found MS-DOS 4.00 in Microsoft Archives |
| **Mark Zbikowski (mzbik)** | Original DOS 2.0-4.0 lead; made symbolic "MZ is back!" commit |
| **Larry Osterman** | Long-time Microsoft developer; acknowledged contributor |
| **IBM OSPO** | Partnered with Microsoft (IBM cooperation needed since DOS 4 was IBM-developed) |

### What Was Released

- Full MS-DOS 4.00 Source BAK (Binary Adaptation Kit) under MIT license
- Unreleased Multitasking DOS beta binaries ("Ozzie Drop") in `v4.0-ozzie/`
- Scanned PDFs of MT-DOS documentation
- Complete build toolchain: MSC 5.1 + MASM 5.10 in `src/TOOLS/`
- Disk images

### What Was Missing

- **DOSSHELL** — the TUI shell (used COW framework, possibly in separate tree)
- **HIMEM.SYS** — memory manager (neozeed published source separately)
- **GW-BASIC** — released separately earlier

---

## 3. Release Quality Criticisms

### Michal Necasek's "How Not To Release Historic Source Code"
URL: https://www.os2museum.com/wp/how-not-to-release-historic-source-code/

| Problem | Impact | Reversible? |
|---------|--------|-------------|
| Git destroyed file timestamps | Lost modification dates for all files | **No** |
| UTF-8 conversion of CP437 | MASM line length violations; USA.INF PANEL36/37 data destroyed | Partially |
| LF-only line endings | MASM requires CR/LF | Yes |
| Tim Paterson name removed post-publication | Historical record damaged | Preserved in early forks |

**Connor Hyde explained timestamp removal**: "data protection law mandates
anonymisation of source files, at least that is the policy."

**Necasek's recommendation**: "Historic source code should be released simply as
an archive of files, ZIP or tar or 7z or whatever, with all timestamps preserved
and every single byte kept the way it was."

### The Tim Paterson Comment

Original (`STRIN.ASM` line 70):
```asm
; Brain-damaged Tim Paterson ignored ^F in case his BIOS did not flush the
; input queue.
```
Changed by Mark Zbikowski (GitHub user mzbik) to:
```asm
; Brain-damaged TP ignored ^F in case his BIOS did not flush the
; input queue.
```

### Build Tool Licensing Question

The `src/TOOLS/` folder contains nearly complete MSC 5.1 and MASM 5.10 installations
(confirmed binary-identical to retail by VOGONS user "llm"). Whether including these
under MIT license was intentional is unclear — they were part of the Source BAK because
it was designed to be self-contained for OEMs.

---

## 4. Community Build Efforts

### Key Contributors

| Person | Platform | Key Finding |
|--------|----------|-------------|
| **neozeed** (virtuallyfun.com) | PS/2 16MHz 386 (70 min build) | `msload.asm` stack too small (64 bytes), boot failure |
| **Michal Necasek** | Analysis only | Identified stack overflow root cause |
| **E. C. Masloch (ecm)** | Mercurial repo | Restored SELECT files to exact binary match with retail |
| **Howard M. Harte (hharte)** | GitHub fork | UTF-8 fidelity fix; reconstructed 4.01 differences |
| **Lothar (fabulous.systems)** | Packages | Pre-built bootable floppy images for MS-DOS and PC-DOS |
| **John Elliott** | Analysis | DEBUG.COM CALL 5 bug; OEM ID restriction in MSINIT.ASM |
| **Irq5** (VOGONS) | Linux/QEMU pipeline | FreeDOS SYS.COM `/oem:Ms` for bootable images |
| **Paul Edwards (kerravon)** | PDOS | Built on Windows 2000; noted DosWrite() C-callable interface |

### Bugs Found by Community

| Bug | File | Description | Fix |
|-----|------|-------------|-----|
| Boot stack overflow | `msload.asm` | 64-byte stack too small for BIOS INT 13h | Increase to 128 bytes |
| DEBUG CALL 5 address | `DEBUG.ASM:463` | Wrong PSP CALL 5 address generation | Add `INC AX` instructions |
| OEM ID restriction | `MSINIT.ASM` | Requires "MSDOS", "IBM", or "OS2" OEM ID + broken version check | Relax checks |
| UTF-8 line overflow | `GETMSG.ASM` et al | CP437→UTF-8 expanded lines past MASM 512-byte limit | Replace U+FFFD with `-` |
| USA.INF data corruption | `SELECT/USA.INF` | PANEL36/37 byte arrays destroyed by UTF-8 | Restore from retail binaries |

### Minimal Changes to Build

1. Remove/shorten long comment lines exceeding MASM line buffer (5-6 files)
2. Fix `LIB` and `INCLUDE` paths in `SETENV.BAT` (add `%BAKROOT%\src\tools\bld\lib`)
3. Fix UTF-8 corrupted byte sequences
4. Convert LF → CR/LF line endings

### Binary Match Achieved

**ecm** and **felsqualle** confirmed: built binaries are a **perfect match** against
retail MS-DOS 4.00 (10/06/1988) disk images from archive.org. Files present in retail
but missing from source: AUTOEXEC.BAT, GWBASIC.EXE, PCIBMDRV.MOS, SHELLC.EXE,
SHELL.MEU, CONFIG.SYS, HIMEM.SYS, README.TXT, SHELL.CLR, DOSUTIL.MEU, LINK.EXE,
SHELLB.COM, SHELL.HLP.

---

## 5. Historical Context

### Development History

- **Preliminary name**: MS-DOS 3.40 (confirmed by Microsoft-Zenith contract, August 1987)
- **IBM code name**: "Tugboat"
- **Only DOS version primarily developed by IBM** (Boca Raton)
- Most Microsoft DOS engineers were busy building OS/2
- Mark Zbikowski was DOS development lead from 2.0 through 4.0 but IBM drove this version

### Release Timeline

| Date | Event |
|------|-------|
| July 19, 1988 | IBM PC-DOS 4.0 announced ($150 new, $95 upgrade) |
| ~Aug 1988 | "Quiet" 4.01 update (disks labeled 4.01, software says 4.00) |
| Aug 15, 1988 | CSD UR22624 |
| Oct 6, 1988 | MS-DOS 4.00 distribution (matches our source) |
| Mar 27, 1989 | CSD UR24270 |
| Apr 1989 | MS-DOS 4.01 release (matches hharte's reconstruction) |
| May 10, 1989 | CSD UR25066 (EMS fix) |
| Sep 25, 1989 | CSD UR27164 (>2 hard disks fix) |
| Mar 20, 1990 | CSD UR29015 (up to 7 hard disks) |
| 1990 | **DOS 4.02** shipped in IBM PS/1 ROM (still reports 4.00) |
| Early 1990 | **Russian MS-DOS 4.01** released (one of first Western software for USSR) |
| Jun 29, 1990 | CSD UR31300 |
| Jul 2, 1990 | MS-DOS 4.02 mentioned in Microsoft document (never found) |

### OEM Releases Tracked by BetaWiki

- **4.00**: RM Nimbus, Sampo (very rare)
- **4.01**: AGI, Amstrad, AST, Bondwell, Emerson, EPSON, HP, Inves, Mitac, Olivetti,
  Packard Bell, Philips, Phoenix, RM Nimbus, Sharp, Trigem, Tulip, Twinhead, VEGAS, Zenith
- **4.01a**: Nokia, Victor
- **4.01d**: Compaq

### Why DOS 4.0 Had a Bad Reputation

**The EMS BUFFERS /X bug**:
- `BUFFERS /X` assumed 6+ EMS handles (IBM boards provided this)
- Non-IBM EMS boards (Intel, AST, etc.) often provided only 4
- Result: data corruption potential on clone hardware

**Memory usage**: ~70KB vs ~54KB for DOS 3.3 (16KB difference, NOT "double" as claimed)

**Other issues**: DOSSHELL mouse bug (keyboard lock check), internal structure changes
breaking TSRs, >2 hard disk infinite loop

**Necasek's analysis**: "Once the 'DOS 4.0 is bad' meme set in, it was all over."
DR DOS skipped version 4.0 entirely (3.41→5.0). Microsoft raised DOS 3.3 prices to
force OEM upgrades to 4.01 — this backfired and created an opening for DR DOS.

### The Two Different "DOS 4.0" Products

1. **Multitasking MS-DOS 4.0** (1985-1986): Based on MS-DOS 2.0. Preemptive multitasking,
   shared memory, semaphores, NE format, session manager. Licensed to Goupil (France),
   ICL (UK), Siemens (Germany). Never shipped in North America. Proto-OS/2.

2. **Retail MS-DOS 4.0** (1988): IBM-developed successor to DOS 3.3. The one that was
   open-sourced. Completely unrelated to the multitasking version despite sharing a number.

---

## 6. Source Code Archaeological Highlights (Found by Community)

### Extended Attributes (XA/EA) — Abandoned OS/2 Integration
- COPY command has EA support code
- DOS kernel has XA support stubbed out (substantial code written then commented out)
- Planned HPFS support that never shipped

### IFSFUNC.EXE — Installable File System Interface
- IBM's attempt to abstract the INT 2Fh/11h redirector interface
- Never publicly documented by IBM
- Only IBM's PC LAN Program (REDIRFS.EXE) used it
- Removed in DOS 5.0

### CP/DOS API Mapper (`src/MAPPER/`)
- Translates OS/2 Family API (FAPI) calls to DOS INT 21h
- Every file titled `"CP/DOS DosXxx mapper"`
- Direct artifact of IBM-Microsoft co-development period

### Version Fake Table
- `MSINIT.ASM` has hardcoded version-lying for specific programs
- Includes `WIN200.BIN`, `IBMCACHE.COM`
- Predecessor to DOS 5.0+ SETVER mechanism

### SELECT Installer Quirks (Necasek)
- Determines media type from drive type, not actual disk (breaks in VMs)
- Attempts to modify installation disk in-place
- `\\SPIDERMAN\DRIVEA` IBM internal network share in slack space
- File timestamps: June 14, 1988

### Developer Initials in Source

| Initials | Name | Role |
|----------|------|------|
| **MZ** | Mark Zbikowski | DOS 2.0-4.0 lead, "MZ" executable format |
| **ARR** | Aaron Reynolds | Key DOS 2.0 developer, ALLOC.ASM, PROC.ASM |
| **J.K.** | Unknown | Primary BIOS and boot developer (DOS 4) |
| **HKN** | Unknown | Late-stage MS-specific fixes (Jul-Aug 1988) |
| **mw** | Unknown | Secondary cache fix (Aug 5, 1988) |
| **F.C.** | Unknown | Earlier DOS 3.3 work |
| **LB.** | Bill L.? | Buffer management |
| **RMG** | Unknown | IFS subsystem |
| **SP/sunilp** | Sunil P. | RAMDRIVE, various fixes |
| **ISP/isp** | Unknown | MEMM/EMM subsystem (latest dates) |
| **EMG** | Ellen G.? | COMMAND.COM additions for 4.00 |
| **SEH** | Unknown | SELECT installer |
| **DBM** | Unknown | STRIN.ASM (IBM developer, 1987) |
| **PC** | Unknown | A20TRAP.ASM (Oct 1988, latest date in tree) |

---

## 7. Community Forks

| Project | Maintainer | Purpose |
|---------|-----------|---------|
| [neozeed/dos400](https://github.com/neozeed/dos400) | neozeed | Working build with boot + CALL 5 fixes |
| [hharte/MS-DOS](https://github.com/hharte/MS-DOS) | Howard Harte | UTF-8 fixes, 4.01 reconstruction |
| [HanHeld/DOS4x](https://github.com/HanHeld/DOS4x) | HanHeld | Aggregated community fixes |
| [ecm's Hg repo](https://hg.pushbx.org/ecm/msdos4/) | E. C. Masloch | Binary-matching restoration |
| [dos-fx/DOS-FX](https://github.com/dos-fx/DOS-FX) | dos-fx | Fork targeting i586+ |
| [OwnedByWuigi/DOS](https://github.com/OwnedByWuigi/DOS) | OwnedByWuigi | Preserves original comments; Envy OS subsystem |
| [neozeed/himem.sys-2.06](https://github.com/neozeed/himem.sys-2.06) | neozeed | HIMEM.SYS source (not in MS release) |

---

## 8. What The Community Has NOT Analyzed

Despite significant activity, **no public deep-dive exists** that systematically:
- Walks through the INT 21h dispatch table implementation
- Analyzes the undocumented API calls visible in the source
- Compares kernel architecture vs DOS 3.3 or DOS 5.0
- Surveys dead/commented-out code comprehensively
- Documents the OEM backdoor (AH=F8h) mechanism
- Maps the full SLEAZE function set
- Catalogs all `IF NOT IBMCOPYRIGHT` divergences

---

## 9. Future Release Prospects

- **Hanselman said DOS 3.3, 5, and 6 are "next on the list"** (as of April 2024)
- Some utilities would need to be stripped due to third-party licensing
- GitHub issue #424 (archived) showed Microsoft citing third-party licensing restrictions
- As of February 2026, **no additional DOS versions have been released**
- DOS 6 leaked source is an "incomplete unbuildable 6.0 beta, not 6.22"

---

## 10. Key External References

### Primary Analysis
- [OS/2 Museum: How Not To Release Historic Source Code](https://www.os2museum.com/wp/how-not-to-release-historic-source-code/)
- [OS/2 Museum: DOS 4.0 main page](https://www.os2museum.com/wp/dos/dos-4-0/)
- [OS/2 Museum: DOS 4.0, Bum Rap](http://www.os2museum.com/wp/dos-4-0-bum-rap-and-mismatched-expectations/)
- [OS/2 Museum: Shell Mouse Mystery](http://www.os2museum.com/wp/the-dos-4-0-shell-mouse-mystery/)
- [OS/2 Museum: SELECT Is Too Clever](https://www.os2museum.com/wp/learn-something-old-every-day-part-xvi-dos-4-0-select-is-too-clever/)
- [OS/2 Museum: >2 Hard Disks Bug](https://www.os2museum.com/wp/more-than-two-hard-disks-in-dos/)
- [OS/2 Museum: MS-DOS OAKs](http://www.os2museum.com/wp/ms-dos-oaks/)
- [OS/2 Museum: Multitasking DOS Lives](https://www.os2museum.com/wp/multitasking-ms-dos-4-0-lives/)
- [OS/2 Museum: Goupil OEM](http://www.os2museum.com/wp/multitasking-ms-dos-4-0-goupil-oem/)
- [OS/2 Museum: Anti-DR-DOS Detection](https://www.os2museum.com/wp/how-to-void-your-valuable-warranty/)

### Release Announcements
- [Microsoft Open Source Blog](https://opensource.microsoft.com/blog/2024/04/25/open-sourcing-ms-dos-4-0)
- [Scott Hanselman's Blog](https://www.hanselman.com/blog/open-sourcing-dos-4)
- [Starfrost: MT-DOS History](https://starfrost.net/blog/001-mdos4-part-1/)

### Community Build Work
- [fabulous.systems: Broken Source Restored](https://fabulous.systems/posts/2024/05/the-broken-source-code-for-ms-dos-4-has-been-restored/)
- [fabulous.systems: MS-DOS 4.01 Source](https://fabulous.systems/posts/2024/05/a-minor-update-ms-dos-4-1-is-here/)
- [fabulous.systems: New Installation Media](https://fabulous.systems/posts/2024/06/new-installation-media-for-ms-dos-4/)
- [Virtually Fun: Build on PS/2](https://virtuallyfun.com/2024/04/28/compiling-ms-dos-4-0-from-dos-4-0-on-a-ps-2/)
- [Virtually Fun: Build under OS/2 2.x](https://virtuallyfun.com/2024/06/10/building-ms-dos-4-00-under-os-2-2-x/)

### Forum Discussions
- [VOGONS: MS-DOS 4.00 is now open source](https://www.vogons.org/viewtopic.php?t=100241)
- [VOGONS: DOS 4.0 from source to system](https://www.vogons.org/viewtopic.php?t=100714)
- [VOGONS: Why is MS-DOS 4.0 disliked?](https://www.vogons.org/viewtopic.php?t=58191)
- [BTTR: MSDOS 4.0 discussion](https://www.bttr-software.de/forum/mix_entry.php?id=21722)

### Preservation/Documentation
- [BetaWiki: MS-DOS 4](https://betawiki.net/wiki/MS-DOS_4)
- [BetaWiki: Multitasking MS-DOS 4](https://betawiki.net/wiki/Multitasking_MS-DOS_4)
- [WinWorldPC: MS-DOS 4.x](https://winworldpc.com/product/ms-dos/4x)
- [PCjs: IBM DOS 4.00](https://www.pcjs.org/software/pcx86/sys/dos/ibm/4.00/)
- [Internet Archive: MS-DOS 4 source](https://archive.org/details/ms-dos4-source-code)

### News Coverage
- [Tom's Hardware](https://www.tomshardware.com/software/operating-systems/museum-criticizes-microsoft-for-mutilated-ms-dos-4-open-source-release-posting-on-stupid-git-blamed-for-the-buggy-blunder)
- [The Register](https://www.theregister.com/2024/04/26/ms_dos_4_open_source/)
- [TechSpot](https://www.techspot.com/news/102802-ms-dos-400-source-code-sloppy-git-dump.html)
- [Hackaday](https://hackaday.com/2024/04/26/microsoft-updates-ms-dos-github-repo-to-4-0/)
