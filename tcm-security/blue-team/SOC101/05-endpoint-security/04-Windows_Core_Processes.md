# Windows Core Processes

## Tools Overview

### Task Manager (`taskmgr.exe`)
- Built-in Windows tool — accessible via `Ctrl+Shift+Esc` or `Ctrl+Alt+Del`
- **Processes tab**: shows name, PID, CPU, memory, disk, network usage
- **Details tab**: more granular view with PID, status, username, and description
- **Services tab**: lists running/stopped services and their PIDs
- Tip: right-click column headers to add **Command Line** and **Image Path Name** columns — extremely useful for spotting anomalies

### Process Explorer (Sysinternals Suite)
- Download from: [https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer)
- Part of the **Microsoft Sysinternals Suite** — free and trusted
- Displays **full process tree** with parent-child relationships (color-coded)
- Right-click any `.exe` → **Properties**:
  - **Image tab**: full executable path, version info, description
  - **Strings tab**: shows hardcoded strings inside the binary (useful for malware analysis)
  - **TCP/IP tab**: active network connections per process
- Can **verify process signatures** against VirusTotal (Options → VirusTotal.com)
- Highlights processes with **no verified signature** in pink/red

### Process Hacker (alternative to Process Explorer)
- Open-source, download from: [https://processhacker.sourceforge.io/](https://processhacker.sourceforge.io/)
- Similar to Process Explorer with added memory inspection and thread analysis capabilities

---

## Process Tree Overview

The following tree represents the standard, **expected** process hierarchy on a healthy Windows system. Any deviation from this structure should be treated as suspicious and investigated.

```
System (PID 4)
└── smss.exe  [Master Session Manager]
    │
    ├── smss.exe  [Session 0 - System/Services]
    │   ├── csrss.exe         (Session 0)
    │   └── wininit.exe
    │       ├── services.exe
    │       │   ├── svchost.exe  -k DcomLaunch
    │       │   ├── svchost.exe  -k NetworkService
    │       │   ├── svchost.exe  -k LocalService
    │       │   └── svchost.exe  -k ...  (many instances)
    │       └── lsass.exe
    │
    └── smss.exe  [Session 1 - Interactive User]
        ├── csrss.exe         (Session 1)
        └── winlogon.exe
            └── userinit.exe
                └── explorer.exe
                    └── [user applications]
```

> **Session 0** = system/services session (no user interaction)  
> **Session 1+** = interactive user sessions (one per logged-in user)

---

## Core Processes

---

### System
**Description:** The System process is not a normal user-mode process. It represents the Windows kernel itself and is the home of all kernel-mode threads.

**Key responsibilities:**
- Runs in **kernel mode** — highest privilege level
- Manages **CPU scheduling**, memory, and disk I/O
- Loads and runs **device drivers**
- Handles **hardware abstraction** via HAL (Hardware Abstraction Layer)
- The kernel executable is `ntoskrnl.exe` (NT OS Kernel)

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `None` or `C:\Windows\System32\ntoskrnl.exe` |
| PID | **Always `4`** |
| Parent Process | `None` or `System Idle Process (PID 0)` |
| Number of Instances | **Exactly `1`** |
| User Account | `NT AUTHORITY\SYSTEM` |
| Start Time | At boot |

**🔴 Red Flags:**
- PID is anything other than `4`
- More than one `System` process
- Running from any path other than `ntoskrnl.exe`
- Has a visible parent process

---

### smss.exe — Session Manager Subsystem
**Full name:** Windows Session Manager  
**Executable:** `%SystemRoot%\System32\smss.exe`

**Key responsibilities:**
- **First user-mode process** created by the kernel at boot
- Initializes the Windows environment before user interaction
- Creates and manages **Sessions 0 and 1** (and additional sessions for each logged-in user)
- Launches **csrss.exe** and either **wininit.exe** (Session 0) or **winlogon.exe** (Session 1+)
- Sets up the **paging file** (`pagefile.sys`) and loads known DLLs into shared memory
- Child `smss.exe` instances **self-terminate** after spawning their children — only the master remains

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `%SystemRoot%\System32\smss.exe` |
| Parent Process | `System (PID 4)` |
| Number of Instances | `1` master + `1` child per session (children exit quickly) |
| User Account | `NT AUTHORITY\SYSTEM` |
| Start Time | Within seconds of boot |

**🔴 Red Flags:**
- Parent process is anything other than `System (PID 4)`
- Running from outside `C:\Windows\System32\`
- More than one persistent `smss.exe` instance
- Unusual command line arguments

---

### csrss.exe — Client/Server Runtime Subsystem
**Full name:** Client/Server Runtime Subsystem  
**Executable:** `%SystemRoot%\System32\csrss.exe`

**Key responsibilities:**
- Manages **console windows** (cmd.exe, PowerShell) — handles creation and deletion
- Imports essential DLLs for the **Windows API**
- Handles GUI tasks during the **shutdown sequence**
- Maintains a list of **active processes and threads** used by the OS
- Critical system process — **terminating it causes a Blue Screen of Death (BSOD)**

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `%SystemRoot%\System32\csrss.exe` |
| Parent Process | `smss.exe` (will appear as non-existent since smss.exe exits) |
| Number of Instances | **Minimum 2** (one per session: Session 0 + Session 1) |
| User Account | `NT AUTHORITY\SYSTEM` |
| Start Time | Within seconds of boot |

**🔴 Red Flags:**
- Running from outside `C:\Windows\System32\`
- A visible parent process (smss.exe should have exited already)
- Only one instance (should always be at least 2)
- Name variations: `cssrs.exe`, `crss.exe`, `csrss.exe` located in a non-standard path

> ⚠️ Because `csrss.exe` is critical, some malware specifically targets this name to hide itself. Always verify the **path** and **digital signature**.

---

### wininit.exe — Windows Initialization
**Full name:** Windows Initialization Process  
**Executable:** `%SystemRoot%\System32\wininit.exe`

**Key responsibilities:**
- Starts **Session 0** — the isolated system session introduced in Windows Vista as a security improvement
- Spawns three critical child processes:
  - `services.exe` — Service Control Manager
  - `lsass.exe` — Local Security Authority
  - `lsm.exe` (on older systems) — Local Session Manager
- Sets up the **`%TEMP%`** environment variable for the system session

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `%SystemRoot%\System32\wininit.exe` |
| Parent Process | `smss.exe` (will appear as non-existent) |
| Number of Instances | **Exactly `1`** |
| User Account | `NT AUTHORITY\SYSTEM` |
| Start Time | Within seconds of boot |

**🔴 Red Flags:**
- More than one instance
- A visible, running parent process
- Running from outside `System32`
- Name variations: `wininit.exe` found in `%TEMP%` or Desktop paths

---

### services.exe — Service Control Manager
**Full name:** Service Control Manager (SCM)  
**Executable:** `%SystemRoot%\System32\services.exe`

**Key responsibilities:**
- Loads and manages all **Windows services** (background processes)
- Handles **starting, stopping, pausing, and resuming** services
- Reads service configurations from the registry:  
  `HKLM\SYSTEM\CurrentControlSet\Services`
- Sets the **Last Known Good Configuration** registry key on successful boot
- Direct parent of all `svchost.exe` instances

**Registry key of interest:**
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services
```
Each subkey represents a service, including its `ImagePath`, `Start` type, and `ObjectName` (the user account it runs under).

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `%SystemRoot%\System32\services.exe` |
| Parent Process | `wininit.exe` |
| Number of Instances | **Exactly `1`** |
| User Account | `NT AUTHORITY\SYSTEM` |
| Start Time | Within seconds of boot |

**🔴 Red Flags:**
- More than one instance
- Parent is anything other than `wininit.exe`
- Unexpected child processes (other than `svchost.exe`, `dllhost.exe`, etc.)
- Direct network connections originating from `services.exe` (unusual)

---

### svchost.exe — Service Host
**Full name:** Service Host Process  
**Executable:** `%SystemRoot%\System32\svchost.exe`

**Key responsibilities:**
- Hosts **Windows services implemented as DLLs** (not standalone `.exe` files)
- Multiple instances run simultaneously — each groups related services using the **`-k` parameter**
- The actual service DLL is defined in the registry under the service's `Parameters\ServiceDll` key
- Common `-k` groups:

| `-k` Parameter | Services Hosted |
|---|---|
| `DcomLaunch` | DCOM Server Process Launcher, Power |
| `NetworkService` | DNS Client, DHCP Client |
| `LocalService` | Windows Time, SSDP Discovery |
| `netsvcs` | Task Scheduler, Windows Update, Themes, etc. |
| `rpcss` | Remote Procedure Call (RPC) |
| `LocalSystemNetworkRestricted` | Windows Audio, Plug and Play |

**How to verify the `-k` flag:**
1. Open Task Manager → Details tab
2. Right-click column headers → Add **Command Line** column
3. Every legitimate `svchost.exe` should show: `C:\Windows\System32\svchost.exe -k <GroupName>`

**Registry key to check which DLL a service hosts:**
```
HKLM\SYSTEM\CurrentControlSet\Services\<ServiceName>\Parameters\ServiceDll
```

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `%SystemRoot%\System32\svchost.exe` |
| Parent Process | `services.exe` |
| Number of Instances | Many (`10+` is normal) |
| User Account | `SYSTEM`, `NETWORK SERVICE`, `LOCAL SERVICE`, or a local account |
| Start Time | At boot or when the associated service starts |
| Command Line | Must include `-k <GroupName>` |

**🔴 Red Flags:**
- Running **without** the `-k` parameter
- Parent is **not** `services.exe` (e.g., launched from a user process or `explorer.exe`)
- Running from outside `C:\Windows\System32\`
- Name variations: `svch0st.exe`, `scvhost.exe`, `svchost.exe` in `%TEMP%` or `%APPDATA%`
- Establishing unexpected **outbound network connections**
- Running under a **standard user account** (not SYSTEM / LOCAL SERVICE / NETWORK SERVICE)

> ⚠️ `svchost.exe` is the **most commonly abused** Windows process by malware. Always check path, parent, and command line arguments.

---

### lsass.exe — Local Security Authority Subsystem Service
**Full name:** Local Security Authority Subsystem Service  
**Executable:** `%SystemRoot%\System32\lsass.exe`

**Key responsibilities:**
- **Authenticates users** — processes logon requests and verifies credentials
- Enforces **local security policies** (password policies, account lockout, etc.)
- Generates **access tokens** for authenticated users
- Writes to the **Windows Security Event Log**
- Manages **credential caching** — stores credentials in memory (primary target of credential dumping attacks)
- Works with authentication packages: `msv1_0`, `kerberos`, `wdigest`, `tspkg`, `livessp`

**Security Event Log IDs (important for SOC analysts):**

| Event ID | Description |
|---|---|
| `4624` | Successful account logon |
| `4625` | Failed account logon |
| `4634` | Account logoff |
| `4648` | Logon with explicit credentials (e.g., runas) |
| `4672` | Special privileges assigned to new logon |
| `4768` | Kerberos TGT requested |
| `4776` | NTLM authentication attempted |

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `%SystemRoot%\System32\lsass.exe` |
| Parent Process | `wininit.exe` |
| Number of Instances | **Exactly `1`** |
| User Account | `NT AUTHORITY\SYSTEM` |
| Start Time | Within seconds of boot |

**🔴 Red Flags:**
- More than one `lsass.exe` instance
- Parent is anything other than `wininit.exe`
- Running from outside `C:\Windows\System32\`
- Name variations (very common attack): `lsaas.exe`, `lssas.exe`, `lsas.exe`, `isass.exe` (capital **I** instead of lowercase **l**)
- Any process opening a memory handle to `lsass.exe` with `PROCESS_VM_READ` rights (credential dumping)
- Abnormally high CPU usage

> ⚠️ **MITRE ATT&CK T1003.001** — `lsass.exe` memory is the primary target for **credential dumping** tools like Mimikatz and ProcDump. In a SOC environment, alert on any process that opens a handle to `lsass.exe` with read memory access rights.

---

### winlogon.exe — Windows Logon
**Full name:** Windows Logon Application  
**Executable:** `%SystemRoot%\System32\winlogon.exe`

**Key responsibilities:**
- Manages the **interactive user logon/logoff** process
- Responds to the **Secure Attention Sequence (SAS)**: `Ctrl+Alt+Del`
- Loads the user's **registry hive** (`NTUSER.DAT`) on login
- Loads the user's **shell** (typically `explorer.exe`) via `userinit.exe`
- Handles **screen lock** and interacts with **Credential Providers** (the login UI)
- Manages interactions with **Winlogon Notification Packages** (authentication providers)

**Registry keys — common persistence targets:**
```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
  → Shell    = explorer.exe        (attackers may append a second binary here)
  → Userinit = C:\Windows\system32\userinit.exe,   (check for extra entries)
```

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `%SystemRoot%\System32\winlogon.exe` |
| Parent Process | `smss.exe` (will appear as non-existent) |
| Number of Instances | `1` per interactive user session |
| User Account | `NT AUTHORITY\SYSTEM` |
| Start Time | Within seconds of boot |

**🔴 Red Flags:**
- Parent process is visible and still running (smss.exe should have exited)
- Running from outside `C:\Windows\System32\`
- Unusual child processes other than `userinit.exe` or `LogonUI.exe`
- Modified `Shell` or `Userinit` registry values pointing to additional executables
- Name variations: `winl0gon.exe`, `winlogon.exe` in non-standard paths

> ⚠️ **MITRE ATT&CK T1547.004** — Attackers abuse the **Winlogon Helper DLL** to achieve persistence by modifying the `Shell` and `Userinit` registry values under `HKLM\...\Winlogon`.

---

### explorer.exe — Windows Explorer
**Full name:** Windows Explorer  
**Executable:** `%SystemRoot%\explorer.exe`

**Key responsibilities:**
- Provides the **graphical user interface** for the file system (File Explorer)
- Manages the **desktop, taskbar, Start Menu, and notification area (system tray)**
- Acts as the default **shell** for interactive user sessions
- Spawns user-launched applications as child processes
- Handles **drag-and-drop**, file operations, and shell extensions (context menu handlers, etc.)

**Expected properties:**

| Field | Expected Value |
|---|---|
| Image Path | `%SystemRoot%\explorer.exe` (note: **not** System32!) |
| Parent Process | `userinit.exe` (will appear as non-existent after login) |
| Number of Instances | `1` per logged-in user (more if running in separate process mode) |
| User Account | The **logged-in user's account** (never SYSTEM) |
| Start Time | When the interactive user session began |

**🔴 Red Flags:**
- Running under the `SYSTEM` account (should always run as the logged-in user)
- Parent process is visible and still running (userinit.exe should have exited)
- Running from outside `C:\Windows\` — especially from `System32`, `%TEMP%`, or `%APPDATA%`
- Unexpected **outbound network connections**
- Injected malicious code (use Process Explorer's memory map to inspect loaded modules)
- Name variations: `expl0rer.exe`, `explorer.exe` in non-standard paths

> ⚠️ **MITRE ATT&CK T1055** — `explorer.exe` is a common target for **process injection** because it runs with user-level privileges, is always present, and is generally trusted by security tools.

---

## Process Investigation Checklist

When analyzing a suspicious process, systematically work through the following:

### 1. Identity Check
- [ ] Is the **process name** exactly correct? (watch for typos: `svch0st`, `lsas`, `isass`)
- [ ] Is the **image path** in the expected directory?
- [ ] Is the file **digitally signed** by Microsoft? (verify in Process Explorer)

### 2. Hierarchy Check
- [ ] Is the **parent process** correct?
- [ ] Are the **child processes** expected for this parent?
- [ ] Does the **session** (0 or 1+) match expectations?

### 3. Instance Count Check
- [ ] Is the **number of instances** within the expected range?
- [ ] Are there duplicate processes where only one should exist? (`lsass`, `wininit`, `services`)

### 4. Account & Privilege Check
- [ ] Is the **user account** correct for this process?
- [ ] Does the process have **unexpected elevated privileges**?

### 5. Command Line Check
- [ ] Are the **command line arguments** expected? (e.g., `-k` flag for `svchost.exe`)
- [ ] Are there **unusual flags or parameters**?

### 6. Network Activity Check
- [ ] Is the process making **unexpected network connections**?
- [ ] What are the **destination IPs/domains**? Are they known-good?
- [ ] Use the **TCP/IP tab** in Process Explorer to check live connections

### 7. Behavioral Analysis
- [ ] Is **CPU/memory usage** abnormally high?
- [ ] Are there **suspicious loaded DLLs**? (check with Process Explorer → DLLs view)
- [ ] Are there **strings** in the binary that seem out of place? (Strings tab in Process Explorer)

---

## Common Masquerading Techniques

Attackers often disguise malicious processes by mimicking legitimate Windows process names. Common techniques:

| Technique | Malicious Example | Legitimate Name |
|---|---|---|
| Character substitution (I vs l) | `isass.exe` | `lsass.exe` |
| Zero substitution | `svch0st.exe`, `winl0gon.exe` | `svchost.exe`, `winlogon.exe` |
| Extra/missing letter | `scvhost.exe`, `csrsss.exe` | `svchost.exe`, `csrss.exe` |
| Wrong directory | `C:\Users\Public\svchost.exe` | `C:\Windows\System32\svchost.exe` |
| Spoofed parent PID (PPID spoofing) | Malware fakes its PPID to appear as a child of `wininit.exe` | — |
| DLL sideloading | Legitimate process loads a malicious DLL from a writable path | — |
| Process hollowing | Legitimate process shell is launched suspended, then its memory is replaced with malicious code | — |

> **Rule of thumb:** If the name looks right but the **path or parent is wrong** — it's malicious. Real Windows processes always live in `%SystemRoot%\System32\` (or `%SystemRoot%\` for `explorer.exe`).

---

## Quick Reference Table

| Process | Parent | Instances | Path | User |
|---|---|---|---|---|
| `System` | None / PID 0 | 1 | `ntoskrnl.exe` | SYSTEM |
| `smss.exe` | System (PID 4) | 1 master | `System32\smss.exe` | SYSTEM |
| `csrss.exe` | smss (exited) | 2+ | `System32\csrss.exe` | SYSTEM |
| `wininit.exe` | smss (exited) | 1 | `System32\wininit.exe` | SYSTEM |
| `services.exe` | wininit | 1 | `System32\services.exe` | SYSTEM |
| `svchost.exe` | services | 10+ | `System32\svchost.exe` | SYS / NET SVC / LOCAL SVC |
| `lsass.exe` | wininit | 1 | `System32\lsass.exe` | SYSTEM |
| `winlogon.exe` | smss (exited) | 1/session | `System32\winlogon.exe` | SYSTEM |
| `explorer.exe` | userinit (exited) | 1+/user | `explorer.exe` | Logged-in User |

---

## MITRE ATT&CK References

| Technique ID | Name | Relevant Process |
|---|---|---|
| [T1003.001](https://attack.mitre.org/techniques/T1003/001/) | LSASS Memory – Credential Dumping | `lsass.exe` |
| [T1055](https://attack.mitre.org/techniques/T1055/) | Process Injection | `explorer.exe`, `svchost.exe` |
| [T1036](https://attack.mitre.org/techniques/T1036/) | Masquerading | All core processes |
| [T1547.004](https://attack.mitre.org/techniques/T1547/004/) | Winlogon Helper DLL – Persistence | `winlogon.exe` |
| [T1569.002](https://attack.mitre.org/techniques/T1569/002/) | System Services: Service Execution | `services.exe`, `svchost.exe` |
| [T1134](https://attack.mitre.org/techniques/T1134/) | Access Token Manipulation | `lsass.exe` |
| [T1574.002](https://attack.mitre.org/techniques/T1574/002/) | DLL Side-Loading | `svchost.exe`, `explorer.exe` |
| [T1055.012](https://attack.mitre.org/techniques/T1055/012/) | Process Hollowing | Any core process |

---
