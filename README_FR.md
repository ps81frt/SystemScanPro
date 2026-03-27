# SystemScanPRO

> Scanner forensique Windows et analyseur de malwares — v7.3.0  
> Auteur : [ps81frt](https://github.com/ps81frt) · Dépôt : [github.com/ps81frt/SystemScanPro](https://github.com/ps81frt/SystemScanPro)

---

## ⚠️ Avis important

> **Ce dépôt distribue uniquement l'exécutable compilé (`SystemScanPRO.exe`).**  
> Le code source n'est pas disponible publiquement. Toute propriété intellectuelle est réservée à l'auteur.

- **La rétro-ingénierie, la décompilation, la modification ou la redistribution non autorisée de l'exécutable sont strictement interdites.**
- Les logiciels antivirus (Microsoft Defender et autres) peuvent signaler `SystemScanPRO.exe` comme suspect. Il s'agit d'un faux positif connu affectant les exécutables Python autonomes compilés avec PyInstaller.
- Si votre solution de sécurité met en quarantaine ou bloque le fichier, vous pouvez le soumettre pour examen :
  - **Microsoft Defender :** [https://www.microsoft.com/en-us/wdsi/filesubmission](https://www.microsoft.com/en-us/wdsi/filesubmission)
  - **VirusTotal :** [https://www.virustotal.com/gui/home/upload](https://www.virustotal.com/gui/home/upload)

---

## Version

**Version actuelle : 7.2.0**  
Historique complet disponible dans [CHANGELOG.md](CHANGELOG.md).

---

## Présentation du projet

SystemScanPRO est un scanner forensique Windows autonome développé en Python et distribué sous forme d'un exécutable compilé unique. Il effectue une analyse statique et comportementale approfondie d'un système Windows 10/11 pour détecter les malwares, les mécanismes de persistance, les configurations suspectes et les indicateurs de compromission (IOC).

L'outil est conçu pour fonctionner **100% hors ligne**. Lorsqu'une connexion Internet est détectée, il active optionnellement l'enrichissement (recherches de hash VirusTotal et liens Microsoft Update Catalog pour les erreurs PnP).

Le scanner est organisé en deux phases séquentielles :

| Phase | Description |
|-------|-------------|
| **Scan Registre & Système** | Inspection approfondie du registre, processus en cours, pilotes, services, état réseau |
| **Scan Complémentaire** | Artefacts forensiques, historique d'exécution, état des navigateurs, chemins de persistance, journaux d'événements, partitions, BCD, WMI, connectivité |

Une interface graphique (tkinter) présente les résultats dans des onglets. Après chaque scan, un instantané JSON est automatiquement sauvegardé dans le répertoire de travail.

---

## Caractéristiques techniques

### Modules d'analyse (76 au total)

| Module | Description |
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

### Moteur de détection

Le moteur de détection attribue un score de risque numérique (0–100) à chaque élément basé sur :

- Correspondance avec des noms de processus malveillants connus (RAT, stealers, moteurs adware)
- Correspondance avec 50+ patterns regex (abus LOLBin, indicateurs C2, obfuscation PowerShell, mécanismes de persistance, appels d'​injection DLL)
- Fragments de noms PUP/riskware connus
- Chemin d'exécutable hors des répertoires système de confiance
- Exécutable dans `%TEMP%` ou `AppData\Roaming` (non-Microsoft)
- Éditeur inconnu ou non fiable

Niveaux de risque : `CRITIQUE`, `ELEVE`, `MOYEN`, `INFO`, `FAIBLE`.

### Score global de compromission

Après le scan, l'outil calcule un Score Global de Compromission (0–100) agrégeant :

- Nombre d'éléments malware/PUP détectés
- Processus suspects actifs
- Détections de beaconing C2
- Indicateurs d'altération AV/EDR
- Connexions vers des adresses IP C2 connues
- Processus d'outils offensifs

### Rapport compatible FRST

Un rapport texte compatible Farbar Recovery Scan Tool (FRST) est généré, listant les processus suspects, les entrées autorun, les services, les tâches planifiées et l'état réseau dans un format normalisé compatible avec les workflows courants de suppression de malwares.

### Intégration VirusTotal

L'onglet VirusTotal dédié permet de soumettre les hash SHA256 des exécutables autorun suspects et des processus signalés à l'API VirusTotal v3. Nécessite une clé API valide (niveau gratuit disponible sur `virustotal.com`). La fonctionnalité est optionnelle et active uniquement lorsqu'une connexion Internet est détectée.

### Moteur d'instantanés et de comparaison

Chaque scan produit un fichier d'instantané JSON (`scan_YYYYMMDD_HHMMSS.json`) dans le répertoire de travail. Deux instantanés peuvent être comparés pour identifier les changements entre les scans (lignes ajoutées/supprimées par module).

### Intégration SSCore

Un module compagnon natif optionnel (`tools\SSCore.exe`) fournit une analyse au niveau noyau. Il est recherché au démarrage dans `tools\` et dans le même répertoire que l'exécutable principal. S'il est absent, ce module est silencieusement ignoré.

### Intégration SSProCheck

Un module compagnon .NET 8/10 optionnel (`tools\SSProCheck.exe`) fournit :
- **SigCheck** — Vérification de signature Authenticode via WinVerifyTrust P/Invoke
- **MBR brut** — Lecture MBR brute de 512 octets via CreateFile sur PhysicalDrive avec parsing de la table des partitions
- **Connectivité** — Test de connectivité DNS + TCP + HTTP contre plusieurs points de terminaison

S'il est absent, des implémentations de secours en Python sont utilisées automatiquement.

### Réinitialisation des politiques navigateur

Réinitialisation intégrée des politiques navigateur qui supprime les clés de politique registre de Chrome, Edge et Firefox ainsi que le fichier `policies.json` de Firefox. Exécute `gpupdate /force` après le nettoyage.

---

## Module Fixlist

Le moteur Fixlist traite un fichier `fixlist.txt` utilisant une syntaxe compatible FRST pour effectuer une remédiation ciblée complète. **58 commandes** sont disponibles. Chaque ligne est une action. Les lignes vides et celles commençant par `#` sont ignorées.

### Registre

| Syntaxe | Action |
|---------|--------|
| `HKLM\Software\Chemin\Vers\Cle` | Supprimer la clé de registre entière |
| `HKCU\Software\Chemin\Vers\Cle\|NomValeur` | Supprimer une valeur de registre |
| `HKLM\Software\Chemin\Vers\Cle\|NomVal\|REG_SZ\|Donnee` | Créer / écraser une valeur de registre |

### Fichiers & Dossiers

| Syntaxe | Action |
|---------|--------|
| `C:\chemin\vers\fichier.exe` | Mise en quarantaine puis suppression |
| `C:\chemin\vers\dossier\` | Suppression de l'arborescence |
| `C:\Users\*\AppData\Temp\*.tmp` | Suppression par motif wildcard |
| `C:\chemin\vers\fichier.exe\|Reboot` | Suppression puis redémarrage planifié |
| `File: C:\chemin` | Mise en quarantaine du fichier |
| `Folder: C:\chemin` | Suppression de l'arborescence |
| `EmptyFolder: C:\chemin` | Suppression du contenu, conservation de la racine |

### Services & Tâches

| Syntaxe | Action |
|---------|--------|
| `Service: Nom` | Arrêt + suppression du service |
| `Service: Nom\|Stop` | Arrêt uniquement |
| `Service: Nom\|Disable` | Désactivation au démarrage |
| `StartupService: Nom` | Désactivation au démarrage |
| `Driver: nom.sys` | Arrêt/suppression du service + quarantaine du .sys |
| `Task: \NomTache` | Suppression de la tâche planifiée |
| `Task: \NomTache\|Disable` | Désactivation de la tâche planifiée |
| `StartupFolder: chemin` | Suppression de l'élément de démarrage |

### Extensions & politiques navigateur

| Syntaxe | Action |
|---------|--------|
| `ChromeExt: id` | Suppression de l'extension Chrome |
| `EdgeExt: id` | Suppression de l'extension Edge |
| `FirefoxExt: id` / `FFExt: id` | Suppression de l'extension Firefox |
| `Policies: HKLM\...` | Suppression de la clé de politique |
| `ResetPolicies` | Réinitialisation complète des politiques navigateur |

### Paramètres navigateur

| Syntaxe | Action |
|---------|--------|
| `ResetChromeSearch` | Réinitialiser la recherche par défaut Chrome |
| `ResetEdgeSearch` | Réinitialiser la recherche par défaut Edge |
| `ResetFirefoxSearch` | Réinitialiser la recherche Firefox |
| `FixIESearch` | Restaurer la recherche IE vers Bing |
| `ResetChromeHomepage` | Réinitialiser la page d'accueil Chrome |
| `ResetEdgeHomepage` | Réinitialiser la page d'accueil Edge |
| `ResetFirefoxHomepage` | Réinitialiser la page d'accueil Firefox |
| `KillBHO` | Supprimer toutes les clés BHO |

### Réseau

| Syntaxe | Action |
|---------|--------|
| `CleanDNS` | Vider le cache DNS |
| `ResetWinsock` | Réinitialiser le catalogue Winsock |
| `CleanHosts` / `Hosts: Reset` | Restaurer le fichier hosts par défaut |
| `Hosts: ligne_custom` | Ajouter une ligne personnalisée au fichier hosts |
| `ResetProxy` | Réinitialiser le proxy WinHTTP + WinInet |
| `RemoveProxy:` | Suppression complète du proxy (WinHTTP + WinInet + registre) |
| `FixProxyIE` | Importer les paramètres proxy IE vers WinHTTP |
| `ResetTCPIP` | Réinitialiser la pile TCP/IP |
| `FixDNSServers` | Remettre le DNS en DHCP sur toutes les interfaces |
| `CleanSSL` | Vider le URLcache |
| `FlushARP` | Vider le cache ARP |
| `ResetFirewall` | Réinitialiser le pare-feu aux valeurs par défaut |

### Système

| Syntaxe | Action |
|---------|--------|
| `FixWinlogon` | Restaurer les valeurs par défaut Userinit/Shell |
| `Run: commande` | Exécuter une commande arbitraire |
| `DeletePrefetch: motif` | Supprimer les fichiers Prefetch correspondants |
| `DeleteBamDam: motif` | Supprimer les entrées registre BAM/DAM |
| `Reboot` | Planifier un redémarrage dans 30 secondes |
| `CreateRestorePoint:` | Créer un point de restauration système |
| `EmptyTemp:` | Vider tous les dossiers temporaires |
| `Move: source => dest` | Déplacer fichier/dossier |
| `Copy: source => dest` | Copier fichier/dossier |
| `Zip: source => dest.zip` | Archiver en ZIP |
| `ExportKey: HKLM\...\Cle` | Exporter la clé de registre en .reg dans Quarantine/ |
| `ImportKey: chemin.reg` | Importer un fichier .reg dans le registre |
| `Powershell: commande` / bloc multi-ligne | Exécuter PowerShell (terminer avec `End::`) |

### Blocs Batch & Registre

| Syntaxe | Action |
|---------|--------|
| `StartBatch:` / `EndBatch:` | Exécuter un bloc CMD entre les marqueurs |
| `StartRegedit:` / `EndRegedit:` | Import de fichier .reg inline entre marqueurs |

### Vérification de signature

| Syntaxe | Action |
|---------|--------|
| `VerifySignature: C:\chemin\fichier.exe` | Vérifier la signature Authenticode (via SSProCheck ou PowerShell) |

### Recherche & Forensique

| Syntaxe | Action |
|---------|--------|
| `ListPermissions: C:\chemin` | Afficher les permissions ACL fichier/dossier |
| `SearchAll: motif` | Rechercher dans le registre ET le système de fichiers |
| `SearchFiles: C:\chemin motif` | Recherche récursive de fichiers dans un répertoire |
| `FindFolder: nom` | Localiser des dossiers par nom sur tous les lecteurs |
| `SetPermissions: C:\chemin Utilisateur:Perm` | Modifier les permissions fichier/dossier |
| `TakeOwnership: C:\chemin` | Prendre possession de fichiers/dossiers |
| `UnlockFile: C:\chemin` | Identifier les processus verrouillants, planifier la suppression au redémarrage |

---

## Architecture & Workflow

### Structure

```
SystemScanPRO.py               # Application Python monofichier (~14 300 lignes)
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
└── SystemScanApp (tkinter)    # GUI — 13 onglets, Quick Scan, Recherche,
                               # mode CLI (argparse), barre d'état, progression,
                               # raccourcis clavier (F5, Ctrl+F, Échap)
```

### Flux d'exécution

1. Analyser les arguments CLI (argparse : `--scan`, `--quick`, `--fixlist`, `--output`, `--format`, `--no-gui`)
2. Si mode headless : exécuter le pipeline scan/fixlist, sauvegarder la sortie, quitter
3. Détecter le contexte administrateur (`ctypes.windll.shell32.IsUserAnAdmin`)
4. Détecter la connectivité Internet (connexion TCP vers `8.8.8.8:53`)
5. Détecter l'environnement WinPE/WinRE
6. Initialiser l'interface graphique (`SystemScanApp`)
7. Lancer la vérification de mise à jour en arrière-plan (API GitHub Releases)
8. Au déclenchement du scan (F5 ou bouton) : exécuter tous les modules séquentiellement dans un thread d'arrière-plan
9. Mettre à jour la barre de progression et le libellé d'état à chaque étape (82–99%)
10. Agréger les résultats dans les tampons `registry_scan_text` et `add_text`
11. Exécuter la passe de détection malware sur la sortie combinée
12. Calculer le Score Global de Compromission
13. Générer le rapport FRST
14. Exécuter SSCore (si présent)
15. Exécuter SSProCheck (si présent)
16. Exécuter la détection de beaconing avec verrouillage thread-safe
17. Exécuter le scan d'altération AV/EDR
18. Sauvegarder l'instantané JSON dans le répertoire de travail
19. Afficher tous les résultats dans les onglets de l'interface

---

## Dépendances

| Composant | Requis | Notes |
|-----------|--------|-------|
| Windows OS | **Oui** | Windows 10 ou 11 (64-bit) |
| Python | Non (runtime) | 3.x — intégré dans le `.exe` par PyInstaller |
| psutil | Non (runtime) | Intégré dans le `.exe` |
| reportlab | Non (runtime) | Intégré — requis pour l'export PDF |
| tkinter | Non (runtime) | Intégré dans le `.exe` |
| PowerShell | **Oui** | 5.1+ — utilisé pour les tâches planifiées, Authenticode, PnP, journaux d'événements, partitions, BCD |
| SSCore.exe | Non | Module compagnon optionnel d'analyse noyau (`tools\SSCore.exe`) |
| SSProCheck.exe | Non | Compagnon .NET 8/10 optionnel (`tools\SSProCheck.exe`) — SigCheck, MBR, connectivité |
| Clé API VirusTotal | Non | Optionnelle — requise uniquement pour l'enrichissement de hash |
| Connexion Internet | Non | Optionnelle — active les recherches VirusTotal et les liens catalogue PnP |

> **Note :** Le `.exe` distribué est autonome. Aucune installation Python n'est requise sur la machine cible.

---

## Installation

1. **Télécharger** `SystemScanPRO.exe` depuis la page [Releases](https://github.com/ps81frt/SystemScanPro/releases).
2. **Créer un dossier dédié :**
   ```
   C:\Tools\SystemScanPRO\
   ```
3. **Placer tous les fichiers dans le même répertoire :**
   ```
   C:\Tools\SystemScanPRO\
   ├── SystemScanPRO.exe
   └── tools\
       ├── SSCore.exe        ← module compagnon d'analyse noyau (optionnel)
       └── SSProCheck.exe    ← compagnon .NET 8/10 (SigCheck, MBR, connectivité) (optionnel)
   ```
4. Aucun assistant d'installation ni entrée registre requis. L'outil est entièrement portable.

---

## Utilisation

### Privilèges

> **Les privilèges administrateur sont requis pour un scan approfondi complet.**

Sans droits administrateur, plusieurs modules retourneront des résultats incomplets. L'en-tête affichera `Admin: NO`.

### Lancement

```powershell
cd C:\Chemin\Vers\SystemScanPRO
.\SystemScanPRO.exe
```

Ou clic droit sur `SystemScanPRO.exe` → **Exécuter en tant qu'administrateur**.

### Mode CLI (headless)

```powershell
# Scan complet, sortie TXT
.\SystemScanPRO.exe --scan

# Scan rapide (9 modules critiques), sortie JSON
.\SystemScanPRO.exe --quick --format json

# Exécuter un fixlist
.\SystemScanPRO.exe --fixlist C:\Chemin\Vers\fixlist.txt

# Scan complet avec répertoire et format FRST
.\SystemScanPRO.exe --scan --output C:\Rapports --format frst

# Afficher la version
.\SystemScanPRO.exe --version
```

### Interface

- **F5** — Lancer le scan
- **⚡ Quick Scan** — Exécuter les 9 modules critiques uniquement
- **🔍 Recherche** — Ouvrir le dialogue de recherche (fichiers/registre/dossiers)
- **Ctrl+F** — Focus sur le champ de recherche
- **Échap** — Fermer les fenêtres pop-up

### Fichiers de sortie

Chaque scan sauvegarde un instantané JSON :
```
scan_YYYYMMDD_HHMMSS.json
```

Formats d'exportation : TXT, PDF, JSON, CSV, HTML, TXT compatible FRST, BBCode (forum).

---

## Structure du projet

```
SystemScanPRO/
├── SystemScanPRO.exe         # Exécutable compilé autonome
├── SystemScanPRO.py          # Source (non distribué publiquement)
├── CHANGELOG.md              # Historique des versions
├── README.md                 # Documentation (EN)
├── README_FR.md              # Ce fichier
├── ico/
│   └── scan.ico              # Icône de l'application
└── tools/
    ├── SSCore.exe            # Module compagnon d'analyse noyau (optionnel)
    └── SSProCheck.exe        # Compagnon .NET 8/10 (SigCheck, MBR, connectivité) (optionnel)
```

---

## Comparaison avec des outils similaires

| Fonctionnalité | SystemScanPRO | Autoruns (Sysinternals) | FRST (Farbar) | Malwarebytes |
|----------------|---------------|------------------------|---------------|--------------|
| Scan autorun registre | Oui (50+ emplacements) | Oui | Oui | Partiel |
| Score de risque par élément | Oui | Non | Non | Oui |
| Détection beaconing | Oui (MITRE T1071) | Non | Non | Non |
| Rapport compatible FRST | Oui | Non | Oui (natif) | Non |
| Détection altération AV/EDR | Oui | Non | Non | Non |
| Réinitialisation politiques nav. | Oui | Non | Non | Non |
| Remédiation fixlist | Oui (58 commandes) | Non | Oui (natif, 38) | Non |
| Comparaison / instantané | Oui | Non | Non | Non |
| Intégration VirusTotal | Oui (optionnel) | Oui (optionnel) | Non | Non |
| Analyse niveau noyau | Oui (SSCore) | Oui | Non | Oui |
| 100% hors ligne | Oui | Oui | Oui | Non |
| Interface graphique | Oui (13 onglets) | Oui | Non | Oui |
| Portable (sans installation) | Oui | Oui | Oui | Non |
| Mode CLI headless | Oui (argparse) | Oui | Oui | Non |
| Score global compromission | Oui (0-100) | Non | Non | Non |
| Détection beaconing C2 | Oui | Non | Non | Non |
| Chronologie d'infection | Oui | Non | Non | Non |
| Moteur détection malware | Oui (scoring 0-100) | Non | Non | Oui |
| Audit extensions navigateur | Oui | Non | Non | Non |
| Export PDF | Oui (avec TdM) | Non | Non | Non |
| Quick Scan | Oui (9 modules) | Non | Non | Oui |
| Table des partitions | Oui | Non | Oui | Non |
| Configuration BCD | Oui | Non | Oui | Non |
| Analyse MBR | Oui + lecture brute 512 octets (C#) | Non | Oui | Non |
| Souscriptions WMI | Oui | Non | Oui | Non |
| Known DLLs | Oui | Non | Oui | Non |
| IFEO | Oui | Oui | Oui | Non |
| AppInit_DLLs | Oui | Oui | Oui | Non |
| Associations de fichiers | Oui | Non | Oui | Non |
| SigCheck Authenticode (C#) | Oui (SSProCheck) | Oui (natif) | Non | Non |
| Détection WinPE/WinRE | Oui | Non | Oui | Non |
| Vérification MAJ auto | Oui (GitHub Releases) | Non | Non | Oui |
| Moteur de comparaison | Oui | Non | Non | Non |
| Windows 10/11 uniquement | Oui | Non | Non | Non |

---

## FAQ / Dépannage

**L'outil est signalé par l'antivirus.**  
Faux positif. Les exécutables PyInstaller sont fréquemment signalés. Soumettez le fichier pour examen via les liens ci-dessus.

**Certaines sections du scan sont vides ou affichent « accès refusé ».**  
Relancez en tant qu'administrateur.

**Le module SSCore est ignoré.**  
`SSCore.exe` n'a pas été trouvé dans `tools\`. Assurez-vous que le dossier est présent aux côtés de l'exécutable.

**Le module SSProCheck est ignoré.**  
`SSProCheck.exe` n'a pas été trouvé dans `tools\`. Le fallback Python est utilisé automatiquement pour SigCheck, MBR et la connectivité.

**L'onglet VirusTotal n'affiche aucun résultat.**  
Pas de connexion Internet ou pas de clé API saisie.

---

## Gestion des erreurs & Journalisation

- Chaque module de scan est encapsulé dans un bloc `try/except`. Si un module échoue, un message d'erreur inline (`[Erreur <module> : <exception>]`) est ajouté à la section de sortie de ce module. Le scan continue sans interruption.
- Un logger unifié (`log_info`, `log_warning`, `log_error`, `log_critical`) ajoute des entrées horodatées à une liste `LOGS` en mémoire et les affiche sur stdout.
- Les commandes exécutées via `subprocess` (`run_command`) gèrent le fallback d'encodage entre UTF-8, CP850, CP1252 et Latin-1.
- Les erreurs d'accès refusé de `psutil` (ex. : processus privilégiés) sont silencieusement ignorées.
- Les clés de registre inexistantes sont silencieusement ignorées.

---

## Licence

**Tous droits réservés.**

Copyright © 2024–2026 ps81frt

Ce logiciel et son exécutable compilé sont propriétaires. Le code source n'est pas distribué publiquement.

**Vous ne pouvez pas :**
- Rétro-ingénierer, décompiler ou désassembler l'exécutable
- Modifier, adapter ou créer des œuvres dérivées
- Redistribuer, sous-licencier ou vendre des copies de l'exécutable
- Utiliser ce logiciel à des fins commerciales sans autorisation écrite explicite de l'auteur

**Vous pouvez :**
- Utiliser l'exécutable compilé pour une analyse de sécurité personnelle et non commerciale de systèmes que vous possédez ou êtes autorisé à analyser
- Partager l'exécutable original et non modifié avec attribution à l'auteur

Pour les demandes de licence, contactez l'auteur via GitHub.

---

## Auteur / Contact

**Auteur :** ps81frt  
**GitHub :** [https://github.com/ps81frt](https://github.com/ps81frt)  
**Dépôt :** [https://github.com/ps81frt/SystemScanPro](https://github.com/ps81frt/SystemScanPro)

Pour les rapports de bugs, veuillez ouvrir une issue sur le dépôt GitHub.  
Pour les divulgations de sécurité, contactez l'auteur directement via GitHub.
