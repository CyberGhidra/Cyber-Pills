# LimaCharlie — EDR Platform Overview

**Official links:**
- [https://limacharlie.io/](https://limacharlie.io/) — product overview and documentation
- [https://app.limacharlie.io/signup](https://app.limacharlie.io/signup) — create a free account

---

## Part 1 — Introduction to LimaCharlie

### 1.1 The EDR Landscape

**EDR (Endpoint Detection and Response)** tools sit on individual machines (endpoints) and collect real-time telemetry — process execution, network connections, file activity, registry changes — feeding it into a central platform for detection, alerting, and response.

The course starts by putting LimaCharlie in context:

| Tool | Type | Cost | Notes |
|------|------|------|-------|
| **CrowdStrike Falcon** | Enterprise EDR | Paid (expensive) | Industry-leading, used by large organizations — the reference standard, but completely out of reach for personal labs or study |
| **Wazuh** | Open-source SIEM/EDR | Free | Powerful but requires significant self-hosting and configuration effort |
| **LimaCharlie** | Cloud EDR | Free tier available | The course choice — modern, cloud-native, no infrastructure to manage, generous free tier ideal for labs |

The point of this comparison is practical: you don't need a $50k enterprise contract to learn EDR concepts. LimaCharlie gives you a real, production-grade platform to learn on for free.

### 1.2 Creating a Free Account

Sign up at `app.limacharlie.io/signup`. The free plan allows you to create **two organizations** (separate environments/tenants), each with up to two sensors (endpoints) connected at no cost.

During organization creation, LimaCharlie offers **templates** — pre-configured setups tailored to different use cases:

- Incident Response
- Threat Hunting
- Managed Security Service (MSSP)
- And others

The course selects **Incident Response**, which pre-loads a set of detection rules, sensors, and automation relevant to active threat investigation — aligning directly with the lab scenarios (the same `notmalware.elf` reverse shell from the previous labs).

---

## Part 2 — Endpoint Detection and Response: Platform Overview

### 2.1 Dashboard Layout

Once inside an organization, the left sidebar is the main navigation menu. The primary sections are:

```
├── Sensors
│   ├── Sensor List          ← all connected endpoints
│   ├── Event Collection     ← configure what telemetry to collect
│   ├── Payload              ← deploy files/scripts to endpoints
│   ├── Sensor Cull          ← remove stale/offline sensors
│   ├── Deployer Version     ← manage agent versions
│   ├── Installation Keys    ← generate keys for new agent deployments
│   └── Artifacts            ← collected files and forensic artifacts
│
├── Automation
│   ├── D&R Rules            ← Detection & Response rules
│   └── Add-ons              ← platform extensions and integrations
│
└── (other sections: Output, Integrations, etc.)
```

### 2.2 Sensor List — Viewing Connected Endpoints

```
Sensors → Sensor List
```

This is the central inventory of all endpoints running the LimaCharlie agent. The Incident Response template comes pre-populated with demo sensors so you can explore the platform immediately without needing to deploy agents first.

With the default filter `is online: true`, you see the active pre-configured sensors:

| Hostname | Tag | Status |
|----------|-----|--------|
| `ext-atomic-red-team` | `ext:ext-atomic-red-team` | Online |
| `ext-yara` | — | Online |
| `ext-reliable-tasking` | — | Online |

Removing the filter reveals all sensors including offline demo hosts:
`demo1`, `IT-01.corp.net`, and others.

> **What are these?** The `ext-*` sensors are pre-built integrations with well-known security tools (Atomic Red Team for adversary simulation, YARA for signature scanning, etc.). The `demo1` / `IT-01.corp.net` hosts simulate real corporate endpoints.

### 2.3 Exploring an Endpoint — `IT-01.corp.net`

Clicking on `IT-01.corp.net` opens the **endpoint detail view**, showing:

- **Status** — online/offline, OS version, agent version
- **IP address and MAC address**
- **Last seen timestamp**

And the following tabs:

| Tab | Purpose |
|-----|---------|
| **Analytics** | Overview of events and anomaly scores for this endpoint |
| **Detection** | All alerts triggered on this endpoint |
| **Timeline** | Chronological stream of every event captured (process start, network connection, file write, etc.) |

#### Examining a Detection Alert

Inside the **Detection** tab, alerts are listed with severity and a short identifier, e.g.:

```
00574-win-browser_with_suspicious_child
```

Clicking it opens a **side panel** with full context:

- **Detection name and category** (e.g., *Suspicious Child Process*)
- **Author** of the rule that triggered it
- **Event details** — what process, from where, with what arguments
- **Command line** — the exact command that was run
- **Hash** — SHA-256 of the involved binary, ready to pivot to VirusTotal

This is the core EDR workflow: an alert fires → you click it → you get immediate context to decide whether it's a true positive and what to do next, without manually digging through logs.

#### Working with Filters

The Detection tab supports flexible filtering (by time range, category, rule name, severity, etc.), letting you narrow down alerts on a busy endpoint to just the signals that matter — critical in real environments where dozens or hundreds of alerts can fire per day.

### 2.4 Automation — Detection & Response (D&R) Rules

```
Automation → D&R Rules
```

D&R Rules are the detection engine of LimaCharlie. Each rule has two parts:
- **Detect** — a condition that matches specific telemetry events (e.g., a registry key being written to `Run` for persistence)
- **Respond** — what to automatically do when the condition is met (e.g., generate an alert, isolate the endpoint, send a notification)

The Incident Response template pre-loads a library of community rules, for example:

| Rule name | What it detects |
|-----------|----------------|
| `AUDIT_CVE` | Known CVE exploitation attempts |
| `App Uninstalled` | Security software removal (common attacker tactic) |
| `win_reg_add_run_key` | Persistence via registry Run key (T1547.001) |

Opening `win_reg_add_run_key` shows:
- **Status** (enabled/disabled)
- **Tags and author**
- **Detect block** — the event type and field conditions
- **Output/Respond block** — what action is triggered (in this case, generating a detection alert with metadata)

> **MITRE ATT&CK reference:** T1547.001 (Boot or Logon Autostart Execution: Registry Run Keys) — one of the most common Windows persistence techniques, which is why it has a dedicated pre-built rule.

### 2.5 Add-ons

```
Automation → Add-ons
```

Add-ons extend LimaCharlie's capabilities with additional modules and integrations — for example, threat intelligence feeds, YARA scanning engines, or integrations with external SIEM/SOAR platforms. This is how the platform scales from a basic EDR to a full security operations stack without requiring separate tooling.

---

## Part 3 — Deploying Endpoint Agents

This section covers the full deployment flow: generating an installation key, downloading and installing the agent on a Windows machine, and exploring the live telemetry it produces — including catching a real credential dumping tool in action.

### 3.1 Generating an Installation Key

```
Sensors → Installation Keys
```

Installation keys are pre-authorization tokens — an agent can only connect to your organization if it presents a valid key during installation. This prevents unauthorized sensors from registering.

Steps:
1. Click **Add new key**
2. Set a **description** (e.g., `Windows Lab`)
3. Add **tags** to classify sensors created with this key (e.g., `workstation`, `windows`)
4. Save — the key string is generated and displayed

Copy the key string — you'll need it during agent installation.

### 3.2 Downloading the Agent

Directly below the key management section, **Sensor Download** provides pre-built agent binaries for each supported platform:

| Platform | File |
|----------|------|
| Windows 64-bit | `hcp_win_x64_release_4.29.3.exe` |
| Windows 32-bit | `hcp_win_x86_release_4.29.3.exe` |
| Linux 64-bit | `.deb` / binary |
| macOS | `.pkg` |

Download the **Windows 64-bit** installer.

### 3.3 Installing the Agent on a Windows Endpoint

On the target Windows machine, open **PowerShell** and navigate to the downloads folder:

```powershell
cd C:\Users\tcm\Downloads\
dir
```

You'll see the downloaded installer:
```
hcp_win_x64_release_4.29.3.exe
```

Install the agent with the `-i` flag followed by the installation key:

```powershell
.\hcp_win_x64_release_4.29.3.exe -i <YOUR_INSTALLATION_KEY>
```

The agent installs as a Windows service and immediately connects back to LimaCharlie's cloud. Within seconds, returning to **Sensor List** in the dashboard, the new endpoint appears — tagged with `workstation` and `windows` as configured.

### 3.4 Exploring Live Endpoint Telemetry

Clicking into the newly connected sensor reveals a set of live investigation tabs far richer than the demo sensors:

#### Autoruns
Shows everything configured to **run automatically on startup** — registry Run keys, scheduled tasks, startup folder entries, services, browser extensions. This is the first place to look for persistence mechanisms on a live host.

#### Console
An **interactive remote command interface** — you can type commands and receive output directly from the endpoint in real time, as if you were sitting at the machine. For example:

```
> netstat
```
Returns a live list of all network connections from that machine — equivalent to running `netstat` locally, but done remotely from the LimaCharlie dashboard.

> **MITRE ATT&CK reference:** This remote console capability maps to T1021 (Remote Services) from the attacker's perspective, and to live response capabilities from the defender's — LimaCharlie uses it as a legitimate investigation tool.

#### File System
A **remote file browser** presenting the endpoint's directory tree (starting from `C:\`). You can browse directories, inspect files, and — critically — select any executable and **submit it for SHA-256 hashing**, then cross-reference the hash on VirusTotal. This is how you investigate suspicious binaries (like a `notmalware.exe` dropped in `Downloads\`) without having to physically access the machine.

#### Network
Shows live and recent **network connections**, including:
```
services.exe    0.0.0.0    49707    TCP
```
Displays process name, local address, port, and protocol — the same information you'd get from `netstat -tnp` on Linux or `netstat -ano` on Windows, but surfaced in a clean dashboard view with process attribution built in.

#### Processes
Full live **process list** with PID, PPID, command line, and owning user — equivalent to `ps -AFH` on Linux or Task Manager's Details tab on Windows, but accessible remotely and with EDR-level metadata.

### 3.5 Catching Mimikatz — Live Detection in Action

To demonstrate LimaCharlie's detection capability, the lab runs **Mimikatz** — one of the most well-known credential dumping tools — and observes how the platform responds.

On the Windows endpoint, open a command prompt:

```cmd
cd C:\Users\tcm\Downloads\
dir
```

`mimikatz.exe` is present in the folder. Launch it:

```cmd
mimikatz.exe
```

Inside the Mimikatz prompt, run the classic credential dump command:

```
sekurlsa::logonpasswords
```

This attempts to extract plaintext passwords and NTLM hashes from LSASS (the Windows process that manages authentication), dumping credentials for all users currently logged into the machine.

> **MITRE ATT&CK reference:** T1003.001 (OS Credential Dumping: LSASS Memory) — one of the most commonly used post-exploitation techniques, executed by attackers immediately after gaining initial access to harvest credentials for lateral movement.

Returning to **LimaCharlie → Detection**, a new alert has appeared automatically:

```
HACKTOOL
```

Clicking it opens the full detection detail — including the **SHA-256 hash** of `mimikatz.exe`, which can be immediately submitted to VirusTotal to confirm its identity and reputation. The platform detected the tool based on its behavior and/or signature without any manual configuration required — this is the D&R rules engine working as intended.

---

## 4. End-to-End Summary

```
PHASE 1 — Setup
├── Create free LimaCharlie account
├── Choose "Incident Response" template
└── Organization pre-loaded with demo sensors + D&R rules

PHASE 2 — Platform Exploration
├── Sensor List → view all endpoints (demo + live)
├── Endpoint detail → Status, IP, MAC, Analytics, Detection, Timeline
├── Detection alerts → full context (rule, event, command line, hash)
└── D&R Rules → pre-built detections (registry persistence, CVEs, etc.)

PHASE 3 — Agent Deployment & Live Investigation
├── Generate installation key (with tags)
├── Download Windows agent → install with -i <key>
├── Sensor appears in dashboard immediately
├── Live tabs: Autoruns, Console, File System, Network, Processes
└── Run Mimikatz → HACKTOOL alert fires automatically → hash → VirusTotal
```

**Key takeaway:** LimaCharlie provides a full production-grade EDR capability at zero cost for lab use. The combination of live telemetry (processes, network, files, autoruns), remote console, and an automated D&R rules engine covers the entire detection and investigation workflow — from initial alert to artifact recovery — without requiring any physical access to the endpoint.

---
