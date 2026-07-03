# Malicious Code 

## 1. What Is Malware?

**Malware** (malicious software) is any software intentionally designed to cause damage, gain unauthorized access, steal data, or disrupt normal operations of a system. It is an umbrella term covering many specific categories — ransomware, trojans, worms, viruses, spyware, rootkits, and more — each with distinct behaviors, delivery methods, and objectives.

Understanding malware isn't just about knowing *what* each type does, but also recognizing the **indicators of compromise (IoCs)** — the observable signs that a system has been infected — since detection often relies on spotting these patterns rather than the malware announcing itself.

---

## 2. Ransomware

**Ransomware** encrypts a victim's files (or locks access to a system) and demands payment — typically in cryptocurrency — in exchange for the decryption key. Modern ransomware often uses **double extortion**: exfiltrating data before encrypting it, then threatening to leak it publicly even if the ransom is paid.

### Indicators of Compromise (Ransomware)
- Files suddenly becoming inaccessible or renamed with unusual extensions
- Ransom note files appearing in multiple directories (e.g., `README_DECRYPT.txt`)
- A spike in disk I/O activity as files are being encrypted in bulk
- Inability to open common file types (documents, images, databases)
- Shadow copies / backup files being deleted (a common ransomware technique to prevent recovery)

---

## 3. Trojans (Trojan Horses)

A **trojan** disguises itself as legitimate, useful software to trick the user into installing it. Unlike a virus, it does not self-replicate — it relies entirely on **social engineering** to get the user to run it voluntarily.

### Example Flow: Trojan Infection via a Fake App

```
[User searches for an app]
          │
          ▼
[User downloads a malicious app,
 believing it to be legitimate]
          │
          ▼
[Malicious app installs and runs,
 sending system/user info to a
 remote Command & Control (C2) server]
          │
          ▼
[Remote C2 server sends commands
 back to the infected device,
 which executes them silently]
```

### Indicators of Compromise (Trojan)
- Unexpected outbound network connections to unknown/external IP addresses
- New, unrecognized processes or services running at startup
- Unusual permission requests from a newly installed app (e.g., a "flashlight app" requesting SMS or contact access)
- Unexplained increase in data usage or CPU/battery consumption
- Antivirus/EDR alerts flagging C2 beaconing behavior

---

## 4. Worms

A **worm** is similar to a virus in that it's self-replicating, but critically, it does **not** require a host file or user interaction to spread — it propagates autonomously across networks by exploiting vulnerabilities.

```
[Worm exploits a network vulnerability
 on a target system]
          │
          ▼
[Worm installs itself and immediately
 scans the network for other vulnerable hosts]
          │
          ▼
[Worm replicates itself onto new hosts
 without any user action, and the
 infected host may also communicate
 with a remote C2 server]
```

### Indicators of Compromise (Worm)
- Sudden, abnormal spikes in network traffic/bandwidth consumption
- Rapid, simultaneous infection of multiple systems across the network
- Systems becoming slow or unresponsive due to worm scanning/propagation activity
- Unexpected outbound connections attempting to reach other internal hosts on unusual ports

> 🧩 **Raspberry Robin**: A notable real-world worm, initially spread via infected **USB removable drives**, that has since evolved into a delivery mechanism for other malware (including ransomware and other malicious payloads) and is known for using legitimate Windows processes to evade detection. It's a good modern example connecting the **removable device threat vector** (from the Threat Landscape section) directly to worm propagation.

---

## 5. Bots and Botnets

A **bot** is a single compromised device under the remote control of an attacker. When many bots are networked together and controlled centrally, they form a **botnet**.

- The controlling attacker (often called a "bot herder") issues commands through a **Command and Control (C2)** infrastructure.
- Botnets are commonly used for **DDoS attacks**, spam distribution, cryptomining, and credential stuffing at scale.
- C2 communication can be centralized (single server) or **decentralized/peer-to-peer**, which is harder to disrupt since there's no single point of failure.

---

## 6. Stuxnet (Worm Case Study)

**Stuxnet** is one of the most famous worms in history, discovered in 2010. It specifically targeted **industrial control systems (ICS)**, particularly Siemens SCADA systems used in uranium enrichment facilities. It is widely regarded as a nation-state-developed cyberweapon and is a textbook example of:

- A worm that spread partly via **removable USB drives** (bridging air-gapped networks)
- The use of **multiple zero-day exploits** in a single campaign
- Malware designed to cause **physical damage** (it subtly altered centrifuge speeds to cause equipment failure) rather than just steal data

---

## 7. Spyware

**Spyware** covertly monitors user activity and collects information (browsing habits, keystrokes, credentials, location) without the user's consent, typically transmitting it to a remote attacker.

### Common Indicators of Compromise (Spyware)
- Unexplained pop-ups or changes to browser homepage/search engine
- New, unrecognized browser extensions or toolbars
- Noticeable system slowdown, especially during web browsing
- Increased network activity even when the system is idle
- Webcam or microphone activity indicator lights turning on unexpectedly

---

## 8. Bloatware

**Bloatware** refers to pre-installed or bundled software that isn't strictly malicious but consumes system resources, clutters the device, and can sometimes introduce unnecessary security risk (larger attack surface, outdated/unpatched components). It's often included by device manufacturers or bundled during software installations.

---

## 9. Viruses

A **virus** is malicious code that attaches itself to a legitimate host file or program and requires **user action** (e.g., opening a file, running a program) to execute and spread. Every virus carries a **payload** — the actual malicious action it performs once triggered (e.g., deleting files, exfiltrating data, displaying a message).

### Types of Viruses

| Type | Description |
|---|---|
| **Memory-resident virus** | Installs itself and resides in RAM, remaining active and able to infect other files/programs even after the original host program has closed |
| **Non-resident virus** | Executes, performs its action, and does not remain active in memory afterward — it only runs when the infected file itself is executed |
| **Boot sector virus** | Infects the master boot record (MBR) of a storage device, executing before the operating system even loads, making it particularly hard to detect/remove |
| **Macro virus** | Embedded in documents (e.g., Word, Excel) using macro scripting languages; executes when the document is opened and macros are enabled |
| **Email virus** | Spreads via email attachments or embedded links, often leveraging the victim's contact list to self-propagate further |

---

## 10. Fileless Malware (Example Walkthrough)

**Fileless malware** operates without writing a traditional executable file to disk, instead living in memory and leveraging legitimate system tools (e.g., PowerShell, WMI) — making it significantly harder for traditional signature-based antivirus to detect.

### Example Flow: Fileless Attack via Malicious Link

```
[User clicks a malicious link]
          │
          ▼
[Link redirects to a compromised
 or malicious website]
          │
          ▼
[Website triggers a script that
 launches a legitimate shell/
 scripting tool, e.g., PowerShell]
          │
          ▼
[Shell executes a malicious script
 entirely in memory — no file
 written to disk]
          │
          ▼
[Payload executes and establishes
 persistence, e.g., by creating a
 malicious Windows Registry entry
 that re-triggers the script on
 reboot]
```

> 💡 **Exam tip:** Because fileless malware avoids writing to disk, detection typically relies on **behavioral analysis** (EDR, monitoring for abnormal PowerShell/WMI activity) rather than traditional file-signature scanning.

---

## 11. Removing Malware — And the Problem of Certainty

Malware removal typically follows these general steps: **identify** the infection, **isolate** the affected system (disconnect from the network to prevent further spread), **remove** the malicious code (via antivirus/EDR tools or manual remediation), and **restore** the system to a known-good state.

However, there's a critical, often underappreciated problem: **you can rarely be 100% certain that a system is fully clean after a malware infection.**

- Sophisticated malware (especially rootkits — see below) can hide its own presence and even hide evidence of related malicious files or processes.
- Malware may have created additional **persistence mechanisms** (scheduled tasks, registry keys, backdoor accounts) that aren't obvious even after the primary payload is removed.
- Some malware disables or tampers with security tools themselves, making their scan results unreliable.

> 🧩 **Best practice:** For critical or highly sensitive systems, especially those suspected of deep compromise (e.g., rootkit infection), the recommended approach is often a **full wipe and reimage** from a known-clean, trusted source — rather than trusting removal tools to guarantee complete eradication.

---

## 12. Keyloggers

A **keylogger** records keystrokes typed by a user, typically to capture credentials, personal data, or sensitive communications. Keyloggers can be software-based (running as a hidden process) or hardware-based (a physical device inserted between the keyboard and computer).

### Indicators of Compromise (Keylogger)
- Delayed or "laggy" response when typing (particularly with some software keyloggers)
- Unknown or hidden processes running in the background
- Unusual outbound network traffic transmitting small, regular bursts of data (exfiltrating captured keystrokes)
- Presence of an unfamiliar physical device connected between the keyboard cable and the computer (for hardware keyloggers)
- Antivirus/EDR flags on processes with keyboard hook/input-monitoring behavior

---

## 13. Logic Bombs

A **logic bomb** is malicious code intentionally inserted into a legitimate program that remains dormant until a specific **trigger condition** is met (e.g., a certain date, a specific user action, or the absence of a particular event, such as an employee's account being deactivated).

### Indicators of Compromise (Logic Bomb)
- Discovery of unexplained/unauthorized code segments during a code review, often containing conditional statements tied to dates or specific triggers
- Sudden, unexplained malicious activity occurring at a very specific, predictable time (e.g., exactly at midnight on a particular date)
- Malicious activity correlating with a specific employee's termination or account deactivation date
- Code with no clear legitimate business purpose embedded within otherwise normal application logic

---

## 14. Rootkits

A **rootkit** is a stealthy form of malware designed to gain and maintain privileged (root/administrator) access to a system while actively hiding its own presence — and often the presence of other malware — from the operating system, users, and security tools.

### Indicators of Compromise (Rootkit)
- Antivirus/security tools failing to run, update, or report unusual "clean" results despite other signs of compromise
- System files, processes, or registry entries that are invisible through normal OS tools but detectable via specialized rootkit scanners or offline analysis
- Unexplained changes in system behavior (crashes, slowdowns) with no identifiable cause through standard monitoring
- Discrepancies between what's reported by the OS and what's found via a low-level/offline scan (a classic rootkit detection technique)
- Network traffic to known C2 infrastructure despite no visible process appearing to generate it

> 💡 **Exam tip:** Rootkits are often the reason behind the "can't be sure it's fully removed" problem discussed in Section 11 — because a well-designed rootkit's entire purpose is to make the system *appear* clean even when it isn't. This is why offline/bootable-media scanning is often recommended over relying on tools running from within a potentially compromised OS.

---

## Summary

| Concept Group | Key Idea |
|---|---|
| Malware (general) | Umbrella term for intentionally malicious software; detection relies on Indicators of Compromise (IoCs) |
| Ransomware | Encrypts/locks data for ransom; double extortion; watch for shadow copy deletion |
| Trojan | Disguised as legitimate software; relies on social engineering; communicates with C2 |
| Worm | Self-replicating, no host file/user action needed; spreads via network vulnerabilities (e.g., Raspberry Robin, Stuxnet) |
| Bot/Botnet | Compromised devices controlled centrally via C2; used for DDoS, spam, cryptomining |
| Spyware | Covertly monitors/collects user data and activity |
| Bloatware | Unwanted pre-installed/bundled software; not malicious but risky/wasteful |
| Virus | Requires host file + user action; types: memory-resident, non-resident, boot sector, macro, email |
| Fileless malware | Operates in memory using legitimate tools (e.g., PowerShell); avoids signature-based detection |
| Malware removal | Identify, isolate, remove, restore — but full certainty of eradication is never guaranteed |
| Keylogger | Captures keystrokes (software or hardware-based) |
| Logic bomb | Dormant malicious code triggered by a specific condition/event |
| Rootkit | Gains privileged access while actively hiding itself and other malware from detection |

---
