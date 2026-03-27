# SystemScanPRO

> Windows forensic and malware analysis scanner — v7.3.1  
> Author: [ps81frt](https://github.com/ps81frt) · Repository: [github.com/ps81frt/SystemScanPro](https://github.com/ps81frt/SystemScanPro)

---

## ⚠️ Important Notice

> **This repository distributes only the compiled executable (`SystemScanPRO.exe`).**  
> The source code is not publicly available. All intellectual property is retained by the author.

- **Unauthorized reverse engineering, decompilation, modification, or redistribution of the executable is strictly prohibited.**
- Antivirus software (Microsoft Defender and others) may flag `SystemScanPRO.exe` as suspicious. This is a known false positive affecting self-contained Python executables compiled with PyInstaller.
- If your security product quarantines or blocks the file, you may submit it for review:
  - **Microsoft Defender:** [https://www.microsoft.com/en-us/wdsi/filesubmission](https://www.microsoft.com/en-us/wdsi/filesubmission)
  - **VirusTotal:** [https://www.virustotal.com/gui/home/upload](https://www.virustotal.com/gui/home/upload)

---

## Version

**Current version: 7.3.1**  
Full history available in [CHANGELOG.md](CHANGELOG.md).

---

## Project Overview

SystemScanPRO is a standalone Windows forensic scanner built in Python and distributed as a single compiled executable. It performs an in-depth static and behavioral analysis of a Windows 10/11 system to detect malware, persistence mechanisms, suspicious configurations, and indicators of compromise (IOCs).

The tool is designed to operate **100% offline**. When an internet connection is detected, it optionally enables enrichment (VirusTotal hash lookups and Microsoft Update Catalog links for PnP errors).

The scanner is organized into two sequential phases:

| Phase | Description |
|-------|-------------|
| **Registry & System Scan** | Deep registry inspection, running processes, drivers, services, network state |
| **Addition Scan** | Forensic artefacts, execution history, browser state, persistence paths, event logs, partitions, BCD, WMI, connectivity |

A graphical interface (tkinter) presents results in labeled tabs. After each scan, a JSON snapshot is saved automatically to the working directory.

---

## Technical Features

### Scan Modules (76 total)

| Module | What it does |
|--------|-------------|
| **System Information** | OS version, hostname, architecture, CPU, RAM, active antivirus (via WMI), firewall status (all profiles) |
| **Running Processes** | Lists all processes with PID, path, user, start time, and a multi-factor risk score. Detects baseline process masquerading |
| **Autorun Entries** | Inspects 50+ registry autorun locations (Run, RunOnce, Winlogon, Session Manager, AppInit_DLLs, KnownDLLs, LSA, NetSh, Print Monitors, Credential Providers, Shell extensions, Winsock, CLSID, Startup folders, etc.) |
| **Loaded Drivers** | Enumerates kernel and file system drivers. Flags drivers loading from `%TEMP%` or `AppData\Local\Temp` as critical |
| **PnP Devices** | Reports PnP devices with configuration errors, warnings, and USB connection/disconnection history |
| **Windows Services** | Lists all Win32 services with start type, service account, and image path. Flags non-standard service accounts |
| **ServiceDLL (svchost hijack)** | Enumerates `ServiceDll` values to detect svchost DLL hijacking |
| **LSA Packages** | Reads Security, Authentication, and Notification Packages from the LSA registry hive |
| **Print Monitors** | Enumerates print monitor and provider DLLs (PrintNightmare detection) |
| **Scheduled Tasks** | Queries all active scheduled tasks. Flags suspicious script extensions and paths |
| **Network Connections** | Lists all active TCP/UDP connections. Checks against known suspicious IPs. Inspects hosts file |
| **Winsock / LSP** | Dumps the Winsock protocol and namespace catalogs |
| **Authenticode Signatures** | Validates digital signatures of autorun executables |
| **DLL Injection** | Inspects memory maps for DLLs loaded from untrusted directories |
| **Alternate Data Streams** | Scans key directories for NTFS alternate data streams |
| **Environment Variables** | Checks PATH entries against known safe directories |
| **DNS Cache** | Dumps and flags suspicious DNS resolver cache entries |
| **Shell Extensions** | Enumerates Icon Overlay, Shell Execute Hooks, and context menu handlers |
| **URL Protocol Handlers** | Lists custom URL protocol handlers |
| **Sensitive Registry** | Checks Winlogon, IFEO, AppInit_DLLs, BHOs, IE search hooks |
| **Installed Programs** | Full installed software list with PUP/adware flagging |
| **User Accounts** | Local accounts, admin group, password policy, recently created accounts |
| **Browser Extensions** | Chrome, Edge, Firefox extension audit with malicious ID check |
| **Browser Policies** | Chrome/Edge/Firefox policy key audit |
| **Root Certificates** | Lists root CAs, flags non-standard authorities |
| **Suspicious Files** | Scans temp/appdata directories for executables |
| **Shortcut Hijack** | Detects hijacked `.lnk` files |
| **Defender Exclusions** | Reads Defender exclusion paths/processes/extensions |
| **Prefetch** | Lists Prefetch files |
| **BITS Jobs** | Enumerates BITS transfer jobs |
| **Firewall Rules** | Lists Windows firewall rules |
| **AppCompat Flags** | Reads compatibility shim database flags |
| **Advanced Persistence** | WMI subscriptions, COM hijack, Netsh helpers, boot execute |
| **AV/EDR Tampering** | Detects antivirus/EDR tampering indicators |
| **Beaconing Detection** | Identifies C2-style regular-interval connections (MITRE T1071) |
| **Event Log (Advanced)** | Queries Security, System, and PowerShell event logs for high-severity events |
| **Restore Points** | Lists System Restore points |
| **Windows Updates** | Lists installed Windows updates |
| **Minidump Analysis** | Scans crash dumps for offensive tool signatures |
| **Amcache** | Program execution history via Amcache.hve |
| **Shimcache** | AppCompatCache execution evidence |
| **Recent Files (MFT)** | Recent executables with double-extension detection |
| **UserAssist** | ROT-13 decoded execution history |
| **MUICache** | Program launch history including deleted programs |
| **BAM/DAM** | Background/Desktop Activity Moderator entries |
| **Recent LNK Files** | Recent shortcut files analysis |
| **PowerShell History** | PowerShell command history |
| **Windows Activation** | Activation status |
| **BIOS / Secure Boot / TPM** | Firmware-level information |
| **Office Persistence** | Office template persistence locations |
| **Credentials (Registry)** | Stored credentials in registry |
| **VM Detection** | Virtual machine/sandbox indicators |
| **Personal Certificates** | User certificate store |
| **Partition Table** | GPT/MBR disk layout, partition types, system/boot/hidden flags |
| **Netsvcs** | Svchost netsvcs group with ServiceDll paths and start types |
| **SFC/DISM Integrity** | CBS.log SFC results, component store health, pending file renames |
| **WMI Repository** | Repository size, non-Microsoft providers, persistent event subscriptions |
| **Boot BCD** | Complete Boot Configuration Data (`bcdedit /enum all`) |
| **MBR/Boot Sector** | Firmware type, SecureBoot, disk signatures, EFI partition contents |
| **Network Detail** | IP configuration, routing table, ARP cache, proxy settings, HOSTS file |
| **WMI Subscriptions** | EventConsumer/Filter/Binding persistence detection |
| **Known DLLs** | Session Manager KnownDLLs audit (DLL hijack vector) |
| **IFEO** | Image File Execution Options Debugger redirects |
| **AppInit DLLs** | DLLs loaded into all user-mode processes (32/64) |
| **Custom CLSID** | COM InProcServer32/LocalServer32 non-standard entries |
| **File Associations** | .exe/.com/.bat/.cmd/.vbs/.js/.ps1/.reg/.scr hijack detection |
| **Safe Mode** | Non-standard SafeBoot Minimal/Network entries |
| **IE Security Zones** | Trusted/Restricted zone policy audit |
| **One Month Files** | Files created/modified last 30 days in System32/drivers |
| **Codecs** | Drivers32 non-standard codec entries |
| **Last RegBack** | Last registry backup date, RegBack folder status |
| **Shortcuts Full** | Desktop, Taskbar, StartMenu .lnk/.url files analysis |
| **Internet Connectivity** | DNS/TCP/HTTP connectivity check (SSProCheck or Python fallback) |
| **MBR Raw** | Raw 512-byte MBR read, partition table parsing, anomaly detection |
| **Infection Timeline** | Correlation timeline across BAM, LNK, PS History, Amcache, Shimcache |

### Detection Engine

The detection engine assigns a numeric risk score (0–100) to every item based on:

- Match against known malicious process names (RATs, stealers, adware engines)
- Match against 50+ regex patterns (LOLBin abuse, C2 indicators, PowerShell obfuscation, persistence mechanisms, DLL injection calls)
- Known PUP/riskware name fragments
- Executable path outside trusted system directories
- Executable in `%TEMP%` or `AppData\Roaming` (non-Microsoft)
- Unknown or untrusted publisher

Risk labels produced: `CRITIQUE`, `ELEVE`, `MOYEN`, `INFO`, `FAIBLE`.

### Global Compromise Score

After scanning, the tool computes a single Global Compromise Score (0–100) aggregating:

- Number of detected malware/PUP elements
- Active suspicious processes
- C2 beaconing detections
- AV/EDR tampering indicators
- Connections to known C2 IP addresses
- Offensive tool processes

### FRST-Compatible Report

A Farbar Recovery Scan Tool (FRST)-compatible text report is generated, listing suspicious processes, autorun entries, services, scheduled tasks, and network state in a normalized format compatible with common malware removal workflows.

### VirusTotal Integration

The dedicated VirusTotal tab allows submitting SHA256 hashes of suspicious autorun executables and flagged processes to the VirusTotal API v3. Requires a valid API key (free tier available at `virustotal.com`). The feature is optional and only active when an internet connection is detected.

### Snapshot and Diff Engine

Each scan produces a JSON snapshot file (`scan_YYYYMMDD_HHMMSS.json`) in the working directory. Two snapshots can be compared to identify changes between scans (added/removed lines per module).

### SSCore Integration

An optional native companion module (`tools\SSCore.exe`) provides kernel-level analysis. It is searched at runtime in `tools\` and in the same directory as the main executable. If absent, this module is silently skipped.

### SSProCheck Integration

An optional .NET 8/10 companion module (`tools\SSProCheck.exe`) provides:
- **SigCheck** — Authenticode signature verification via WinVerifyTrust P/Invoke
- **MBR Raw** — Raw 512-byte MBR read via CreateFile on PhysicalDrive with partition table parsing
- **Connectivity** — DNS + TCP + HTTP connectivity test against multiple endpoints

If absent, Python-based fallback implementations are used automatically.

### Browser Policy Reset

Built-in browser policy reset that removes Chrome, Edge, and Firefox registry policy keys and the Firefox `policies.json` file. Runs `gpupdate /force` after cleanup.

---

## Fixlist Module

The Fixlist engine processes a `fixlist.txt` file using FRST-compatible syntax to perform comprehensive targeted remediation. **58 commands** are available. Each line is one action. Empty lines and lines starting with `#` are ignored.

### Registry

| Syntax | Action |
|--------|--------|
| `HKLM\Software\Path\To\Key` | Delete entire registry key |
| `HKCU\Software\Path\To\Key\|ValueName` | Delete individual registry value |
| `HKLM\Software\Path\To\Key\|ValName\|REG_SZ\|Data` | Create / overwrite registry value |

### Files & Folders

| Syntax | Action |
|--------|--------|
| `C:\path\to\file.exe` | Quarantine then delete file |
| `C:\path\to\folder\` | Remove folder tree |
| `C:\Users\*\AppData\Temp\*.tmp` | Wildcard glob delete |
| `C:\path\to\file.exe\|Reboot` | Delete file then schedule reboot |
| `File: C:\path` | Quarantine file |
| `Folder: C:\path` | Remove folder tree |
| `EmptyFolder: C:\path` | Delete contents, keep root |

### Services & Tasks

| Syntax | Action |
|--------|--------|
| `Service: Name` | Stop + delete service |
| `Service: Name\|Stop` | Stop only |
| `Service: Name\|Disable` | Disable at startup |
| `StartupService: Name` | Disable at startup |
| `Driver: name.sys` | Stop/delete service + quarantine .sys |
| `Task: \TaskName` | Delete scheduled task |
| `Task: \TaskName\|Disable` | Disable scheduled task |
| `StartupFolder: path` | Delete startup item |

### Browser Extensions & Policies

| Syntax | Action |
|--------|--------|
| `ChromeExt: id` | Remove Chrome extension |
| `EdgeExt: id` | Remove Edge extension |
| `FirefoxExt: id` / `FFExt: id` | Remove Firefox extension |
| `Policies: HKLM\...` | Delete policy key |
| `ResetPolicies` | Full browser policy reset |

### Browser Settings

| Syntax | Action |
|--------|--------|
| `ResetChromeSearch` | Reset Chrome default search |
| `ResetEdgeSearch` | Reset Edge default search |
| `ResetFirefoxSearch` | Reset Firefox search |
| `FixIESearch` | Restore IE search to Bing |
| `ResetChromeHomepage` | Reset Chrome homepage |
| `ResetEdgeHomepage` | Reset Edge homepage |
| `ResetFirefoxHomepage` | Reset Firefox homepage |
| `KillBHO` | Delete all BHO keys |

### Network

| Syntax | Action |
|--------|--------|
| `CleanDNS` | Flush DNS cache |
| `ResetWinsock` | Reset Winsock catalog |
| `CleanHosts` / `Hosts: Reset` | Restore default hosts file |
| `Hosts: custom_line` | Add a custom line to the hosts file |
| `ResetProxy` | Reset WinHTTP + WinInet proxy |
| `RemoveProxy:` | Complete proxy removal (WinHTTP + WinInet + registry) |
| `FixProxyIE` | Import IE proxy to WinHTTP |
| `ResetTCPIP` | Reset TCP/IP stack |
| `FixDNSServers` | Set DNS to DHCP on all interfaces |
| `CleanSSL` | Clear URLcache |
| `FlushARP` | Clear ARP cache |
| `ResetFirewall` | Reset firewall to defaults |

### System

| Syntax | Action |
|--------|--------|
| `FixWinlogon` | Restore Userinit/Shell defaults |
| `Run: command` | Execute arbitrary command |
| `DeletePrefetch: pattern` | Delete Prefetch files matching pattern |
| `DeleteBamDam: pattern` | Delete BAM/DAM registry entries |
| `Reboot` | Schedule restart in 30 seconds |
| `CreateRestorePoint:` | Create system restore point |
| `EmptyTemp:` | Empty all temporary folders |
| `Move: source => dest` | Move file/folder |
| `Copy: source => dest` | Copy file/folder |
| `Zip: source => dest.zip` | Archive to ZIP |
| `ExportKey: HKLM\...\Key` | Export registry key to .reg in Quarantine/ |
| `ImportKey: path.reg` | Import .reg file into registry |
| `Powershell: command` / multi-line block | Execute PowerShell (end with `End::`) |

### Batch & Registry Blocks

| Syntax | Action |
|--------|--------|
| `StartBatch:` / `EndBatch:` | Execute a CMD batch block between markers |
| `StartRegedit:` / `EndRegedit:` | Inline .reg file import between markers |

### Signature Verification

| Syntax | Action |
|--------|--------|
| `VerifySignature: C:\path\file.exe` | Verify Authenticode signature (via SSProCheck or PowerShell) |

### Search & Forensics

| Syntax | Action |
|--------|--------|
| `ListPermissions: C:\path` | Display file/folder ACL permissions |
| `SearchAll: pattern` | Search registry AND filesystem for a pattern |
| `SearchFiles: C:\path pattern` | Recursive file search in directory |
| `FindFolder: name` | Locate folders by name across all drives |
| `SetPermissions: C:\path User:Perm` | Modify file/folder permissions |
| `TakeOwnership: C:\path` | Take ownership of files/folders |
| `UnlockFile: C:\path` | Identify locking processes, schedule reboot deletion |

---

## Architecture & Workflow

### Structure

```
SystemScanPRO.py               # Single-file Python application (~14,300 lines)
│
├── Constants & Signatures     # MALICIOUS_PROCESS_NAMES, MALICIOUS_PATTERNS,
│                              # KNOWN_C2_IPS, SUSPICIOUS_DOMAINS,
│                              # AUTORUN_LOCATIONS (50+ registry keys),
│                              # TRUSTED_PUBLISHERS, MALICIOUS_EXT_IDS
│
├── Registry Utilities         # reg_read_values(), reg_read_single(),
│                              # reg_list_subkeys(), reg_delete_value()
│
├── System Utilities           # run_command(), run_cmd_list(),
│                              # normalize_nt_path(), get_file_info(),
│                              # get_file_sha256()
│
├── Detection Engine           # calculate_risk_score(), get_risk_label(),
│                              # is_suspicious_ip(), baseline_check_process()
│
├── Registry Scan Modules (22) # System info, processes, drivers, services...
│
├── Addition Scan Modules (54) # Forensic artefacts, browser, persistence,
│                              # partitions, BCD, WMI, connectivity...
│
├── Global Score Engine        # calculate_global_compromise_score()
│
├── Beaconing Engine           # detect_beaconing() with thread-safe locking
│
├── AV/EDR Tampering           # scan_av_tampering()
│
├── VirusTotal Module          # vt_query_hash(), vt_batch_query()
│
├── FRST Report Builder        # build_frst_report()
│
├── Snapshot Engine            # save/load/compare scan snapshots
│
├── Fixlist Engine (58 cmds)   # Remediation via fixlist.txt
│
├── Browser Policy Reset       # reset_browser_policies()
│
├── SSCore Integration         # sscore_run(), sscore_filter_false_positives()
│
├── SSProCheck Integration     # _run_ssprocheck(), SigCheck/MBR/Connectivity
│
├── Auto-Update Check          # check_update() via GitHub Releases API
│
├── WinPE Detection            # _detect_winpe() — WinPE/WinRE environment
│
└── SystemScanApp (tkinter)    # GUI — 13 tabs, Quick Scan, Search dialog,
                               # CLI mode (argparse), status bar, progress bar,
                               # keyboard shortcuts (F5, Ctrl+F, Esc)
```

### Execution Flow

1. Parse CLI arguments (argparse: `--scan`, `--quick`, `--fixlist`, `--output`, `--format`, `--no-gui`)
2. If headless mode: execute scan/fixlist pipeline, save output, exit
3. Detect administrator context (`ctypes.windll.shell32.IsUserAnAdmin`)
4. Detect internet connectivity (TCP connect to `8.8.8.8:53`)
5. Detect WinPE/WinRE environment
6. Initialize GUI (`SystemScanApp`)
7. Start background auto-update check (GitHub Releases API)
8. On scan trigger (F5 or button): execute all scan modules sequentially in a background thread
9. Update progress bar and status label at each step (82-99%)
10. Aggregate results into `registry_scan_text` and `add_text` string buffers
11. Run malware detection pass over combined output
12. Compute Global Compromise Score
13. Run FRST report builder
14. Run SSCore (if present)
15. Run SSProCheck (if present)
16. Execute beaconing detection with thread-safe locking
17. Execute AV/EDR tampering scan
18. Save JSON snapshot to working directory
19. Render all results in the GUI tabs

---

## Dependencies

| Component | Required | Notes |
|-----------|----------|-------|
| Windows OS | **Yes** | Windows 10 or 11 (64-bit) |
| Python | No (runtime) | 3.x — bundled inside the `.exe` by PyInstaller |
| psutil | No (runtime) | Bundled inside the `.exe` |
| reportlab | No (runtime) | Bundled — required for PDF export |
| tkinter | No (runtime) | Bundled inside the `.exe` |
| PowerShell | **Yes** | 5.1+ — used for scheduled tasks, Authenticode, PnP, event log, partitions, BCD |
| SSCore.exe | No | Optional kernel-level companion module (`tools\SSCore.exe`) |
| SSProCheck.exe | No | Optional .NET 8/10 companion (`tools\SSProCheck.exe`) — SigCheck, MBR, connectivity |
| VirusTotal API key | No | Optional — required only for hash enrichment |
| Internet connection | No | Optional — enables VirusTotal lookups and PnP catalog links |

> **Note:** The distributed `.exe` is self-contained. No Python installation is required on the target machine.

---

## Installation

1. **Download** `SystemScanPRO.exe` from the [Releases](https://github.com/ps81frt/SystemScanPro/releases) page.
2. **Create a dedicated folder:**
   ```
   C:\Tools\SystemScanPRO\
   ```
3. **Place all files in the same directory:**
   ```
   C:\Tools\SystemScanPRO\
   ├── SystemScanPRO.exe
   └── tools\
       ├── SSCore.exe        ← optional kernel analysis companion
       └── SSProCheck.exe    ← optional .NET 8/10 companion (SigCheck, MBR, connectivity)
   ```
4. No installation wizard or registry entry is required. The tool is fully portable.

---

## Usage

### Privileges

> **Administrator privileges are required for a complete deep scan.**

Without administrator rights, several modules will return incomplete results. The header will display `Admin: NO`.

### Running

```powershell
cd C:\Path\To\SystemScanPRO
.\SystemScanPRO.exe
```

Or right-click `SystemScanPRO.exe` → **Run as administrator**.

### CLI Mode (Headless)

```powershell
# Full scan, output to Desktop as TXT
.\SystemScanPRO.exe --scan

# Quick scan (9 critical modules), JSON output
.\SystemScanPRO.exe --quick --format json

# Execute a fixlist
.\SystemScanPRO.exe --fixlist C:\Path\To\fixlist.txt

# Full scan with custom output directory and FRST format
.\SystemScanPRO.exe --scan --output C:\Reports --format frst

# Show version
.\SystemScanPRO.exe --version
```

### Interface

- **F5** — Start scan
- **⚡ Quick Scan** — Run 9 critical modules only
- **🔍 Search** — Open search dialog (files/registry/folders)
- **Ctrl+F** — Focus the search field
- **Escape** — Close pop-up windows

### Output Files

Each scan saves a JSON snapshot:
```
scan_YYYYMMDD_HHMMSS.json
```

Export formats: TXT, PDF, JSON, CSV, HTML, FRST-compatible TXT, BBCode (forum).

---

## Project Structure

```
SystemScanPRO/
├── SystemScanPRO.exe
├── CHANGELOG.md
├── README.md
├── README_FR.md
└── tools/
    ├── SSProCheck.exe
    └── SSCore.exe
```

### Files

| File | Purpose |
|------|---------|
| **SystemScanPRO.exe** | Main scanner (50 MB) — 76 scan modules, risk scoring, FRST reporting, fixlist remediation. Includes embedded known_clsid.json (6,287 Windows COM GUIDs). |
| **CHANGELOG.md** | Version history and release notes (v7.0.0 → v7.3.1). |
| **README.md** | English documentation — features, usage, CLI, fixlist syntax. |
| **README_FR.md** | French documentation (français). |
| **tools/SSProCheck.exe** | Optional .NET 8/10 companion (~500 KB) — Authenticode signature verification, raw 512-byte MBR read, DNS/TCP/HTTP connectivity test. |
| **tools/SSCore.exe** | Optional kernel-level analysis (deprecated v7.3.1+, SSProCheck preferred). |

---

## Comparison with Similar Tools

| Feature | SystemScanPRO | Autoruns (Sysinternals) | FRST (Farbar) | Malwarebytes |
|---------|---------------|------------------------|---------------|--------------|
| Autorun registry scan | Yes (50+ locations) | Yes | Yes | Partial |
| Risk scoring per item | Yes | No | No | Yes |
| Beaconing detection | Yes (MITRE T1071) | No | No | No |
| FRST-compatible report | Yes | No | Yes (native) | No |
| AV/EDR tampering detection | Yes | No | No | No |
| Browser policy reset | Yes | No | No | No |
| Fixlist remediation | Yes (58 commands) | No | Yes (native, 38) | No |
| Scan diff / snapshot | Yes | No | No | No |
| VirusTotal integration | Yes (optional) | Yes (optional) | No | No |
| Kernel-level analysis | Yes (SSCore) | Yes | No | Yes |
| 100% offline capable | Yes | Yes | Yes | No |
| Graphical interface | Yes (13 tabs) | Yes | No | Yes |
| Portable (no install) | Yes | Yes | Yes | No |
| CLI headless mode | Yes (argparse) | Yes | Yes | No |
| Global compromise score | Yes (0-100) | No | No | No |
| C2 beaconing detection | Yes | No | No | No |
| Infection timeline | Yes | No | No | No |
| Malware detection engine | Yes (risk scoring 0-100) | No | No | Yes |
| Browser extension audit | Yes | No | No | No |
| PDF export | Yes (with TOC) | No | No | No |
| Quick Scan | Yes (9 modules) | No | No | Yes |
| Partition table | Yes | No | Yes | No |
| BCD configuration | Yes | No | Yes | No |
| MBR analysis | Yes + raw 512-byte read (C#) | No | Yes | No |
| WMI Subscriptions | Yes | No | Yes | No |
| Known DLLs | Yes | No | Yes | No |
| IFEO | Yes | Yes | Yes | No |
| AppInit_DLLs | Yes | Yes | Yes | No |
| File Associations | Yes | No | Yes | No |
| Authenticode SigCheck (C#) | Yes (SSProCheck) | Yes (native) | No | No |
| WinPE/WinRE detection | Yes | No | Yes | No |
| Auto-update check | Yes (GitHub Releases) | No | No | Yes |
| Snapshot diff engine | Yes | No | No | No |
| Windows 10/11 only | Yes | No | No | No |

---

## FAQ / Troubleshooting

**The tool is flagged by antivirus.**  
False positive. PyInstaller executables are commonly flagged. Submit for review using the links above.

**Some scan sections are empty or show "access denied".**  
Relaunch as administrator.

**SSCore module is skipped.**  
`SSCore.exe` not found in `tools\`. Ensure the folder is present alongside the executable.

**SSProCheck module is skipped.**  
`SSProCheck.exe` not found in `tools\`. Python fallback is used automatically for SigCheck, MBR, and connectivity.

**The VirusTotal tab shows no results.**  
No internet connection or no API key entered.

---

## Error Handling & Logging

- Every scan module is wrapped in a `try/except` block. If a module fails, an inline error message (`[Erreur <module> : <exception>]`) is appended to that module's output section. The scan continues uninterrupted.
- A unified logger (`log_info`, `log_warning`, `log_error`, `log_critical`) appends timestamped entries to an in-memory `LOGS` list and prints them to stdout.
- Commands executed via `subprocess` (`run_command`) handle encoding fallback across UTF-8, CP850, CP1252, and Latin-1 to accommodate different Windows locale configurations.
- Access-denied errors from `psutil` (e.g., privileged processes) are silently skipped.
- Registry keys that do not exist are silently skipped.

---

## License

**All rights reserved.**

Copyright © 2024–2026 ps81frt

This software and its compiled executable are proprietary. The source code is not publicly distributed.

**You may not:**
- Reverse engineer, decompile, or disassemble the executable
- Modify, adapt, or create derivative works
- Redistribute, sublicense, or sell copies of the executable
- Use this software for commercial purposes without explicit written permission from the author

**You may:**
- Use the compiled executable for personal, non-commercial security analysis of systems you own or are authorized to analyze
- Share the original, unmodified executable with attribution to the author

For licensing inquiries, contact the author via GitHub.

---

## Author / Contact

**Author:** ps81frt  
**GitHub:** [https://github.com/ps81frt](https://github.com/ps81frt)  
**Repository:** [https://github.com/ps81frt/SystemScanPro](https://github.com/ps81frt/SystemScanPro)

For bug reports, please open an issue on the GitHub repository.  
For security disclosures, contact the author directly via GitHub.
