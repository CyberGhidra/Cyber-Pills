# The Digital Forensics Investigation Process

**Reference document:**
- **DFRWS — A Road Map for Digital Forensic Research** — [https://dfrws.org/wp-content/uploads/2019/06/2001_USA_a_road_map_for_digital_forensic_research.pdf](https://dfrws.org/wp-content/uploads/2019/06/2001_USA_a_road_map_for_digital_forensic_research.pdf)

---

## Overview

Digital forensics is the application of scientific methods to the identification, preservation, collection, examination, analysis, presentation, and decision-making process around digital evidence. It underpins both criminal investigations and corporate incident response — and the same rigorous methodology applies in both contexts, because evidence that is handled incorrectly loses its value in court and in post-incident review alike.

The seven phases below are not strictly sequential in all cases — some overlap, and some may iterate — but the order matters because each phase is designed to protect the integrity of the evidence passed to the next.

```
Identification → Preservation → Collection → Examination
                                                  │
                Decision ← Presentation ← Analysis
```

> **Core principle running through all seven phases:** digital evidence is fragile and easily altered. Every decision made throughout the process must prioritize maintaining the integrity and admissibility of the evidence — even when doing so is slower or less convenient.

---

## Phase 1 — Identification

**Goal:** Confirm that an incident or crime has actually taken place, locate the relevant evidence, and define the scope of the investigation before touching anything.

### Confirming the Incident
Before a forensic investigation begins, there must be reasonable confirmation that something has happened that warrants one. Acting on an unconfirmed report wastes resources and risks disturbing a scene unnecessarily.

Sources of identification include:
- **Victim reports** — a user, employee, or organization reports suspicious activity, data loss, unauthorized access, or other indicators. This is often the starting point, but victim reports alone are not sufficient — they must be corroborated.
- **Detection and control systems** — alerts from SIEM, EDR, IDS/IPS, or other monitoring tools (as covered in the Identification phase of Incident Response) providing technical evidence that something occurred.
- **Audits** — scheduled or unscheduled reviews of logs, access records, or system configurations that surface anomalies not flagged by automated tools.

### Preparing to Respond and Collect Evidence
Once the incident is confirmed, the investigator must:
- **Locate the evidence** — identify which systems, devices, storage media, accounts, or network segments are likely to hold relevant evidence. This requires understanding the incident type and the environment.
- **Preserve before collecting** — establish that nothing will be altered before evidence is secured (more on this in Phase 2).
- **Define scope and focus efforts accordingly** — a forensic investigation cannot examine everything. The scope defines the boundaries: which systems are in scope, which time window is relevant, which user accounts are of interest. A clearly defined scope prevents investigation sprawl and keeps the work manageable and legally defensible.

> **Scoping matters legally:** evidence collected outside the defined and authorized scope may be inadmissible. In corporate investigations, this intersects with employee privacy rights and HR policies. In criminal investigations, it intersects with search warrant limitations.

---

## Phase 2 — Preservation

**Goal:** Ensure the evidence remains intact and unaltered from the moment the investigation begins. Preservation is the foundation on which the entire investigation rests — if it fails here, nothing that follows is reliable.

### Capturing a Snapshot of the Device's Current State
The first preservation action is to capture the current state of the device before anything changes:
- **Powered-on systems** — capture a memory dump before shutdown to preserve volatile data (running processes, open connections, encryption keys, cached credentials). As demonstrated in the containment phase, memory is lost the moment power is cut.
- **Powered-off systems** — do not power on. Booting a device from its own operating system can alter timestamps, write to disk, and modify the very evidence being investigated. Use a forensic boot environment instead.
- **Mobile devices** — enable airplane mode immediately to prevent remote wipes, data sync, or incoming messages that could alter the evidence state.

### Identifying Relevant Data — Location and Volatility
Not all data has the same urgency. Evidence must be collected in **order of volatility** — most volatile first, because it disappears fastest:

```
Most volatile  ─►  CPU registers and cache
                   RAM (running processes, open connections, encryption keys)
                   Network connections and ARP tables
                   Temporary files and swap space
                   Disk (files, logs, unallocated space)
                   Remote logs and backups
◄─  Least volatile    Printed documents and physical media
```

Disk evidence will still be there in an hour; RAM evidence will not survive a reboot.

### Chain of Custody
The chain of custody is the documented, unbroken record proving that evidence has been handled properly from the moment of collection to its presentation in court or in a report. It is what separates admissible evidence from inadmissible evidence.

Chain of custody documentation must record:
- **Who** — every person who has had possession of or access to the evidence
- **What** — a precise description of the evidence (device type, serial number, hash value)
- **Where** — where the evidence was collected from and where it has been stored
- **When** — exact date and time of every transfer, access, or action taken
- **Why** — the reason each person accessed the evidence

Any gap in the chain of custody — even an innocent, undocumented transfer between two legitimate investigators — can be used by a defense attorney to challenge the integrity of the evidence.

### Securing the Evidence
Once captured, evidence must be physically and logically secured:
- Forensic images stored on write-protected media or in access-controlled storage
- Physical devices stored in tamper-evident bags with seals, in a locked evidence room
- Hash values (SHA-256) recorded at collection and re-verified at each subsequent access to confirm no alteration has occurred

---

## Phase 3 — Collection

**Goal:** Gather all of the identified evidence in a forensically sound manner that preserves its integrity and legal admissibility.

### Legal Authority
Before collecting evidence — especially in criminal or regulatory contexts — the investigator must have clear legal authority to do so:
- **Criminal investigations** — typically requires a search warrant specifying what can be seized and from where
- **Corporate investigations** — typically authorized by company policy, employment agreements, and the mandate of the legal or HR team; the scope of authority should be documented before collection begins
- **Cross-border investigations** — evidence held in different jurisdictions may require separate legal processes; international investigations add significant legal complexity

Collecting evidence without the appropriate authority, even with the best intentions, can result in the evidence being ruled inadmissible and the investigator facing legal liability.

### Order of Volatility
As identified during Preservation, collection follows volatility order — most volatile first. This means memory capture before disk imaging, and live system data before powered-off storage.

### Data Reduction
A large investigation may involve terabytes of data. Data reduction strategies narrow the collection to what is actually relevant:
- **Filtering by time window** — only collect data from the relevant period (e.g., logs from the 48 hours surrounding the suspected incident)
- **Filtering by user or account** — focus on accounts known to be involved
- **Deduplication** — remove duplicate files (same hash = same content) to reduce the volume without losing unique evidence
- **Known-good exclusion** — use hash libraries of known-good system files to exclude them from analysis, focusing attention on files not already accounted for

### Documentation
Every item collected must be logged immediately:
- Description and unique identifier of the evidence item
- Hash value at time of collection (MD5 and/or SHA-256)
- Collection method and tool used
- Collector's name, date, and time
- Chain of custody form updated

---

## Phase 4 — Examination

**Goal:** Inspect the collected evidence to extract the relevant data from it, without altering the original.

### Maintaining Evidence Integrity
All examination work is done on **forensic copies** — never on the original evidence. The original is preserved untouched; the forensic image (verified by hash) is what the examiner works with. If the working copy is accidentally modified, another copy can be made from the original. If the original were worked on directly and altered, the evidence would be permanently compromised.

### Data Extraction and Interpretation
Examination involves making the evidence readable and navigable:
- **File system analysis** — recovering the directory structure, file metadata (creation, modification, access timestamps), and file contents from disk images
- **Deleted file recovery** — files "deleted" by the operating system are often still present in unallocated disk space until overwritten; forensic tools can frequently recover them
- **Log parsing** — extracting and normalizing log data from system logs, application logs, security logs, and event logs into a format that can be analyzed
- **Artifact extraction** — pulling specific forensic artifacts: browser history, registry hives, prefetch files, jump lists, LNK files, shellbags — the digital footprints left by user and system activity

### Examination Checklist
A structured checklist ensures no category of evidence is overlooked:
- [ ] Disk image created and hash verified
- [ ] Memory dump captured (if applicable)
- [ ] File system timeline generated
- [ ] Deleted files recovery attempted
- [ ] Log files extracted and parsed
- [ ] Browser artifacts examined
- [ ] Registry/system configuration reviewed (Windows)
- [ ] User account activity reviewed
- [ ] Network artifacts extracted (connection logs, DNS cache, ARP tables)
- [ ] Metadata examined for all relevant files

### Data Discovery
Examination also involves actively searching for evidence that may not be immediately obvious:
- Keyword searches across disk images for specific terms (email addresses, domain names, file names)
- Hash matching against known malware libraries (NSRL, VirusTotal)
- Steganography detection — data hidden inside legitimate-looking files
- Encrypted volume detection — identifying containers that may hold evidence the examiner cannot yet access

---

## Phase 5 — Analysis

**Goal:** Interpret the evidence extracted during Examination to understand what happened, how, when, who was involved, and what the impact was.

Examination is *what the data says*; Analysis is *what the data means*.

### Data Context
Raw evidence rarely tells a complete story on its own. Context determines meaning:
- A file in an unusual location may be innocent or may be an attacker's dropped payload — context (what process created it, when, under what user account, in what sequence of events) determines which
- A login at 3 AM is suspicious on a system that normally has no after-hours activity, but normal on a 24/7 operations platform
- Context comes from combining evidence across multiple sources and comparing it against what "normal" looks like for that environment

### Iteration
Analysis is rarely linear. New findings raise new questions, which require revisiting the evidence, running additional searches, or returning to earlier phases to collect additional data that wasn't initially identified as relevant. A good investigator follows the evidence, not a predetermined conclusion.

### Relational Analysis
Establishing connections between entities in the evidence:
- Which user accounts interacted with which systems?
- Which IP addresses communicated with the compromised host?
- Which files were accessed by the malicious process?
- Which other systems were accessed using the same compromised credentials?

Relational analysis maps the attack surface and identifies lateral movement, helping answer "how far did the attacker get?"

### Functional Analysis
Understanding the *purpose* of what was found:
- What does this piece of malware actually do? (connect back to C2, dump credentials, encrypt files, exfiltrate data)
- What was this tool used for in the context of this incident?
- What capability did the attacker gain from this artifact?

Functional analysis often requires looking up malware families, decoding obfuscated scripts, or running samples in a controlled sandbox environment.

### Temporal Analysis
Building the timeline of events:
- When did the initial compromise occur?
- When did the attacker establish persistence?
- When did lateral movement begin?
- When was data accessed or exfiltrated?
- How long was the attacker present before detection?

Temporal analysis is one of the most powerful tools in forensics — a precise timeline of attacker actions, built from timestamps across multiple data sources, is both the clearest way to understand an incident and one of the most compelling forms of evidence in court or in an executive report.

---

## Phase 6 — Presentation

**Goal:** Communicate the findings of the investigation clearly, accurately, and appropriately for the intended audience.

### Ensuring Findings Are Communicated Clearly
The most technically rigorous investigation loses its value if the findings cannot be understood and acted upon. Presentation requires translating technical findings into language appropriate for each audience — without losing accuracy in the process.

### Guiding the Audience Through the Investigation
A forensic presentation is not a data dump — it is a narrative. The investigator guides the audience through:
1. What happened (the incident summary)
2. How we know (the evidence and methodology)
3. What it means (the impact and conclusions)
4. What comes next (recommendations)

The structure must be logical and the conclusions must be directly supported by the evidence presented — every finding must be traceable back to a specific artifact.

### Executive Summary
A concise, non-technical summary for leadership and stakeholders who need to understand the impact and make decisions, but do not need the technical detail:
- What happened, in plain language
- What data or systems were affected
- What the business impact is or could be
- What actions are being recommended

The executive summary should stand alone — a reader who reads only this should understand the situation well enough to make the decisions they need to make.

### Comprehensive Documentation
The full technical report for investigators, legal teams, regulators, and anyone who may need to reproduce or audit the investigation:
- Methodology — every step taken, every tool used, every decision made
- Evidence inventory — every artifact collected, with hashes and chain of custody
- Findings — detailed, evidence-supported description of what the investigation determined
- Timeline — the complete chronological reconstruction of events
- Appendices — raw data, tool outputs, logs, and supporting materials

> **Documentation standard:** the report must be written such that another qualified forensic investigator, given the same evidence, could reproduce the same findings following the documented methodology. Reproducibility is what makes forensic evidence scientific rather than testimonial.

---

## Phase 7 — Decision

**Goal:** Evaluate the findings and determine the appropriate course of action based on what the investigation concluded.

### Evaluating the Results
The decision phase begins with a clear-eyed assessment of what the investigation actually proved:
- What can be stated with high confidence, supported by strong evidence?
- What is probable but not definitively proven?
- What remains unknown or uncertain?
- Are there gaps in the evidence that could be challenged?

Overstating conclusions is as damaging as understating them — in legal proceedings, conclusions that cannot be fully supported by the evidence can undermine the entire case.

### Making a Decision
Based on the findings, the relevant stakeholders (IR team, legal, management, law enforcement) decide on a course of action:

| Outcome | Action |
|---------|--------|
| **Criminal activity confirmed** | Refer to law enforcement, preserve evidence for prosecution |
| **Policy violation confirmed** | HR and legal process, disciplinary action |
| **External threat actor** | Coordinate with law enforcement, notify cyber insurer, regulatory disclosure if required |
| **No malicious intent found** | Close the investigation, document findings, review detection capabilities that triggered the investigation |
| **Inconclusive** | Determine whether additional evidence gathering is feasible; if not, document the uncertainty and close with recommendations |

### Prosecution
When the decision is to pursue criminal prosecution:
- All evidence must be handed to law enforcement in accordance with chain of custody requirements — any gap here can result in evidence being ruled inadmissible
- The forensic investigator may be required to provide expert testimony explaining the technical findings to a court in non-technical terms
- The investigation documentation (methodology, findings, chain of custody) becomes a legal document subject to discovery — it must be complete, accurate, and defensible
- Timelines for evidence preservation and legal holds must be respected; destroying or modifying evidence after a legal hold is in place constitutes spoliation, which carries serious legal consequences

---

## Summary

| Phase | Core Question | Key Output |
|-------|--------------|-----------|
| **Identification** | Did something happen? What and where is the evidence? | Confirmed incident, defined scope |
| **Preservation** | How do we ensure the evidence stays intact? | Memory/disk snapshots, chain of custody established |
| **Collection** | How do we gather all relevant evidence legally and safely? | Evidence inventory, forensic copies, documentation |
| **Examination** | What does the evidence contain? | Extracted artifacts, parsed logs, recovered files |
| **Analysis** | What does the evidence mean? | Timeline, attack reconstruction, impact assessment |
| **Presentation** | How do we communicate the findings? | Executive summary + full technical report |
| **Decision** | What do we do with the findings? | Prosecution, remediation, or closure |

> **The thread that runs through all seven phases:** every action must be documented, every artifact must be hashed, every handoff must be recorded. Digital evidence is only as strong as the process used to handle it.
