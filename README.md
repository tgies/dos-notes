# MS-DOS Research Notes

Working research notes from an ongoing investigation of MS-DOS source code across
multiple versions (1.25 through 6.0), covering internals, undocumented APIs, build
systems, hidden features, development history, and more.

## Methodology

Much of this content was produced through LLM-assisted research: automated and
semi-automated source code analysis, cross-referencing, and summarization, with
findings verified through a combination of exhaustive automated checking and manual
review against the original source code and available documentation.

That said, these are working research notes, not an encyclopedia. They reflect
the state of an ongoing investigation and are shared in that spirit. Errors,
omissions, and stale information are possible despite best efforts. If you spot
something wrong, please open an issue.

## Contents

| File | Topic |
|------|-------|
| [dos-source-collection.md](dos-source-collection.md) | Inventory of all known DOS source code, provenance, and completeness |
| [undocumented-apis.md](undocumented-apis.md) | INT 21h "CAVEAT PROGRAMMER" calls, OEM backdoor, IOCTLs, INT 2Fh |
| [hidden-features-dos4.md](hidden-features-dos4.md) | DOS 4 debug modes, dead code, undocumented CONFIG.SYS directives |
| [hidden-features-dos6.md](hidden-features-dos6.md) | DOS 6 hidden switches, SmartDrive debug hooks, FORMAT /AUTOTEST |
| [build-flavor-differences.md](build-flavor-differences.md) | Code differences between MS-DOS and PC-DOS build flavors |
| [development-tags-analysis.md](development-tags-analysis.md) | SCCSID, PTM, AN/AC, and DCR tag systems; developer identification |
| [message-system-localization.md](message-system-localization.md) | Message system internals, BUILDMSG format, localization |
| [dos4-community-research.md](dos4-community-research.md) | Community findings: version identification, forks, known bugs |
| [dos4-retail-reference.md](dos4-retail-reference.md) | Retail disk layouts and build configurations |
| [disk-image-reference.md](disk-image-reference.md) | Disk image creation and emulator testing procedures |
| [compaq-dos-investigation.md](compaq-dos-investigation.md) | Investigation of claims about Compaq's role in DOS development |
