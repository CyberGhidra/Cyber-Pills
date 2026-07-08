# Incident Response — Containment

**Reference document:**
- **SANS Incident Handling — Containment Form** — [https://www.sans.org/media/score/incident-forms/IH-Containment.pdf](https://www.sans.org/media/score/incident-forms/IH-Containment.pdf)

---

## Why Containment Comes Before Eradication

The instinct during an active incident is to immediately remove the threat — delete the malware, reset the passwords, wipe the machine. This is the wrong order of operations. Acting too fast causes two serious problems:

1. **Evidence destruction** — wiping a system before capturing a forensic image permanently destroys the data needed to understand what happened, how the attacker got in, and what they accessed.
2. **Incomplete response** — if you don't fully understand the scope of the compromise before you act, you may remediate one machine while leaving three others untouched, and the attacker simply pivots back through those.

Containment is the deliberate, controlled step that **stops the situation from getting worse** while you gather enough information to act decisively. It is divided into two stages — short-term and long-term — for exactly this reason.

> **Key principle:** Contain first, understand fully, then eradicate. Speed without understanding creates the illusion of response while leaving the attacker in place.

---

## 1. Short-Term Containment

Short-term containment is the immediate, tactical response to an active threat. The priority is to **stop the bleeding right now** — limit the attacker's ability to move, escalate, or exfiltrate — without necessarily cleaning up or restoring systems yet.

### Quarantining and Isolation
The most immediate action available: cutting an affected system off from the rest of the network so the attacker cannot use it as a pivot point to reach other systems.

Methods of isolation:

| Method | How | When to use |
|--------|-----|-------------|
| **Network isolation** | Remove the network cable, disable the NIC, or use EDR (e.g., LimaCharlie's isolation feature) to cut network access while keeping the machine powered on | Most common first action — preserves memory/processes for evidence while cutting C2 communication |
| **VLAN segmentation** | Move the affected host(s) to an isolated VLAN with no routing to the rest of the network | When multiple hosts in a segment are affected |
| **Taking down a system** | Full shutdown | Last resort — destroys volatile evidence (RAM, running processes); only appropriate when isolation is not possible and the risk of further damage outweighs the evidence loss |

> **Important:** prefer network isolation over full shutdown whenever possible. A powered-on, network-isolated machine retains its memory, running processes, and open file handles — all of which are valuable evidence (as demonstrated in the Linux Process Analysis lab with `/proc` and `lsof +L1`).

### Network Filters and Rules
Beyond isolating individual hosts, blocking malicious traffic at the network level prevents the attacker from communicating with compromised systems or exfiltrating data even if you haven't identified every affected host yet.

- **Firewall rules** — block the attacker's known IPs, C2 domains, or the specific ports/protocols used by the malware (e.g., blocking outbound connections to port 4444, as used by the `notmalware.elf` reverse shell in the labs)
- **IPS/IDS signatures** — deploy signatures matching the known attack patterns to detect and block similar activity elsewhere on the network in real time
- **DNS sinkholes** — redirect known malicious domains to a controlled IP, cutting C2 communication for any host attempting to call back to those domains

### Killing Processes
If an attacker's process (reverse shell, malware, C2 agent) is actively running on a host, terminating it immediately cuts their interactive access — though this alone does not remove persistence mechanisms. Kill the process only **after** evidence has been captured (memory dump, process snapshot) if at all possible.

### Temporary Patches
In some scenarios, a known vulnerability is being actively exploited. Applying a temporary workaround — disabling a vulnerable service, removing a vulnerable component, or blocking the specific exploit payload — can stop the active exploitation without requiring a full patch/reboot cycle that could be disruptive to business operations during the incident.

---

## 2. Business Impact Assessment

Short-term containment actions are disruptive by nature — taking systems offline, blocking network traffic, and killing processes all affect legitimate business operations. This is why containment decisions must be made carefully and in coordination with the right people.

### Plan Containment Measures Carefully
Before taking a system offline or blocking a network segment, ask:
- What business functions does this system support?
- Who depends on it, and how urgently?
- Is there a fallback or redundant system that can take over?
- What is the cost of taking it down vs. the cost of leaving it up while compromised?

These are not purely technical questions — they require business judgment, which is why containment decisions should involve more than just the IR team.

### Work with Data Owners and Stakeholders
As defined during Preparation, **data owners** are the business-side owners of the systems and data involved. They must be consulted before major containment actions because:
- They understand the operational impact better than IT does
- They may have time-sensitive dependencies (e.g., a payroll run, a customer deadline, a regulatory reporting window) that affect the timing of containment
- They share accountability for the decision — and that shared accountability is important both operationally and legally

### Mission-Critical Systems
Some systems are so central to business operations that taking them offline requires explicit executive approval, regardless of the severity of the compromise. These systems should be identified and flagged during Preparation (in the asset inventory). During an incident, the IR lead must escalate quickly and clearly when a mission-critical system is involved — presenting the options, the risks of each, and a recommendation, then allowing the appropriate authority to make the call.

> Never unilaterally take down a mission-critical system without authorization. The business impact of the wrong call here can rival the damage of the incident itself.

---

## 3. Evidence Collection and System Backup

Before any remediation action that could alter or destroy data, evidence must be collected and preserved. This is the bridge between short-term and long-term containment.

### Forensic Image
A **forensic image** is a bit-for-bit copy of a storage device — not just the files, but every sector of the disk, including deleted files, unallocated space, and file system metadata. Tools like `dd`, FTK Imager, or Autopsy are used for this purpose.

The forensic image:
- Preserves the state of the disk at the time of capture
- Can be verified with a hash (SHA-256 of the image must match the hash of the original) to prove it has not been tampered with
- Allows analysis to be done on the copy, leaving the original untouched
- Is admissible in legal proceedings if properly documented

### Memory Capture
**RAM (volatile memory)** contains information that disappears the moment a system is powered off:
- Running processes and their arguments
- Open network connections
- Encryption keys held in memory
- Attacker tools that were never written to disk (fileless malware)
- Credentials cached in memory (as demonstrated with Mimikatz/LSASS in the LimaCharlie lab)

Tools like **Volatility**, **WinPmem** (Windows), or `LiME` (Linux kernel module) are used to capture a full memory dump before shutdown or remediation. Memory forensics is one of the most powerful techniques available to IR analysts precisely because it captures what disk forensics cannot.

### Evidence Documentation
Every piece of collected evidence must be documented immediately at the time of collection:

- What was collected (description of the artifact)
- Where it came from (system name, IP, physical location)
- Who collected it (name and role)
- When it was collected (exact date and time, ideally UTC)
- Hash value at time of collection (MD5 and/or SHA-256)
- Storage location and access controls applied

This documentation forms the **chain of custody** — the unbroken, verifiable record proving that evidence has not been altered between collection and presentation (whether in a legal proceeding, regulatory audit, or internal review).

---

## 4. Long-Term Containment

Once evidence is secured and the immediate crisis is stabilized, long-term containment focuses on **making the environment stable and defensible** while the full rebuilding and remediation work (Eradication phase) takes place. This may last hours, days, or weeks depending on the scale of the incident.

### Removing the Attacker's Access

The attacker likely established multiple ways to maintain access. All of them must be identified and removed:

- **Deleting unauthorized user accounts** — accounts created by the attacker during the intrusion (often named to blend in with legitimate accounts)
- **Resetting credentials** — any account that may have been compromised must have its password reset and, where possible, MFA re-enrolled. This includes service accounts, admin accounts, and any account whose credentials were exposed (e.g., captured by Mimikatz)
- **Disabling remote access** — temporarily disable VPN, RDP, SSH, and other remote access mechanisms until the environment is confirmed clean, to prevent the attacker from re-entering through legitimate remote access channels
- **Removing backdoors** — web shells, scheduled tasks, cron jobs (as covered in the Linux Cron Jobs lab), modified startup entries, and any other persistence mechanisms planted by the attacker must be identified and removed

> **MITRE ATT&CK reference:** T1098 (Account Manipulation), T1053.003 (Cron), T1547.001 (Registry Run Keys) — all common persistence techniques that must be systematically reviewed and removed during long-term containment.

### Traffic Segmentation
Even after removing known attacker access, additional network segmentation reduces the blast radius if any persistence mechanism was missed:
- Isolate sensitive network segments (e.g., domain controllers, financial systems, PII stores) from less critical ones
- Apply stricter firewall rules between segments — zero trust principles: no system should have more network access than it strictly needs
- Block lateral movement paths the attacker used (e.g., if the attacker moved via SMB, apply stricter SMB access controls)

### Additional Monitoring and Detection
During and immediately after an incident, increase monitoring sensitivity across the environment:
- Lower alert thresholds to catch low-and-slow attacker behavior that might otherwise be filtered as noise
- Add specific detection rules targeting the exact TTPs (Tactics, Techniques, and Procedures) used in this incident
- Monitor the attacker's known IOCs (IPs, domains, hashes) across all systems — not just the ones already confirmed as compromised — to detect lateral movement that may not yet have been identified
- Increase log retention temporarily so that if new evidence surfaces later, historical logs are available

### Patching and Hardening
If the attacker gained initial access via a known vulnerability, that vulnerability must be patched on every affected and potentially vulnerable system before those systems are considered clean:
- Apply vendor patches for the exploited CVE
- Identify all other systems running the same vulnerable software version
- Harden configurations (disable unnecessary services, remove default credentials, restrict permissions) to reduce the attack surface for any follow-on attempts
- Consider whether the vulnerability was the root cause or a symptom — if the attacker used a phishing email to deliver the payload, patching the exploited system doesn't address the phishing problem

---

## Summary

```
INCIDENT CONFIRMED
        │
        ▼
┌───────────────────────────────┐
│   SHORT-TERM CONTAINMENT      │
│   ─ Isolate affected hosts    │
│   ─ Block malicious traffic   │
│   ─ Kill active processes     │
│   ─ Apply temporary patches   │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   BUSINESS IMPACT ASSESSMENT  │
│   ─ Consult data owners       │
│   ─ Assess mission-critical   │
│     systems                   │
│   ─ Get authorization for     │
│     high-impact actions       │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   EVIDENCE COLLECTION         │
│   ─ Forensic disk image       │
│   ─ Memory capture            │
│   ─ Chain of custody docs     │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   LONG-TERM CONTAINMENT       │
│   ─ Remove attacker access    │
│   ─ Delete backdoors/crons    │
│   ─ Reset credentials         │
│   ─ Segment traffic           │
│   ─ Increase monitoring       │
│   ─ Patch and harden          │
└───────────────┬───────────────┘
                │
                ▼
          ERADICATION
```

| Area | Key Activity |
|------|-------------|
| Short-term containment | Isolate hosts, block traffic, kill processes, temporary patches |
| Business impact | Consult data owners and stakeholders, escalate for mission-critical systems |
| Evidence collection | Forensic image, memory dump, chain of custody documentation |
| Long-term containment | Remove attacker access, disable backdoors, segment network, patch, increase monitoring |
