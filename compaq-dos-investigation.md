# The Compaq DOS Myth: Comprehensive Investigation

## Background

An article by Gareth Edwards at `https://every.to/the-crazy-ones/the-man-who-beat-ibm`
(June 16, 2025) claims that Compaq secretly took over MS-DOS development from Microsoft,
with the code licensed back. An archived copy exists at `https://archive.is/A8eQx`.

Rod Canion's 2013 memoir *Open: How Compaq Ended IBM's PC Domination and Helped Invent
Modern Computing* is the original source for some of these claims.

The article's claims have already been added to Wikipedia's MS-DOS article under a
"COMPAQ-DOS" section (reference [25], `edwards20250616`), citing only the Edwards article.

A Hacker News discussion at `https://news.ycombinator.com/item?id=44781343` (85 points,
39 comments) contains the most substantive technical rebuttal, by user **ndiddy**.

---

## Source Code Repositories Used

| Version | Location | Contents |
|---------|----------|----------|
| DOS 1.25 | `MS-DOS/v1.25/` | Full source, Tim Paterson email |
| DOS 2.0 | `MS-DOS/v2.0/` | Full source, OEM Adaptation Kit docs |
| DOS 4.0 (multitasking) | `MS-DOS/v4.0-ozzie/` | Binaries and PDFs only, no source |
| DOS 4.0 (retail) | `MS-DOS/v4.0/src/` | Full source, jointly developed by IBM and MS |
| DOS 6.0 (Astro) | `astro/` | Partial/reconstructed source (leaked beta) |

**Caveat on DOS 6**: The Astro repo is a reconstructed build from a leaked DOS 6.0.500
beta. Some code was rewritten from scratch to replace missing pieces. Copyright notices,
change logs, and developer attributions appear original and period-accurate, but some
peripheral code may be modern reconstruction. See `astro/README.MD`.

---

## The Article's Specific Claims and Verdicts

### Claim 1: "Microsoft DOS had never been a true copy of PC DOS"

**Article quote**: "Both men knew why: Microsoft DOS had never been a true copy of PC DOS,
as Gates had admitted to Canion during the development of Compaq's first machine. The
differences had only increased over time, as Microsoft's deal with IBM prohibited the
same developers working on both versions."

**Verdict: MISLEADING**

**Evidence**: MS-DOS and PC DOS were built from the SAME codebase using conditional
compilation flags. This system exists in every version of MS-DOS source code we have:

- **DOS 1.25** (`v1.25/source/MSDOS.ASM`): `IBM EQU FALSE` / `NOT IBM` controls IBM vs
  generic builds. Different BIOS segments, escape characters, device names, keyboard
  modes.
- **DOS 1.25** (`v1.25/source/COMMAND.ASM`): `IBMVER`/`MSVER` flags.
- **DOS 2.0** (`v2.0/source/STDSW.ASM`, lines 4-7):
  ```
  ; Use the switches below to produce the standard Microsoft version or the IBM
  ; version of the operating system
  MSVER   EQU     false
  IBM     EQU     true
  ```
- **DOS 4.0** (`v4.0/src/INC/VERSION.INC`, lines 20-35):
  ```
  ;                     IBMVER          IBMCOPYRIGHT
  ; --------------------------------------------------------
  ;  IBM Version   |     TRUE              TRUE
  ; --------------------------------------------------------
  ;  MS Version    |     FALSE             FALSE
  ; --------------------------------------------------------
  ;  Clone Version |     TRUE              FALSE
  ;
  IBMVER     EQU     TRUE
  IBMCOPYRIGHT EQU   FALSE
  ```
- **DOS 6.0** (`astro/inc/version.inc`): Same IBMVER/IBMCOPYRIGHT system.

**The critical distinction**: There were THREE build flavors, not two:
1. **IBM Version** (IBMVER=TRUE, IBMCOPYRIGHT=TRUE) = IBM PC DOS
2. **Clone Version** (IBMVER=TRUE, IBMCOPYRIGHT=FALSE) = OEM MS-DOS for IBM-compatible
   clones (Compaq, Dell, HP). This was what most consumers used as "MS-DOS."
3. **MS Version** (IBMVER=FALSE, IBMCOPYRIGHT=FALSE) = Generic MS-DOS for non-IBM-
   compatible hardware (Tandy 2000, early Zenith, etc.)

The article conflates the generic "MS Version" (which was indeed less IBM-compatible)
with the "Clone Version" that IBM-compatible OEMs actually shipped. Compaq built with
the Clone Version settings, as did most PC-compatible OEMs. The codebases did NOT
diverge — they were the same code with different compile-time flags.

**Key files for verification**:
- `v1.25/source/MSDOS.ASM` — lines with `IF IBM` / `IF NOT IBM`
- `v2.0/source/STDSW.ASM` — MSVER/IBM flag definitions
- `v4.0/src/INC/VERSION.INC` — three-way build matrix
- `astro/inc/version.inc` — same system in DOS 6

---

### Claim 2: "Compaq DOS was more compatible with PC DOS than Microsoft's own product"

**Article quote**: "With its singular focus on 100 percent compatibility, the result was a
product that was more compatible with PC DOS than Microsoft's own product."

**Verdict: TRUE FOR EARLY PERIOD (pre-1986), INCREASINGLY FALSE AFTERWARD**

**Evidence**: In the early 1980s, some OEMs received MS-DOS with IBMVER=FALSE (the
generic version), which lacked IBM PC-specific features. Compaq built with IBMVER=TRUE,
making their version closer to PC DOS. Additionally, early OEMs had to write their own
versions of IBM-specific utilities (FDISK, MODE, etc.) since Microsoft didn't have
generic equivalents.

However, by **MS-DOS 3.2 (1986)**, Microsoft offered a "packaged product" with its own
IBM-compatible utilities. By **DOS 3.3 (1987)**, IBM itself was writing the code. The
compatibility gap that the article describes was real but narrowed rapidly and was
essentially gone by the mid-1980s.

**Sources**:
- OS/2 Museum (`os2museum.com/wp/dos/`): Confirms MS-DOS 3.2 was the first packaged
  version for generic PC clone OEMs.
- HN user ndiddy: "MS-DOS 3.2 (1986) was the first version that Microsoft made available
  in a packaged form to OEMs that were shipping PC clones."
- HN user fredoralive: Notes that "Microsoft didn't have its own versions of FDISK, MODE,
  etc. for generic PC clones" until 3.2.

---

### Claim 3: "Gates was able to halt all internal development on Microsoft DOS"

**Article quote**: "From Gates's perspective, this was an incredible deal. He was able to
halt all internal development on Microsoft DOS, saving time and money."

**Verdict: FALSE**

**Evidence from source code — developer attribution across versions**:

#### DOS 1.25 (1982) — Microsoft
| Developer | Role | Evidence |
|-----------|------|----------|
| Tim Paterson | Sole author | `TITLE MS-DOS version 1.25 by Tim Paterson March 3, 1982` in STDDOS.ASM |

- Also confirmed by Tim Paterson's email (`v1.25/Tim_Paterson_16Dec2013_email.txt`):
  "MS-DOS 1.25 was the first general release to OEM customers other than IBM"

#### DOS 2.0 (1983) — Microsoft
| Developer | Initials | Role | Evidence Location |
|-----------|----------|------|-------------------|
| Tim Paterson | — | Originator (Ret.) | MSHEAD.ASM header |
| Aaron Reynolds | ARR | Bug fixes, I/O rewrite | MSCODE.ASM history (06/04/82), DEBUG.ASM line 51 |
| Mark Zbikowski | MZ | Architecture lead | "yay zibo!" comment re: MZ exe signature; DEBUG.ASM "Ztrace mode by zibo" |
| Chris Peters | — | BIOS development | MSHEAD.ASM header "(BIOS) (ret.)" |
| Nancy Panners | NP | EDLIN revisions | EDLIN.ASM revisions 4/14/83, 7/23/83 |
| M.A. Ulloa | — | EDLIN primary dev | EDLIN.ASM versions 2.00-2.12 |
| Chris Larson | — | Product Marketing Manager | README.txt |
| Don Immerwahr | — | OEM technical support | README.txt |

#### DOS 3.30 / 4.0 (1987-1988) — IBM (Boca Raton)
| Developer | Initials | Role | Evidence Location |
|-----------|----------|------|-------------------|
| J.K. | J.K. | Primary DOS 4.0 boot architect | MSBOOT.ASM lines 25, 52-59 (PTM d52, d48, P1820, D304) |
| Ellen G | — | COMMAND.COM 3.30-4.0 | COMMAND1.ASM lines 112-138 |
| Sunil P. | SUNILP | SMARTDRV, EMM rewrite | SMARTDRV.ASM line 96 (5/13/87), MEMM386.ASM line 405 |
| Greg H. | GREGH | SMARTDRV modifications | SMARTDRV.ASM (1/08/88) |
| David W. | DAVIDW | SMARTDRV modifications | SMARTDRV.ASM (5/13/87) |
| Bill L. | Bill L | DOS 4.0 utilities | FIND.ASM — explicitly noted as "(IBM)" |
| Ed K. | Ed K | PARSE module | PARSE.ASM lines 135-171 (PTMs 1040, 1222, 1672, 2041, 2042) |
| Edwin M. K. | — | COMP utility | COMP.ASM |
| Paul Chan | Paulch | EMM memory management | EMMINIT.ASM line 35 (06/07/88) |
| EMG | EMG | International/country support | MKCNTRY.ASM lines 24-33 (Aug-Oct 1986) |
| CNS | CNS | International/DCR work | MKCNTRY.ASM lines 34-39 (Jul 1987) |
| DCL | DCL | Keyboard layouts | KDFNOW.ASM lines 39-40 (March 8, 1988) |

**IBM-specific evidence in DOS 4.0**:
- `v4.0/src/INC/MS_DATA.ASM` line 160: **"patch space for Boca folks."** — explicit
  reference to IBM Boca Raton development facility
- **PTM numbers** (Problem Tracking Module) used throughout — this is IBM's defect
  tracking system, not Microsoft's or Compaq's
- **DCR numbers** (Design Change Request) — IBM's change management system
- **SCCS markers** in multiple files (e.g., MSBOOT.ASM: `SCCSID = @(#)msboot.asm 1.1
  85/05/13`) — Unix SCCS was IBM's preferred version control

#### Earlier Microsoft developers still present in DOS 4.0
| Developer | Initials | Evidence |
|-----------|----------|---------|
| Chris Peters | ChrisP | MSBOOT.ASM line 4: "Rev 1.0 ChrisP, AaronR and others" |
| Aaron Reynolds | AaronR/ARR | MSBOOT.ASM line 4; SMARTDRV.ASM lines 57-92 (author, 5/86-9/86) |
| Mark Zbikowski | MarkZ | MSBOOT.ASM lines 6, 8: PC/AT enhancements (Rev 3.0, 3.1) |
| LeeAc | LeeAc | MSBOOT.ASM line 12: >32M support (Rev 3.2) |
| MarkT | MarkT | MSBOOT.ASM line 19: COUNT fix (Rev 3.31) |
| MU | MU | COMMAND1.ASM line 110: "Rev 2.50 all the 2.x new stuff -MU" |
| Neilk | Neilk | SMARTDRV.ASM line 63: feature requests (5/27/86) |

#### DOS 6.0 (1993) — Microsoft
| Developer | Initials | Approx. Changes | Primary Areas |
|-----------|----------|------------------|---------------|
| SMR | SMR | 40+ changes | System init, configuration |
| CAS | CAS | 15+ changes | Disk/partition handling |
| HKN | HKN | 49+ changes | Memory/UMB management, EMM386 |
| SR | SR | 4+ changes | System-level |
| JAH | JAH | 5+ changes | IBM-specific fixes |
| MD | MD | 9+ changes | DOS kernel |
| MRW | MRW | Various | Various modules |
| NSM, DBO, PYS | — | Various | Various modules |

Change logs sourced from `astro/bios/msbio.tag`, `astro/dos/dos.tag`, `astro/dev/emm386/emm386.tag`.

**Summary**: Continuous development by Microsoft (DOS 1-2, 5-6) and IBM (DOS 3.3-4.0)
is documented throughout. At no point did development halt. Compaq developers appear
NOWHERE in the core DOS kernel code.

---

### Claim 4: "Every version of Microsoft DOS he sold was, in fact, Compaq DOS"

**Article quote**: "From this point onward, every version of Microsoft DOS he sold was,
in fact, Compaq DOS, with the digital equivalent of its serial numbers filed off."

**Verdict: FALSE**

**Evidence**: This is the most extraordinary claim and is contradicted by multiple
independent sources:

1. **DOS 3.3 was written entirely by IBM** — OS/2 Museum: "Unlike all previous versions,
   DOS 3.3 development was done solely at IBM. Microsoft was busy working on OS/2."
   Also confirmed by Computer History Wiki and IBM PC DOS Wikipedia article.

2. **DOS 4.0 was developed primarily by IBM** — OS/2 Museum: "Version 4.0, like 3.3, was
   developed by IBM with minimal involvement from Microsoft." Source code confirms IBM
   developer names, IBM tracking systems (PTM, DCR), IBM SCCS version control.

3. **DOS 5.0 was written by Microsoft** — Computer History Wiki: "With version 5.0
   Microsoft was again the main developer of MS-DOS." This was Microsoft's response to
   Digital Research's DR DOS 5.0 threatening their market.

4. **DOS 6.0 was written by Microsoft** — Source code shows Microsoft developers (SMR,
   HKN, CAS, etc.) and Microsoft copyright throughout.

5. The MS-DOS 4.0 source code was released jointly by Microsoft and IBM (April 2024),
   with both companies credited. No mention of Compaq.

**Authoritative sources**:
- OS/2 Museum (`os2museum.com`) — Michal Necasek's forensic DOS analysis, the most
  authoritative technical source on DOS internals. Never mentions Compaq as a mainline
  DOS developer.
- Computer History Wiki (`gunkies.org/wiki/MS-DOS`)
- IBM PC DOS Wikipedia article

---

### Claim 5: "This arrangement was secret for almost 40 years"

**Verdict: EXAGGERATED**

Canion published the licensing-back claim in his 2013 book *Open*. That's ~30 years from
1983, not 40. The Edwards article was published June 16, 2025, which would be ~42 years
from 1983, but the "secret" ended in 2013.

The OEM licensing model itself was publicly known — it was common knowledge that OEMs got
source code and customized DOS. The specific licensing-back arrangement being non-public
is plausible, but the article dramatizes this beyond what the evidence supports.

---

### Claim 6: "The deal happened in late 1982"

**Verdict: WRONG DATE — Canion's own book says late 1983**

Canion's book *Open*: "Then in **late 1983**, in a transaction known only to a few people,
Compaq licensed our own version of Microsoft's Disk Operating System (MS-DOS) back to
Microsoft for it to sell to our competitors."

The Edwards article says "late 1982." Since the Compaq Portable didn't launch until
**March 1983**, the book's date of late 1983 makes more sense. HN user TMWNN also flagged
this discrepancy.

---

## Compaq's ACTUAL Contributions to DOS (What the Source Code Shows)

Compaq's fingerprints DO appear in the source code, but in specific subsystems — not the
core OS kernel. This paints a picture of technology partnership, not takeover.

### A. CEMM / EMM386 — Expanded Memory Manager

**DOS 4.0** (`v4.0/src/MEMM/EMM/`):
- 8+ files labeled "CEMM.EXE - COMPAQ Expanded Memory Manager 386 Driver"
- Files: EMMINC.ASM, EMMDATA.ASM, EMMDISP.ASM, EMMP.ASM, EMMSUP.ASM, EMM.H,
  EMMDEF.INC, EMMFUNCT.C
- All carry "(C) Copyright MICROSOFT Corp." — Microsoft copyright on Compaq-originated
  code
- Developer **SBP** made changes (06/25/86, 06/28/86, 07/06/86): name changed from
  CEMM386 to CEMM, handle count modified
- Developer **PC** modified for WIN386, renamed to MEMM (05/06/88)

**DOS 6.0** (`astro/dev/emm386/`):
- **42+ files** with dual Compaq/Microsoft copyrights
- Copyright line: `(C) Copyright COMPAQ Computer Corp. 1986-1991` AND
  `(C) Copyright MICROSOFT Corp. 1986-1991`
- Files include: dmaserv.asm, vdmseg.inc, dmatrap.asm, romxbios.equ, ekbd.asm,
  pic_def.equ, kbd.asm, winemm.inc, emmdata.inc, dmaeisa.asm, rrtrap.asm, wsinit.asm,
  dma.inc, tabdef.asm, driver.inc, pictrap.asm, modesw.asm, romstruc.equ, dmaps2.asm,
  segfix.asm, winsrch.asm, vcpi.asm, segend.asm, and ~20 more

**Critical evidence** — `astro/dev/emm386/dmatrap.asm` line 40:
```
02/27/91 M012     Use ah to test for LONG READ/WRITE in int 13 handler
04/19/91         Added support for > 64K xfer in int 13 handler. Merged from compaq drop
```
This explicitly documents Microsoft merging code received from Compaq into the shared
codebase — a "compaq drop" of improvements.

### B. SMARTDRV — Disk Cache (DOS 4.0)

File: `v4.0/src/DEV/SMARTDRV/SMARTDRV.ASM`

**A20 line fix** (lines 88-89):
```
;       help us on Compaq machines and faster ATs and
;       80386 machines. Thanks to CC of Compaq for fix.
```
This acknowledges a specific Compaq engineer ("CC") who provided the fix for A20 gate
handling on 80386 machines. Aaron Reynolds (ARR) integrated it in revision 1.32
(8/27/86).

**Compaq bad sector handling** (lines 1878-1883):
```
;       the next two instructions were removed because of the way Compaq
;       handles bad sectors. they mark sectors bad not tracks. so in a
;       track there may be good and bad sectors. However our int13 caching
;       system does i/o from disk in tracks and will not get any track with
;       bad sectors. to take care of this, we pass the read to the old int13
;       handler even when there is an int 13 error
```

### C. Hardware Detection and Compatibility

**Machine ID support** (`v4.0/src/MEMM/MEMM/MACH_ID.INC` line 38):
```
RBMI_CompaqPortable     equ     000h
```

**EMM386 Compaq hardware detection** (`astro/dev/emm386/init.asm`):
- `IsCPQ16` procedure with CMOS ID bytes:
  - `CMOS_CPQ_DP38616` = 31h (DeskPro 386/16)
  - `CMOS_CPQ_P386` = 33h (Portable 386)
- `$chkCPQxmm` routine to detect and handle Compaq's HIMEM.SYS

**Video ROM workarounds** (`astro/dev/emm386/winsrch.asm`):
- Special procedures for Compaq Video ROM detection

**Install-time detection** (`astro/install/common/wdrminit.asm` lines 302-357):
- Detects Compaq dual WDCTRL configuration
- Checks CMOS for Compaq-specific settings
- Special handling for Compaq EXTDISK.SYS driver

**Compaq III portable LCD CSD** (`astro/45/beef/drv/csd/src/compaq3.asm`)

### D. OEM Registration

`astro/inc/oemnum.inc`:
```
; IBM DOS             (00)
; Compaq DOS          (01)     <-- OEM #01, right after IBM
; MS Packaged Product (02)
; AT&T DOS            (04)
; Zenith DOS          (05)
; HP DOS              (06)
```

### E. FAT16B / >32MB Partition Support

The article and HN discussion both reference Compaq DOS 3.31's introduction of >32MB
partition support (FAT16B).

**Boot sector revision history** (`v4.0/src/BOOT/MSBOOT.ASM`):
```
Rev 3.0  MarkZ   PC/AT enhancements
Rev 3.2  LeeAc   Modify layout of extended BPB for >32M support
Rev 3.31 MarkT   The COUNT value has a bogus check
Rev 4.00 J.K.    Modified to handle the extended BPB, and 32 bit sector number
                  calculation to enable the primary partition be started beyond
                  32 MB boundary
```

**FDISK integer overflow fix** (`v4.0/src/CMD/FDISK/CONVERT.C` lines 204-208):
```c
#if IBMCOPYRIGHT
    cylinders_out = ((percent_in * total_cylinders) / 100);
#else
    cylinders_out = (unsigned)((ul(percent_in) * ul(total_cylinders)) / 100);
#endif
```
The MS/Clone version uses unsigned long arithmetic to prevent 16-bit overflow at large
disk sizes. The IBM version lacks this protection.

**Assessment**: The >32MB support code was written by Microsoft developers (MarkZ, LeeAc,
J.K.). Compaq DOS 3.31 shipped this support earlier than mainline DOS 4.0, suggesting
Compaq backported it from the development tree (or Microsoft provided it to Compaq
early). The code itself was NOT written by Compaq.

Multiple sources confirm Compaq DOS 3.31 existed:
- OS/2 Museum: "COMPAQ DOS 3.31 in late 1987 pioneered support for 32-bit logical sector
  numbers and thus partitions larger than 32MB."
- Computer History Wiki: "MS-DOS 3.31 was a Compaq only OEM, identical to 3.3 except for
  partitions up to 512 MB."
- WinWorld: "MS-DOS 3.31 was only sold through a few OEMs, mainly Compaq."
- Silicon Underground: "Which company did the work seems to be lost to history."

Other OEMs also shipped DOS 3.31 (Emerson, Vendex HeadStart), suggesting Microsoft
licensed the capability from Compaq or provided it separately.

---

## What Canion's Book Actually Says vs. What the Article Claims

### From *Open* (2013), via Everand preview and Goodreads:

**The licensing-back claim** (Canion's own words):
> "Then in late 1983, in a transaction known only to a few people, Compaq licensed our
> own version of Microsoft's Disk Operating System (MS-DOS) back to Microsoft for it to
> sell to our competitors."

**On working with Microsoft** (from Goodreads quotes):
> "When we asked Microsoft to fix those incompatibilities for which we didn't have the
> source code, we feared that our request would be turned down or at least acted on
> slowly because of their relationship with IBM. We learned, however, that Microsoft was
> happy to work with us and have our help in identifying problems and verifying fixes."

**Goodreads review summary**: "Very little is said of software, beyond an early mention
of an agreement with Microsoft to make a 'compatible DOS'."

**Note**: Canion's own description is more modest than the article's dramatization.
He says Compaq licensed their version back; he does NOT say Microsoft stopped all
development. The article inflates the claim.

### Petri.com reference:
"Only recently revealed in Open: How Compaq Ended IBM's PC Domination... So Compaq
actually licensed its modified version of MS-DOS back to..."

---

## The OEM Licensing Model — How It Actually Worked

**OS/2 Museum on OEM Adaptation Kits** (`os2museum.com/wp/ms-dos-oaks/`):
- OAKs contained full source code to IO.SYS (hardware-dependent layer)
- Larger OEMs like Compaq, Zenith, HP "still chose to customize DOS"
- By MS-DOS 3.2, the PC platform had standardized enough that generic MS-DOS worked on
  most clones without modification

**BetaWiki** (`betawiki.net/wiki/MS-DOS`):
- "Originally, MS-DOS was not available [as retail]... The manufacturer would receive an
  OEM Adaptation Kit, which would then be used to build a custom version"

**DOS Wikipedia** (`en.wikipedia.org/wiki/DOS`):
- "Microsoft provided an OEM Adaptation Kit (OAK) which allowed OEMs to customize the
  device driver code to their particular system."

**Key point**: Large OEMs receiving source code and customizing DOS was STANDARD PRACTICE.
Compaq was not unique in having a source license. What made Compaq special was:
1. Building with IBMVER=TRUE (IBM-compatible mode)
2. Aggressive compatibility testing
3. Contributing enhancements back (EMM, hardware fixes)

---

## The IBM Joint Development Agreement (JDA)

Signed August 1985, the JDA formalized IBM-Microsoft co-development of DOS and OS/2.

**Key consequences**:
- Microsoft obtained rights to IBM-written utilities
- IBM wrote DOS 3.3 and 4.0 while Microsoft focused on OS/2
- After the IBM-Microsoft split (~1991), Microsoft resumed primary DOS development
- Source: `techrights.org/wiki/IBM_Microsoft_Joint_Development_Agreement/`

---

## Development Timeline Summary

| Version | Year | Primary Developer | Evidence |
|---------|------|-------------------|----------|
| DOS 1.0 | 1981 | Tim Paterson / Microsoft | Source code, universal consensus |
| DOS 1.25 | 1982 | Tim Paterson / Microsoft | Source code: "by Tim Paterson" throughout |
| DOS 2.0 | 1983 | Microsoft (Zbikowski, Reynolds, Peters, Panners) | Source code developer names |
| DOS 3.0-3.2 | 1984-86 | Microsoft with IBM | OS/2 Museum |
| DOS 3.3 | 1987 | **IBM** | OS/2 Museum, Computer History Wiki, source code PTMs |
| DOS 3.31 | 1987 | **Compaq** (OEM-specific FAT16B backport) | OS/2 Museum, WinWorld |
| DOS 4.0 | 1988 | **IBM** | Source code: IBM devs, PTMs, "Boca folks", SCCS |
| DOS 5.0 | 1991 | **Microsoft** (response to DR DOS) | Computer History Wiki |
| DOS 6.0 | 1993 | **Microsoft** | Source code developer initials (SMR, HKN, CAS, etc.) |

**At no point was Compaq the primary developer of any mainline DOS version.**

---

## What Probably Actually Happened

1. Compaq had a source code license to MS-DOS, as all major OEMs did through the OEM
   Adaptation Kit program.

2. Compaq built their DOS with IBMVER=TRUE (the "Clone Version"), making it more
   IBM-compatible than the generic IBMVER=FALSE version some other OEMs used.

3. Compaq may have made additional compatibility fixes beyond the standard codebase,
   particularly in IO.SYS and utilities.

4. In late 1983 (per Canion's book), Compaq licensed some customizations back to
   Microsoft. This likely included their improved IO.SYS, utility customizations, and
   compatibility fixes.

5. Microsoft incorporated Compaq's enhancements into the shared codebase. The most
   documented example is FAT16B support from Compaq DOS 3.31 appearing in later versions.
   The "Merged from compaq drop" comment in DOS 6 confirms this practice continued
   through at least 1991.

6. Microsoft NEVER stopped developing DOS. IBM wrote DOS 3.3 and 4.0 (not Compaq).
   Microsoft wrote DOS 5.0 and 6.0 (not Compaq). The claim that "every version of
   Microsoft DOS was Compaq DOS" is flatly contradicted by the source code.

7. Canion's account in *Open* and in interviews with Edwards is told from Compaq's
   perspective. A company founder naturally emphasizes his company's importance. The
   journalist amplified this further into a dramatic narrative that the evidence does not
   support.

---

## Key Source Files for Quick Reference

### Build System / Conditional Compilation
- `v1.25/source/MSDOS.ASM` — IBM/NOT IBM flags in DOS 1.25
- `v2.0/source/STDSW.ASM` — MSVER/IBM flags in DOS 2.0
- `v4.0/src/INC/VERSION.INC` — Three-way build matrix (IBM/Clone/MS)
- `astro/inc/version.inc` — Same system in DOS 6

### Developer Attribution
- `v1.25/source/STDDOS.ASM` — Tim Paterson sole author
- `v1.25/Tim_Paterson_16Dec2013_email.txt` — Paterson confirms history
- `v2.0/source/MSCODE.ASM` — Aaron Reynolds change history
- `v4.0/src/BOOT/MSBOOT.ASM` — Full revision history (ChrisP → AaronR → MarkZ → LeeAc → J.K.)
- `v4.0/src/CMD/COMMAND/COMMAND1.ASM` — Ellen G (DOS 3.30+)
- `v4.0/src/DEV/SMARTDRV/SMARTDRV.ASM` — ARR then SUNILP/GREGH/DAVIDW
- `v4.0/src/INC/MS_DATA.ASM` line 160 — "patch space for Boca folks"
- `v4.0/src/CMD/FIND/FIND.ASM` — Bill L. noted as "(IBM)"

### Compaq-Specific Code
- `v4.0/src/MEMM/EMM/` — CEMM (Compaq Expanded Memory Manager)
- `v4.0/src/DEV/SMARTDRV/SMARTDRV.ASM` lines 88-89 — "Thanks to CC of Compaq"
- `v4.0/src/DEV/SMARTDRV/SMARTDRV.ASM` lines 1878-1883 — Compaq bad sector handling
- `v4.0/src/MEMM/MEMM/MACH_ID.INC` — Compaq Portable machine ID
- `astro/dev/emm386/` — 42+ files with dual Compaq/Microsoft copyright
- `astro/dev/emm386/dmatrap.asm` line 40 — "Merged from compaq drop"
- `astro/dev/emm386/init.asm` — Compaq hardware detection (IsCPQ16)
- `astro/inc/oemnum.inc` — Compaq = OEM #01
- `astro/install/common/wdrminit.asm` — Compaq WDCTRL detection
- `astro/45/beef/drv/csd/src/compaq3.asm` — Compaq III portable LCD

### FAT16B / >32MB Support
- `v4.0/src/BOOT/MSBOOT.ASM` lines 12-25 — Revision history showing MS developers
- `v4.0/src/CMD/FDISK/CONVERT.C` lines 204-208 — Integer overflow fix (Clone vs IBM)
- `v4.0/src/CMD/FDISK/INT13.C` lines 44-48 — Cylinder calculation difference

---

## Online Sources

### Primary Sources
- Edwards article: `https://every.to/the-crazy-ones/the-man-who-beat-ibm`
- Archive: `https://archive.is/A8eQx`
- HN discussion: `https://news.ycombinator.com/item?id=44781343`
- Rod Canion, *Open* (2013): `https://www.everand.com/book/770585674/`

### Authoritative Technical Sources
- OS/2 Museum (Michal Necasek): `https://www.os2museum.com/wp/dos/`
  - DOS 3.3: `os2museum.com/wp/dos/dos-3-3/`
  - MS-DOS OAKs: `os2museum.com/wp/ms-dos-oaks/`
- Computer History Wiki: `https://gunkies.org/wiki/MS-DOS`
- WinWorld (Compaq DOS 3.31): `https://winworldpc.com/product/ms-dos/331`
- BetaWiki: `https://betawiki.net/wiki/MS-DOS`

### Wikipedia
- MS-DOS article has a "COMPAQ-DOS" section citing ONLY the Edwards article (ref 25).
  This should be corrected/expanded with the source code evidence and authoritative
  technical sources listed above.

---

## Summary

The strongest arguments against the article's claims:

1. **The source code itself** — MS-DOS 1.25 through 6.0 all use the same IBMVER
   conditional compilation system to build IBM, Clone, and generic versions from a
   single codebase. This is not what "separate diverging codebases" looks like.

2. **Developer names** — Core DOS code carries initials and names of Microsoft and IBM
   developers across every version. No Compaq developer names appear in any kernel code.

3. **IBM wrote DOS 3.3 and 4.0** — The source code's IBM PTM numbers, "Boca folks"
   reference, SCCS markers, and developer attributions confirm DOS 3.3-4.0 were IBM-led.
   This directly contradicts the claim that Compaq was writing DOS during this period.

4. **Compaq's real contributions are documented** — EMM386 (with dual copyright),
   SMARTDRV fixes (acknowledged by name), hardware compatibility code. These are
   subsystem contributions, not a wholesale takeover.

5. **The article gets its own source wrong** — Canion's book says late 1983; the article
   says late 1982. Canion describes licensing back customizations; the article inflates
   this to "every version of Microsoft DOS was Compaq DOS."

The kernel of truth:
- Compaq did have a source license and customized DOS aggressively
- Compaq did license some enhancements back to Microsoft (confirmed by Canion's book and
  "Merged from compaq drop" in the source)
- Compaq DOS 3.31 did pioneer shipping FAT16B support to end users
- Compaq was OEM #01 and clearly had a privileged partnership with Microsoft

