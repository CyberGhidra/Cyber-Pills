# Order of Volatility

---
## What Is Evidence?

**Evidence** is any data that supports or refutes an investigative hypothesis — it is the raw material of every forensic investigation. In digital forensics, evidence takes the form of **artifacts**: files, logs, memory contents, network traffic captures, registry entries, timestamps, and any other digital trace left by user or system activity.

Three properties of digital evidence define how it must be handled:

- **Evidence supports or refutes a hypothesis** — it is not inherently proof of guilt or innocence. Its meaning depends on context, correlation with other evidence, and the investigative framework applied to it (as covered in the Analysis phase of the Digital Forensics Investigation Process).
- **Evidence is circumstantial** — digital artifacts rarely tell a complete story on their own. A file in an unusual location, a login at an odd hour, a process connecting to an unfamiliar IP — each is a piece of a puzzle, not a conclusion. Multiple corroborating artifacts build a picture; a single artifact in isolation proves very little.
- **Evidence is volatile** — this is the property that drives everything in this topic. Digital evidence can disappear in milliseconds (when a process ends), in seconds (when a connection closes), on reboot (when RAM is cleared), or when power is lost. The investigator has a limited and rapidly closing window to capture certain types of evidence before they are gone permanently.

> **Core principle:** the volatility of evidence determines the urgency with which it must be collected. You cannot go back and capture RAM from a system that was rebooted two hours ago.

---

## Evidence Sources

Digital evidence can originate from a wide range of sources — each with different characteristics, access requirements, and volatility profiles:

| Source | Examples | Key Considerations |
|--------|---------|-------------------|
| **Desktops and laptops** | Windows/macOS/Linux workstations | Primary endpoint evidence source; RAM, disk, logs all present |
| **Smartphones and tablets** | iOS, Android devices | Highly volatile if not isolated (remote wipe risk); specialized acquisition tools required |
| **Hard drives and external storage** | HDDs, SSDs, USB drives, NAS | Non-volatile storage; can be imaged forensically; SSDs may complicate deleted file recovery due to TRIM |
| **Servers and virtual machines** | Physical servers, VMware/Hyper-V VMs | VM snapshots are a powerful preservation tool; logs often richer than on endpoints |
| **Network devices** | Routers, switches, firewalls, IDS/IPS | Contain routing tables, ARP caches, connection logs — often overlooked but highly valuable |
| **Cloud services** | AWS, Azure, GCP, SaaS platforms | Evidence may be geographically distributed; access requires legal process or provider cooperation; retention policies vary |

---

## The Order of Volatility

The **order of volatility** is the sequence in which evidence must be collected, from the most volatile (disappears fastest) to the least volatile (persists longest). Collecting in any other order risks permanently losing the most time-sensitive evidence while time is spent on data that would have waited.

```
┌─────────────────────────────────────────────────────────────┐
│  MOST VOLATILE                                              │
│                                                             │
│  [1]  CPU Registers and Cache                               │
│  [2]  RAM and Active Network Connections                    │
│  [3]  Temporary Files and System Logs                       │
│  [4]  Disk Storage Data                                     │
│  [5]  Remote Logging                                        │
│  [6]  Archive Media                                         │
│                                                             │
│  LEAST VOLATILE                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Block 1 — CPU Registers and Cache

**Survives:** only while the processor is actively using them — measured in nanoseconds to microseconds.

**What it contains:**

- **Processor registers** — small, extremely fast storage built directly into the CPU itself. Registers hold the data and instructions the processor is actively working with at any given clock cycle: memory addresses, arithmetic operands, instruction pointers, flags. This is the most transient data on any system — it changes billions of times per second and is completely unrecoverable the instant the processor moves on.
- **Cache memory** — a small, fast memory layer between the CPU and RAM (typically L1, L2, and L3 cache). The cache stores frequently used instructions and data so the CPU can retrieve them faster than fetching from RAM. Like registers, cache content is continuously overwritten and cannot be meaningfully captured in a forensic context.

**Practical forensic relevance:** CPU registers and cache are largely theoretical at the bottom of the pyramid — in practice, no forensic tool can usefully capture this data at the speed it changes. Its inclusion in the order of volatility is conceptual, establishing that *something* always sits above RAM in terms of transience.

---

### Block 2 — RAM and Active Network Connections

**Survives:** until the system is powered off or rebooted — can persist for hours or days on a running system, but is gone permanently the instant power is cut.

This is the most forensically actionable volatile tier — RAM can be captured with the right tools, and its contents are extraordinarily rich.

**RAM (Random Access Memory):**
- Running processes and their full memory space (code, stack, heap)
- Decrypted versions of data that is encrypted at rest on disk (full disk encryption is irrelevant to a live memory capture — the plaintext is in RAM)
- Credentials and authentication tokens cached in memory (the basis of Mimikatz/LSASS credential dumping, as demonstrated in the LimaCharlie lab)
- Fileless malware that exists only in memory and was never written to disk — completely invisible to disk-based forensics
- Clipboard contents, recently typed text, and unsaved documents
- Encryption keys for encrypted volumes (BitLocker, VeraCrypt, FileVault)

**Active Network Connections:**
- Current TCP/UDP connections (source/destination IP and port, connection state) — the same data visible via `netstat -tnp` and `ss -tnp` in the Linux Network Analysis lab
- **Routing table** — the rules the system uses to decide where to send network traffic; may reveal attacker-added routes for traffic redirection
- **ARP cache** — the mapping between IP addresses and MAC addresses for recently contacted hosts; can reveal which systems on the local network the compromised host has been communicating with, providing lateral movement leads

**Capture tools:** `WinPmem`, `DumpIt` (Windows); `LiME` kernel module (Linux); built-in memory acquisition in EDR platforms like LimaCharlie.

> **Critical decision point:** before rebooting or shutting down any system that may hold evidence, ask — has RAM been captured? If not, the entire contents of Block 2 will be permanently lost.

---

### Block 3 — Temporary Files and System Logs

**Survives:** typically across reboots (written to disk) but subject to automatic deletion, rotation, and overwriting — hours to days depending on system configuration.

**Temporary files:**
- Files created by the OS, applications, or users for short-term use: installer remnants, browser cache, print spooler files, application crash dumps
- **Swap files / page files** (`pagefile.sys` on Windows, swap partition on Linux) — when RAM fills up, the OS writes portions of it to disk as a swap file. This creates a on-disk record of memory contents that may survive a reboot, potentially containing artifacts from Block 2 (credentials, process data, network buffers) that would otherwise have been lost

**System and application logs:**
- Operating system event logs (`/var/log/` on Linux; Windows Event Log)
- Application logs (web server access logs, database query logs, authentication logs)
- Security logs (firewall logs, IDS/IPS alerts, authentication events)

Logs are the primary source for timeline reconstruction during Analysis — they record *when* things happened with timestamps that can be cross-referenced across multiple systems. However, logs are only as useful as their retention policy allows: a system configured to rotate logs every 24 hours may have already overwritten the evidence of an intrusion that happened last week.

> **Detection tip:** one of the first things an attacker does after gaining access is check log retention settings and, if possible, clear or truncate logs (MITRE ATT&CK T1070.002 — Indicator Removal: Clear Linux or Mac System Logs). Centralized log collection (Block 5) is the defense against this — logs shipped off the compromised system cannot be deleted by an attacker who only controls the endpoint.

---

### Block 4 — Disk Storage Data

**Survives:** indefinitely until overwritten — the most durable local evidence source.

**Hard Disk Drives (HDDs):**
- Traditional magnetic storage; deleted files are marked as available space but the data remains physically present on the disk until the sectors are overwritten by new data — making deleted file recovery frequently possible
- File system metadata: creation, modification, and access timestamps (`ctime`, `mtime`, `atime`) for every file — essential for timeline reconstruction

**Solid State Drives (SSDs):**
- Flash-based storage; the **TRIM** command (issued by the OS when a file is deleted) actively erases the underlying data blocks to maintain write performance. This significantly reduces or eliminates the possibility of deleted file recovery on SSDs — a critical distinction from HDDs for forensic investigators
- Speed and prevalence of SSDs in modern systems means deleted file recovery is increasingly difficult

**System configuration:**
- Installed software inventory
- User account database (`/etc/passwd`, `/etc/shadow`, SAM/NTDS.dit on Windows)
- Scheduled tasks, cron jobs, startup entries — persistence mechanisms that survive on disk
- Registry (Windows) — a rich source of forensic artifacts including recently accessed files (MRU lists), USB device history, user activity, and installed software

**Forensic approach:** bit-for-bit disk imaging using tools like `dd`, FTK Imager, or `dcfldd`. The image is verified by hashing (SHA-256 of the image must match the hash of the source) before any analysis begins.

---

### Block 5 — Remote Logging

**Survives:** according to the retention policy of the remote system — potentially weeks, months, or years; and crucially, **survives even if the local system is wiped**.

- **Endpoint telemetry** — data sent by EDR agents (LimaCharlie, CrowdStrike, Wazuh) to a central cloud or on-premises platform in near real-time. Even if the endpoint is destroyed, compromised, or wiped, the telemetry already shipped to the EDR platform remains available for investigation.
- **Centralized log collection** — logs forwarded from endpoints and network devices to a SIEM (Security Information and Event Management) or log aggregator (Splunk, Microsoft Sentinel, Elastic). Centralized logs are the investigator's fallback when local logs on the compromised system have been cleared, rotated, or are otherwise unavailable.

Remote logging is the primary reason why organizations invest in SIEM infrastructure — not just for real-time alerting, but for the forensic durability it provides. An attacker who deletes all local logs has not deleted the copies already shipped to the SIEM.

---

### Block 6 — Archive Media

**Survives:** years to decades — the most durable evidence source, by design.

- **Long-term storage media** — magnetic tape, optical media (Blu-ray, DVD), cold storage drives used for archival purposes. These are not accessed frequently and are therefore unlikely to have been altered by the attacker or overwritten during normal operations.
- **Backups** — scheduled system and data backups stored offline or offsite. Backups can be forensically significant because they represent the state of a system at a specific point in time — a backup from before the incident can establish a clean baseline; a backup from during the incident may contain a copy of malicious files that have since been deleted.
- **Security controls** — access logs, surveillance footage, badge reader records, and other physical/logical access records that are archived for compliance or audit purposes. These can corroborate digital findings (e.g., a user's badge record showing they were not in the office at the time a login was attributed to them).

---

## Summary — Order of Volatility at a Glance

| Block | Evidence Type | Survives Until | Forensic Priority |
|-------|--------------|---------------|------------------|
| **1** | CPU registers and cache | Microseconds | Conceptual — not practically capturable |
| **2** | RAM, active connections, routing/ARP tables | Power-off or reboot | **CRITICAL — capture immediately** |
| **3** | Temp files, swap, system/app logs | Hours to days (rotation/deletion) | **HIGH — capture before reboot** |
| **4** | Disk storage, system config, file system | Until overwritten | Medium — durable but follow order |
| **5** | Remote logging, EDR telemetry, SIEM | Per retention policy | Low urgency — already off-system |
| **6** | Backups, archive media, physical records | Years | Low urgency — most durable |

> **The golden rule:** always work from the top of the pyramid down. Evidence at Block 2 that is lost because the investigator spent time on Block 4 first is gone forever. Evidence at Block 4 will still be there after Block 2 has been captured.

