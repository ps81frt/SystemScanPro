# CHANGELOG — SystemScanPRO

All notable changes to this project are documented in this file.

---

## [7.3.2] — 2026-03-27

### Fixed
- Fixed command CLI mode

## [7.3.0] — 2026-03-27

### Added
- **known_clsid.json** — base de 6 287 GUIDs COM Windows légitimes (Authenticode vérifié, System32/SysWOW64 uniquement)
- **Tier 0 CLSID** — filtre immédiat sur la base connue avant toute analyse de chemin ou signature
- **Pattern OLE standard** — tous les GUIDs `{*-C000-000000000046}` reconnus comme légitimes sans vérification
- **Auto-update known_clsid.json** — mise à jour automatique depuis GitHub (TTL 7 jours) si connexion disponible
- **_update_clsid.ps1** — script fourni pour régénérer la base known_clsid.json localement (futures mises à jour)

### Changed
- `scan_custom_clsid()` refactorisé en architecture 4 niveaux (Tier 0→4) : faux positifs ~99% réduits
- Version 7.2.1 → 7.3.0

---

## [7.2.1] — 2026-03-27

### Fixed
- Quick Scan GUI now renders results directly in the report pane instead of calling a missing `_display_report()` method
- HTML and Forum BBCode exports no longer fail when no malware scan has been run yet; empty malware datasets are now handled safely
- Headless CLI `--format html` now writes a real HTML report instead of falling back to TXT output

### Changed
- Version bumped from 7.2.0 → 7.2.1
- Report headers, titles, fixlogs, malware exports, and startup logs now consistently use the `SystemScanPRO` product name
- Top-of-file module docstring refreshed to match current capabilities and export formats

---

## [7.2.0] — 2026-03-27

### Added — CLI Headless Mode
- **argparse CLI** with `--scan`, `--quick`, `--fixlist FILE`, `--output DIR`, `--format {txt,frst,html,json}`, `--no-gui`, `--version`
- Full scan, quick scan, and fixlist execution available without GUI

### Added — Quick Scan
- **⚡ Quick Scan** toolbar button — runs 9 critical modules only (system info, processes, autoruns, drivers, services, scheduled tasks, network, malware detection, compromise score)

### Added — Search Dialog
- **🔍 Recherche** toolbar button — opens Toplevel dialog with pattern search across files, registry, and folders with path restriction

### Added — WinPE / WinRE Detection
- `_detect_winpe()` function detecting MININT env, X:\Windows\System32, MiniNT registry key
- Header badge warns when running in WinPE/WinRE environment

### Added — Auto-Update Check
- `check_update()` function querying GitHub Releases API in background thread
- Header notification badge when newer version is available

### Added — SSProCheck C# Companion (.NET 8/10)
- New `tools/SSProCheck/` project (SSProCheck.csproj + Program.cs)
- Multi-target .NET 8.0-windows / .NET 10.0-windows, win-x64, PublishSingleFile
- **SigCheck** — Authenticode signature verification via WinVerifyTrust P/Invoke
- **MBR Raw** — Raw 512-byte MBR read via CreateFile on PhysicalDrive, partition table parsing, anomaly detection
- **Connectivity** — DNS resolve + TCP tests (Google/Cloudflare/Microsoft) + HTTP Microsoft connecttest
- **Batch mode** — `sigcheck-batch` for multiple files, JSON output
- `_SSPROCHECK_EXE` discovery and `_run_ssprocheck()` helper with JSON parsing

### Added — New Scan Modules (76 total, up from 73)
- **`scan_shortcuts_full()`** — Desktop, Taskbar, StartMenu .lnk/.url files analysis
- **`scan_internet_connectivity()`** — DNS/TCP/HTTP connectivity test (via SSProCheck C# or Python fallback)
- **`scan_mbr_raw()`** — Raw 512-byte MBR read with partition table parsing (via SSProCheck C#)

### Added — New Fixlist Commands (58 total, up from 53)
- **`StartBatch:` / `EndBatch:`** — Execute a CMD batch block between markers
- **`StartRegedit:` / `EndRegedit:`** — Inline .reg file import between markers
- **`Hosts: custom_line`** — Add a custom line to the hosts file
- **`VerifySignature: path`** — Verify Authenticode signature (via SSProCheck or PowerShell fallback)
- **`RemoveProxy:`** — Complete proxy removal (WinHTTP + WinInet + registry)

### Changed
- Version bumped from 7.1.0 → 7.2.0
- FIXLIST_SYNTAX_HELP updated to v7.2 with all 58 commands documented
- Addition scan steps expanded from 51 to 54 modules
- `build_frst_report()` Addition.txt now includes 3 new v7.2 modules
- `_SECTION_MAP` expanded with ~15 new entries for v7.1 + v7.2 modules
- About tab updated with v7.2.0 features, CLI usage, companion tools documentation
- README EN/FR fully rewritten for v7.2.0
- Docstring updated to reflect all v7.2.0 features

---

## [7.1.0] — 2026-03-27

### Added — New Scan Modules (73 total, up from 62)
- **`scan_wmi_subscriptions()`** : WMI EventConsumer/Filter/Binding persistence detection
- **`scan_known_dlls()`** : Session Manager KnownDLLs registry audit (DLL hijack vector)
- **`scan_ifeo()`** : Image File Execution Options — Debugger redirect detection
- **`scan_appinit_dlls()`** : AppInit_DLLs loaded into all user-mode processes (32/64-bit)
- **`scan_custom_clsid()`** : Custom COM CLSID InProcServer32/LocalServer32 persistence
- **`scan_file_associations()`** : File association hijack detection (.exe, .com, .bat, .cmd, .vbs, .js, .ps1, .reg, .scr)
- **`scan_safe_mode()`** : Non-standard SafeBoot Minimal/Network entries
- **`scan_ie_zones()`** : IE Security Zones — Trusted/Restricted zone policy audit
- **`scan_one_month_files()`** : Files created/modified last 30 days in System32/drivers
- **`scan_codecs()`** : Installed codec registry audit (Drivers32 non-standard entries)
- **`scan_last_regback()`** : Last registry backup date and RegBack folder status

### Added — New Fixlist Commands (53 total, up from 45)
- **`CreateRestorePoint:`** — Create a system restore point before applying fixes
- **`EmptyTemp:`** — Empty all system and user temporary folders
- **`Move: source => dest`** — Move a file or folder
- **`Copy: source => dest`** — Copy a file or folder
- **`Zip: source => dest.zip`** — Archive to ZIP
- **`ExportKey: HKLM\...\Key`** — Export a registry key to .reg file in Quarantine/
- **`ImportKey: path.reg`** — Import a .reg file into the registry
- **`Powershell: command`** — Execute inline PowerShell or multi-line block (terminated by `End::`)

### Changed
- Version bumped from 7.0.0 → 7.1.0
- Fixlist engine loop converted from for-each to index-based iteration (supports Powershell multi-line blocks)
- FIXLIST_SYNTAX_HELP updated with all 53 commands
- Addition scan steps expanded from 40 to 51 modules
- build_frst_report Addition.txt now includes all 11 new modules

---

## [7.0.0] — 2026-03-27

### Fork
- SystemScanPRO is forked from SystemScan v6.3.0 with all improvements applied.
- Renamed to **SystemScanPRO** with new branding, version 7.0.0.

### Bug Fixes (HIGH)
- **`_close_connection()`** : Fixed wrong column index for local address (`vals[2]` → `vals[1]`)
- **`build_infection_timeline()`** : Removed duplicate BAM/DAM blocks (scanned 3× → 1×), duplicate PS History (3× → 1×), duplicate LNK (2× → 1×); fixed double-comma in PS `-replace` regex
- **`_malware_detail()`** : Fixed treeview column order mismatch — code read `(type, name, detail, score)` but columns are `(type, score, name, detail)` → reordered to `(type, score, name, detail)`

### Bug Fixes (MEDIUM)
- **`_refresh_drivers()`** : Rewrote error device scanner block to use fresh local variables instead of stale outer-scope references (`desc`, `cls`, `svc`, `hw_str` → `_desc`, `_cls`, `_svc`, `_hw_str`)
- **`scan_advanced_persistence()`** : Fixed inverted WMI EventConsumer detection logic — was checking for class name in output instead of actual consumer presence
- **`malware_detector_to_text()`** : Fixed `pup_tasks` KeyError — items only have `detail` key but code accessed `item['name']` → `item.get('name', item.get('detail', '?'))`
- **`_run_cleaner_inline()`** : Removed duplicate recycle bin deletion block
- **`_refresh_eventlog()`** : Fixed double comma in PowerShell `-replace` regex (`,,` → `,`)
- **`record_connection_for_beaconing()` / `detect_beaconing()`** : Added proper thread-safe locking around `_beacon_history` access using `_beacon_lock`

### Bug Fixes (LOW)
- Removed all redundant local `import re as _re`, `import re as _re2`, `import re as _re3` — uses module-level `re` throughout
- **`normalize_nt_path()`** : Removed internal `import re as _r` — uses module-level `re`
- **`addition_steps`** : Fixed progress percentages that exceeded 100% — recalculated to stay within 82-99% range
- Added module-level `import glob` instead of per-iteration imports

### Added — New Scan Modules (FRST Parity+)
- **`scan_partition_table()`** : GPT/MBR partition analysis via diskpart and WMI (Get-Disk, Get-Partition)
- **`scan_netsvcs()`** : Complete listing of svchost netsvcs group with ServiceDll paths and start types
- **`scan_sfc_dism()`** : SFC/DISM integrity check via CBS.log analysis, component store health, pending renames
- **`scan_wmi_repository()`** : WMI repository size/health, non-Microsoft providers, persistent event subscriptions
- **`scan_bcd_config()`** : Boot Configuration Data enumeration (`bcdedit /enum all`)
- **`scan_mbr_analysis()`** : Firmware type (UEFI/Legacy), SecureBoot status, disk signatures, EFI partition contents
- **`scan_network_detail()`** : Full IP config, routing table, ARP cache, proxy settings, HOSTS file content

### Added — New Fixlist Commands (45 total, up from 38)
- **`ListPermissions:`** — Display ACL permissions of any file/folder
- **`SearchAll:`** — Search for a pattern across registry AND filesystem
- **`SearchFiles:`** — Recursive file search by name pattern in a given directory
- **`FindFolder:`** — Locate folders by name across all drive letters
- **`SetPermissions:`** — Modify file/folder permissions (`icacls /grant`)
- **`TakeOwnership:`** — Take ownership of locked files/folders
- **`UnlockFile:`** — Identify processes locking a file and schedule deletion on reboot

### Changed
- Version bumped from 6.3.0 → 7.0.0
- Module-level `threading.Lock()` added (`_beacon_lock`) for thread-safe beaconing
- Addition scan steps expanded from 33 to 40 modules
- All progress percentages recalculated for smooth 82-99% progression

---

## [6.3.0] — 2026-03-26

(See SystemScan CHANGELOG for prior history)
