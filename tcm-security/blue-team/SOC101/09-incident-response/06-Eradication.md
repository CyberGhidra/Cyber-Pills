# Incident Response — Eradication

**Reference documents:**
- **CIS Benchmarks** — [https://www.cisecurity.org/cis-benchmarks](https://www.cisecurity.org/cis-benchmarks)
- **SANS Incident Handling — Eradication Form** — [https://www.sans.org/media/score/incident-forms/IH-Eradication.pdf](https://www.sans.org/media/score/incident-forms/IH-Eradication.pdf)

---

## The Goal of Eradication

By the time you reach Eradication, the attacker's active access has been cut (Containment), evidence has been preserved, and the environment is stable enough to begin cleanup. Eradication answers one question: **is every artifact the attacker left behind — on disk, in memory, on the network, in configuration — now gone?**

The risk of rushing Eradication is missing something. A single forgotten backdoor, a cron job the team didn't notice, a compromised service account that wasn't reset — any of these gives the attacker a way back in, and the incident restarts from the beginning. Thoroughness here is not optional.

> **Key principle:** Eradication is not complete until you can affirmatively answer "yes" to: *Is every attacker artifact gone? Is every access path closed? Is every vulnerability that was exploited now fixed?*

---

## 1. Removing the Attacker's System and Network Artifacts

The first step is a systematic hunt for everything the attacker touched, created, or modified — across both the affected systems and the network infrastructure.

**System artifacts to identify and remove:**

| Artifact Type | Examples | Where to Look |
|--------------|----------|---------------|
| Malware binaries | Dropped executables, scripts, ELF/PE payloads | Common drop locations: `/tmp`, `/var/tmp`, `C:\Users\<user>\AppData`, `C:\Windows\Temp` |
| Persistence mechanisms | Cron jobs, registry Run keys, scheduled tasks, startup folder entries, modified `~/.bashrc` or `~/.profile` | `/etc/cron*`, `/var/spool/cron/`, `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`, `systemctl list-timers` |
| Web shells | PHP/ASPX/JSP files planted in web directories to maintain persistent HTTP-based access | Web root directories, recently modified files in web servers |
| Backdoor accounts | User accounts created by the attacker | `/etc/passwd`, `net user` (Windows), Active Directory |
| Modified system files | Replaced legitimate binaries (rootkit technique), modified config files | Compare hashes against known-good baselines |
| Attacker tools | Reconnaissance tools, credential dumpers (Mimikatz), lateral movement frameworks | Downloads folders, temp directories, any non-standard location |

**Network artifacts to address:**

- **C2 infrastructure** — block all known attacker IPs, domains, and ports at the firewall and DNS level (building on the temporary blocks applied during Containment)
- **Malicious firewall rules** — attackers sometimes add their own firewall rules to maintain access or facilitate exfiltration; audit and remove any unauthorized rules
- **Rogue network services** — check for unauthorized listeners (unusual open ports identified via `netstat`/`ss` during the Network Analysis phase) that the attacker may have set up

> **MITRE ATT&CK reference:** T1070 (Indicator Removal) — attackers often attempt to delete their own artifacts before or during detection. The fact that an artifact is missing doesn't mean it was never there — forensic images captured during Containment preserve the evidence even after cleanup.

---

## 2. Restoring Systems

Once attacker artifacts are confirmed removed, systems must be returned to a known-good state. The method chosen depends on the severity of the compromise and the organization's recovery capabilities.

### Option A — Restore from Known-Good Backup
If clean, verified backups exist from before the compromise:
- Restore the system from the most recent pre-incident backup
- Verify the backup's integrity (hash check) before restoring — a compromised backup restores the attacker along with the data
- Apply all patches and updates released since the backup was taken before bringing the system back online

This is the fastest path to a clean system, but only works if:
1. Backups exist and are current
2. The backup predates the initial compromise (which may be weeks or months before detection)
3. The backup itself was not compromised (attackers sometimes target backup systems specifically)

### Option B — Rebuild from Original Disk Image
Use the clean OS images maintained as part of the asset management process (covered in Preparation) to rebuild the system from scratch:
- Wipe and reimage the machine
- Reinstall applications from verified sources
- Restore only data (not executables or configurations) from backups, after verifying those files are clean

This is more time-consuming but produces a definitively clean system — particularly important for systems where the depth of compromise is uncertain.

### RTO and RPO — Defining Acceptable Recovery Parameters

Two metrics from business continuity planning directly govern recovery decisions:

| Metric | Definition | Example |
|--------|-----------|---------|
| **RTO** (Recovery Time Objective) | The maximum acceptable time a system can be offline before the business impact becomes unacceptable | "This system must be restored within 4 hours" |
| **RPO** (Recovery Point Objective) | The maximum acceptable amount of data loss, expressed as a point in time | "We can tolerate losing up to 24 hours of data" |

These values should be defined per system during Preparation (as part of the criticality classification in asset management) so that during an incident, the IR team knows exactly how long they have to restore each system and how much data loss is acceptable. Without pre-defined RTO/RPO, every restoration decision becomes a negotiation under pressure.

> **Example:** A financial transaction system might have an RTO of 2 hours and an RPO of 0 (zero data loss tolerated). A developer workstation might have an RTO of 48 hours and an RPO of 24 hours. These different tolerances drive completely different restoration strategies.

---

## 3. Manually Removing Artifacts

When a full system restore or rebuild is not immediately possible (due to RTO constraints, complexity, or business impact), artifacts must be removed manually. This requires methodical work through every layer of the system.

### Malware
- Identify all malicious files using hashes, file names, paths, and behavioral indicators gathered during investigation
- Delete the files — then verify deletion (confirm the file no longer exists AND that the process is no longer running)
- Check for copies in temp directories, user profiles, and any other writable location the attacker may have used as a staging area

### Disk Artifacts
- Deleted malicious files may still be recoverable from unallocated disk space — on a system being returned to production, a secure wipe of unallocated space prevents recovery
- Review recently modified files (sorted by `mtime`) for anything that changed during the attacker's presence window that doesn't have a legitimate explanation

### Memory Artifacts
- Fileless malware lives only in memory and leaves no disk artifacts to delete — the only reliable eradication is a reboot, which clears RAM
- Before rebooting, confirm that a memory dump was captured during Containment, so the fileless payload is preserved for analysis even after it's gone from the live system

### Backdoors
Backdoors are persistence mechanisms specifically designed to survive reboots and basic cleanup — they are the attacker's insurance policy. A thorough backdoor hunt covers:
- **Cron jobs** — `sudo crontab -u <user> -l` for every user, plus all `/etc/cron.*` directories (as covered in the Linux Cron Jobs lab)
- **Scheduled tasks** (Windows) — `schtasks /query /fo LIST /v`
- **Registry autorun keys** (Windows) — `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` and equivalent keys
- **Systemd services / init scripts** — `systemctl list-units --type=service`, review `/etc/systemd/system/`
- **SSH authorized keys** — check `~/.ssh/authorized_keys` for every user; attackers commonly add their own public key for persistent SSH access
- **Web shells** — scan web directories for recently added or modified files with webshell signatures

### Network Connections
- Kill any remaining unauthorized network connections (identified via `netstat`/`ss`/`lsof -i`)
- Remove or block any remote access tools the attacker installed (RATs, tunneling tools, reverse proxies)
- Confirm C2 communication has fully stopped — no outbound connections to attacker infrastructure remaining

### Malicious Services
- Review all running services for anything unauthorized (`systemctl list-units`, `sc query` on Windows)
- Disable and remove any services the attacker created or modified to maintain persistence or facilitate access

---

## 4. Improving Defenses

Eradication is not just about removing what the attacker put in — it is also about closing the doors they used to get in and hardening the environment against the next attempt.

### Network Filters
Make the temporary containment network blocks permanent where appropriate:
- Convert emergency firewall rules into formal, reviewed rules in the firewall policy
- Ensure blocks are applied consistently across all perimeter and internal firewall devices, not just the one(s) touched during the incident

### Applying Patches
Every vulnerability exploited during the incident must be patched — on every affected system and on every other system running the same vulnerable software version:
- Identify the CVE(s) involved
- Source and test the vendor patch
- Deploy in a controlled, staged rollout
- Verify patch application on all in-scope systems

Patch deployment should be tracked and documented — both to confirm it was done and to demonstrate due diligence in any regulatory or legal context.

### Hardening Systems
Beyond patching the specific exploited vulnerability, the incident is an opportunity to harden the affected systems more broadly:
- Disable unnecessary services and open ports that are not required for the system's function
- Enforce least-privilege on user accounts and service accounts
- Remove unnecessary software (attack surface reduction)
- Enforce strong authentication (MFA where not already in place)
- Review and tighten file and directory permissions

### CIS Benchmarks
The **CIS Benchmarks** ([cisecurity.org/cis-benchmarks](https://www.cisecurity.org/cis-benchmarks)) are free, consensus-developed configuration hardening guides for virtually every major operating system, application, and cloud platform. They define specific, actionable configuration settings that measurably reduce attack surface — organized into two implementation levels:

| Level | Description |
|-------|-------------|
| **Level 1** | Essential security settings with minimal operational impact — suitable for all environments |
| **Level 2** | More restrictive settings for high-security environments — may affect some functionality |

Using CIS Benchmarks during Eradication ensures hardening is systematic and based on industry best practice, rather than ad hoc and dependent on individual knowledge. They also provide a documented, auditable standard — useful for demonstrating compliance to regulators or auditors after an incident.

---

## 5. Vulnerability Management

The incident revealed at least one exploitable weakness. Eradication is the time to find and address others before the next attacker does.

### Scanning Systems and the Network
- Run authenticated vulnerability scans against all systems in scope (and ideally the broader environment) immediately after the incident — to identify other vulnerabilities that may have been missed before or introduced during the attacker's presence
- Tools: Nessus, OpenVAS, Qualys, Rapid7 InsightVM
- Prioritize findings by CVSS score and exploitability, with particular attention to vulnerabilities matching the TTPs of the threat actor involved

### Related Vulnerabilities
The exploited vulnerability may be part of a family — the same class of weakness existing in other systems or applications. After remediating the specific CVE exploited:
- Search for the same vulnerability across other systems
- Check whether related vulnerabilities (same component, different CVE) exist
- Review vendor advisories for the affected product to understand the full scope of the patch

### Threat Intelligence Integration
Cross-reference findings from the incident with current threat intelligence:
- Are there other CVEs associated with the same threat actor or malware family?
- Are there known follow-on techniques commonly used after this initial access vector?
- Has the attacker infrastructure (IPs, domains, hashes) been seen targeting other organizations?

Threat intelligence helps anticipate the next move — even after eradication, the same actor may attempt re-entry via a different vector.

### Threat Hunting
Vulnerability scanning finds known weaknesses; threat hunting looks for attacker presence or artifacts that automated tools may have missed:
- Proactively search for IOCs from the incident across all systems, not just the confirmed affected ones
- Hunt for TTPs (not just IOCs) — behavioral patterns that match attacker activity even when specific signatures don't match
- Review systems that had network contact with the compromised host(s) for signs of lateral movement

> **MITRE ATT&CK reference:** The ATT&CK framework is the primary reference for structuring threat hunting — use the techniques identified in this incident as a hunting hypothesis and search for evidence of those techniques across the broader environment.

---

## Summary

```
CONTAINMENT COMPLETE
        │
        ▼
┌──────────────────────────────────┐
│  REMOVE ATTACKER ARTIFACTS       │
│  ─ Malware, tools, web shells    │
│  ─ Backdoors, cron jobs,         │
│    scheduled tasks               │
│  ─ Unauthorized accounts         │
│  ─ Malicious network connections │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│  RESTORE SYSTEMS                 │
│  ─ From backup (verify hash)     │
│  ─ Or full rebuild from image    │
│  ─ Within defined RTO/RPO        │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│  IMPROVE DEFENSES                │
│  ─ Permanent firewall rules      │
│  ─ Patch exploited CVE(s)        │
│  ─ Harden with CIS Benchmarks    │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│  VULNERABILITY MANAGEMENT        │
│  ─ Full environment scan         │
│  ─ Related vulnerability review  │
│  ─ Threat intelligence           │
│  ─ Threat hunting                │
└──────────────┬───────────────────┘
               │
               ▼
           RECOVERY
```

| Area | Key Activity |
|------|-------------|
| Artifact removal | Hunt and remove all malware, backdoors, unauthorized accounts, and malicious connections |
| System restoration | Restore from verified backup or rebuild from clean image within RTO/RPO |
| Manual cleanup | Methodically remove artifacts from disk, memory, services, and network layer by layer |
| Defense improvement | Permanent firewall rules, full patching, CIS Benchmark hardening |
| Vulnerability management | Scan environment, hunt for related vulnerabilities, correlate with threat intelligence |
