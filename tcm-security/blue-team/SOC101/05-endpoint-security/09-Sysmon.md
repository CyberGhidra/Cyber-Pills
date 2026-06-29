# Sysmon — System Monitor

---

## Essential Resources

| Resource | URL |
|---|---|
| Sysmon Official Download (Microsoft Sysinternals) | https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon |
| SwiftOnSecurity sysmon-config | https://github.com/SwiftOnSecurity/sysmon-config |
| olafhartong sysmon-modular | https://github.com/olafhartong/sysmon-modular |

---

## Part 1 — Introduction to Sysmon

### 1.1 What is Sysmon?

**System Monitor (Sysmon)** is a free Windows system service and device driver from **Microsoft Sysinternals**. Once installed, it runs persistently across reboots and logs detailed information about system activity to the **Windows Event Log**.

Unlike the built-in Windows Event Log (which covers logins, service installs, etc.), Sysmon goes much deeper — it tracks:

- Exactly **which process spawned which child process** (full process tree)
- **Network connections** made by each process (source IP, destination IP, port)
- **File creation** events with hashes
- **DNS queries** made by processes
- **Registry modifications**
- **Driver and DLL loads**
- And much more

Sysmon is a cornerstone tool for **Blue Team operations**, threat hunting, and incident response. It dramatically increases visibility into what is actually happening on a Windows endpoint.

> **Key distinction:** Windows out of the box gives you *that* something happened (e.g., Event ID 4688 = process created). Sysmon tells you *exactly what, how, and why* — parent process, command line, file hash, network destination — with far more detail and reliability.

---

### 1.2 Downloading Sysmon

Download Sysmon from the official Microsoft Sysinternals page:

🔗 https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

After extracting the ZIP, the folder contains three executables:

| File | Description |
|---|---|
| `Sysmon.exe` | 32-bit version — for legacy 32-bit Windows systems |
| `Sysmon64.exe` | 64-bit version — use this on all modern 64-bit Windows systems |
| `Sysmon64a.exe` | 64-bit version compiled for **ARM64** architecture (e.g., Windows on ARM devices) |

For most environments, **`Sysmon64.exe`** is the correct binary to use.

---

### 1.3 Sysmon Without a Configuration File

If you install Sysmon with no configuration, it uses a very **minimal default ruleset** — it logs only a small subset of events and misses most of what matters for security monitoring. The raw tool alone is not enough.

This is where **community configuration files** come in. They define exactly which events to capture, which to ignore (to reduce noise), and what filters to apply.

---

### 1.4 SwiftOnSecurity — `sysmon-config`

🔗 https://github.com/SwiftOnSecurity/sysmon-config

**"A Sysmon configuration file for everybody to fork"**

This is one of the most widely used Sysmon configuration files in the security community, maintained by SwiftOnSecurity. It provides a **well-balanced, production-ready XML configuration** that:

- Captures all high-value security events
- Filters out known noisy/benign processes to reduce log volume
- Is heavily commented and easy to understand and customize
- Serves as an excellent **starting point** for any organization

The key file is:

📄 [`sysmonconfig-export.xml`](https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml)

This XML file is what you pass to Sysmon at install time to tell it exactly what to monitor.

---

### 1.5 olafhartong — `sysmon-modular`

🔗 https://github.com/olafhartong/sysmon-modular

**"A Sysmon configuration repository for everybody to customise"**

Where SwiftOnSecurity's config is a single monolithic file, `sysmon-modular` takes a **modular approach** — each event type (process creation, network connections, registry changes, etc.) has its own separate XML file. You assemble the final configuration by combining only the modules you need.

**Why use sysmon-modular?**
- Easier to maintain at scale — update one module without touching the rest
- Maps directly to **MITRE ATT&CK** techniques — each module is labeled with the ATT&CK coverage it provides
- Better suited for **enterprise environments** with custom tuning requirements
- Ideal for teams building their own detection engineering pipeline

> **In practice:** Start with SwiftOnSecurity's config to get up and running quickly. Graduate to sysmon-modular when you need fine-grained control and ATT&CK alignment.

---

### 1.6 Installing Sysmon with a Configuration File

**Step 1 — Download and place files**

Download `sysmonconfig-export.xml` from the SwiftOnSecurity GitHub and place it in the same folder as the Sysmon executables:

```
C:\Users\tcm\Downloads\Sysmon\
├── Sysmon.exe
├── Sysmon64.exe
├── Sysmon64a.exe
└── sysmonconfig-export.xml
```

**Step 2 — Open PowerShell and navigate to the folder**

```powershell
cd C:\Users\tcm\Downloads\Sysmon\
dir
```

`dir` confirms all files are present before proceeding.

**Step 3 — View Sysmon help**

```powershell
.\Sysmon64.exe
```

Running the binary without arguments displays the full usage help — available flags, options, and syntax. Useful to review before installing.

**Step 4 — Install Sysmon with the configuration**

```powershell
.\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula
```

| Flag | Meaning |
|---|---|
| `-i` | **Install** Sysmon as a Windows service |
| `.\sysmonconfig-export.xml` | The configuration file to apply at install time |
| `-accepteula` | Silently accepts the End User License Agreement (required for non-interactive installs) |

On success, Sysmon installs itself as a **Windows service and driver** and begins logging immediately.

> **Other useful flags:**
> - `.\Sysmon64.exe -u` — **Uninstall** Sysmon
> - `.\Sysmon64.exe -c .\newconfig.xml` — **Update** the configuration without reinstalling

**Step 5 — Verify the service is running**

```cmd
net start
```

Look for **Sysmon64** in the list of running services. Its presence confirms Sysmon is active and logging.

---

### 1.7 Viewing Sysmon Events in Event Viewer

Open **Event Viewer** (`eventvwr.msc`) and navigate to:

```
Applications and Services Logs
  └── Microsoft
        └── Windows
              └── Sysmon
                    └── Operational
```

Here you will see Sysmon events as they are generated in real time. Common event IDs you will immediately start seeing include:

- **Event ID 1** — Process Creation (every new process)
- **Event ID 3** — Network Connection
- **Event ID 8** — CreateRemoteThread (process injection indicator)
- **Event ID 22** — DNS Query

The full Sysmon log path used in queries is:
```
Microsoft-Windows-Sysmon/Operational
```

---

## Part 2 — Sysmon Event IDs

### 2.1 Key Sysmon Event IDs

Sysmon defines its own set of Event IDs, completely separate from the standard Windows Security/System logs. Each ID maps to a specific type of system activity.

| Event ID | Name | Description | Why It Matters |
|---|---|---|---|
| **1** | **Process Creation** | Logged every time a new process starts. Includes: process name, full path, command line arguments, parent process, user, and **file hash** (MD5/SHA256). | Most important Sysmon event. Catch malware execution, reconnaissance tools (`whoami`, `net`, `ipconfig`), and living-off-the-land binaries (LOLBins). |
| **3** | **Network Connection** | Logs TCP/UDP connections made by a process. Includes: source/destination IP, source/destination port, process name. | Detect C2 (Command & Control) beaconing, reverse shells (e.g., connections to port 4444), lateral movement. |
| **5** | **Process Terminated** | Logged when a process exits. Includes the process GUID and PID. | Correlate with Event ID 1 to get full process lifetime. Useful for short-lived processes that run and exit quickly (common in malware). |
| **7** | **Image Loaded** | Logs when a DLL or executable image is loaded into a process. | Detect **DLL hijacking**, reflective DLL injection, and suspicious library loads (e.g., `mimikatz` loading `sekurlsa.dll`). |
| **8** | **CreateRemoteThread** | Logs when a process creates a thread in another process's memory space. | Classic indicator of **process injection** — a key technique used by malware to hide inside legitimate processes. |
| **10** | **ProcessAccess** | Logs when one process opens a handle to another process (e.g., for memory reading). | Detect **credential dumping** — tools like Mimikatz access `lsass.exe` to extract password hashes. |
| **11** | **FileCreate** | Logs file creation and overwrite events. Includes file path and hash. | Detect malware dropping payloads, scripts being written to disk, ransomware creating encrypted files. |
| **12** | **RegistryEvent (Object create/delete)** | Logs registry key and value creation or deletion. | Detect persistence via registry run keys (e.g., `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`). |
| **15** | **FileCreateStreamHash** | Logs when a file is created with an **Alternate Data Stream (ADS)**. | ADS is a classic NTFS-based hiding technique — malware can be hidden inside a legitimate file's alternate stream. |
| **22** | **DNSEvent** | Logs DNS queries made by processes. Includes the queried domain and the responding IP(s). | Detect **C2 domain lookups**, DGA (Domain Generation Algorithm) traffic, and suspicious domains being resolved. |
| **29** | **FileExecutableDetected** | Logs when an executable file (PE format) is **created on disk**. New in recent Sysmon versions. | Catch malware being written to disk before it even runs — earlier detection than Event ID 1. |

> **The "holy trinity" for detecting a reverse shell:**
> - **Event ID 1** — the malware process was created
> - **Event ID 3** — it made a network connection to the attacker's IP on port 4444
> - **Event ID 22** — it may have resolved a domain first (C2 domain lookup)

---

### 2.2 Querying Sysmon Events with PowerShell

All Sysmon events live in the `Microsoft-Windows-Sysmon/Operational` log. Use `Get-WinEvent` to query them.

---

**Get all Sysmon events:**

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational"
```

Returns every event in the Sysmon log — useful for a first look but will be very large on active systems.

---

**Filter by Event ID (e.g., ID 1 — Process Creation):**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-Sysmon/Operational"
    Id      = 1
}
```

Returns all process creation events. On a busy system, add `-MaxEvents` to limit results.

---

**Get the most recent Network Connection event with full details:**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-Sysmon/Operational"
    Id      = 3
} -MaxEvents 1 | Format-List *
```

`Format-List *` expands **all properties** of the event object — including the raw `Message` field which contains the full Sysmon event data (process name, source/destination IPs and ports, etc.).

---

**XPath filter — find network connections to a specific port (e.g., port 4444 — reverse shell):**

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' `
    -FilterXPath "*[System/EventID=3 and EventData[Data[@Name='DestinationPort']='4444']]" |
    Format-List *
```

This is a powerful **XPath query** that filters at the log API level:
- `System/EventID=3` — only Network Connection events
- `EventData[Data[@Name='DestinationPort']='4444']` — where the destination port equals 4444

This would directly surface a reverse shell connection from the Metasploit demo — a real-world detection query.

---

**Get all Process Creation events via XPath:**

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' `
    -FilterXPath "*[System/EventID=1]"
```

---

**Find all events related to a specific Process ID (PID):**

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' `
    -FilterXPath "*[System/EventID=1 and EventData[Data[@Name='ProcessId']='<ENTER YOUR PID HERE>']]" |
    Format-List *
```

Replace `<ENTER YOUR PID HERE>` with the actual PID (e.g., `1337`).

**Why this matters:** When investigating a suspicious process, you find its PID in Task Manager or via `Get-Process`. You can then pull **all Sysmon Event ID 1 entries for that PID** to reconstruct exactly what command was run, with what arguments, spawned by which parent process.

---

### 2.3 XPath Queries in Event Viewer

Event Viewer supports the same XPath syntax via its **Filter Current Log → XML** tab. This lets you build precise queries visually and save them as Custom Views.

---

**Query 1 — All Sysmon events for a specific Process ID:**

```xml
<QueryList>
  <Query Id="0" Path="Microsoft-Windows-Sysmon/Operational">
    <Select Path="Microsoft-Windows-Sysmon/Operational">
      *[System[Provider[@Name='Microsoft-Windows-Sysmon']]]
      and
      *[EventData[Data[@Name='ProcessId'] and (Data='<ENTER YOUR PID HERE>')]]
    </Select>
  </Query>
</QueryList>
```

**What this does:**
- `Provider[@Name='Microsoft-Windows-Sysmon']` — scopes the query to Sysmon events only (not other providers in the same log)
- `EventData[Data[@Name='ProcessId'] and (Data='...')]` — matches any Sysmon event where the `ProcessId` field equals your target PID

This returns **all event types** (ID 1, 3, 5, 7, 8, 10, 11, etc.) associated with that process — giving you a complete picture of everything that process did: what it spawned, what network connections it made, what files it created, what registry keys it touched.

---

**Query 2 — Only Process Creation events (Event ID 1) for a specific PID:**

```xml
<QueryList>
  <Query Id="0" Path="Microsoft-Windows-Sysmon/Operational">
    <Select Path="Microsoft-Windows-Sysmon/Operational">
      *[System[Provider[@Name='Microsoft-Windows-Sysmon'] and (EventID=1)]]
      and
      *[EventData[Data[@Name='ProcessId'] and (Data='<ENTER YOUR PID HERE>')]]
    </Select>
  </Query>
</QueryList>
```

**What this adds over Query 1:**
- `(EventID=1)` — restricts results to **Process Creation only**

This is useful when you already know the PID of a suspicious process and want to confirm **exactly how it was launched** — the full command line, the parent process, and the file hash — without being flooded by its network or file activity events.

---

**When to use which query:**

| Scenario | Use |
|---|---|
| "I have a suspicious PID — show me everything it did" | Query 1 (all event types, filtered by PID) |
| "I want to confirm how this process was launched" | Query 2 (Event ID 1 only, filtered by PID) |
| "Show me all reverse shell connections" | PowerShell XPath with EventID=3 and DestinationPort=4444 |
| "Show me all new processes in the last hour" | FilterHashtable with Id=1 and StartTime |

---

## 3. Summary — Sysmon at a Glance

```
Sysmon = Windows native tool (Sysinternals)
       + Community config (SwiftOnSecurity / sysmon-modular)
       = Deep endpoint visibility with minimal overhead
```

| What you get | How |
|---|---|
| Every process that ran, with hash and parent | Event ID 1 |
| Every network connection, with destination | Event ID 3 |
| Every DNS query made | Event ID 22 |
| Process injection attempts | Event ID 8 |
| Credential dumping attempts | Event ID 10 |
| Files dropped to disk | Event ID 11, 29 |
| Registry persistence | Event ID 12 |
| Precise per-PID investigation | XPath queries in PowerShell or Event Viewer |

---
